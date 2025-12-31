curl -LO https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl
kubectl version --client



curl -Lo kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
kind version



kubectl run my-nginx \
  --image=nginx:alpine \
  --port=80


root@sunny-server:/home/sunny# kubectl get pod -A
NAMESPACE            NAME                                         READY   STATUS         RESTARTS   AGE
default              my-nginx                                     0/1     ErrImagePull   0          37s

查看详情 排查原因：
root@sunny-server:/home/sunny# kubectl describe pod my-nginx
xxx
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  4m1s                  default-scheduler  Successfully assigned default/my-nginx to kind-control-plane
  Warning  Failed     115s (x3 over 3m31s)  kubelet            Failed to pull image "nginx:latest": rpc error: code = DeadlineExceeded desc = failed to pull and unpack image "docker.io/library/nginx:latest": failed to resolve reference "docker.io/library/nginx:latest": failed to do request: Head "https://registry-1.docker.io/v2/library/nginx/manifests/latest": dial tcp 31.13.96.193:443: i/o timeout
  Normal   Pulling    74s (x4 over 4m1s)    kubelet            Pulling image "nginx:latest"
  Warning  Failed     44s (x4 over 3m31s)   kubelet            Error: ErrImagePull
  Warning  Failed     44s                   kubelet            Failed to pull image "nginx:latest": rpc error: code = DeadlineExceeded desc = failed to pull and unpack image "docker.io/library/nginx:latest": failed to resolve reference "docker.io/library/nginx:latest": failed to do request: Head "https://registry-1.docker.io/v2/library/nginx/manifests/latest": dial tcp 31.13.96.194:443: i/o timeout
  Warning  Failed     19s (x6 over 3m30s)   kubelet            Error: ImagePullBackOff
  Normal   BackOff    7s (x7 over 3m30s)    kubelet            Back-off pulling image "nginx:latest"
  
在宿主机中先拉取镜像，然后“塞进”kind集群 行不通。

配置在容器中拉取镜像使用宿主机的镜像。
vim kind-config.yaml
内容 完整粘贴（别改）：

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = [
      "https://docker.m.daocloud.io",
      "https://mirror.ccs.tencentyun.com",
      "https://dockerproxy.com"
    ]
	

kind create cluster --config kind-config.yaml


再执行kubectl run my-nginx \
  --image=nginx:alpine \
  --port=80
  即可成功

删除pod kubectl delete my-nginx。


直接创建的pod删除了就没有了。deployment才是应用，生产用，通过deployment来管理pod
Pod = 易碎品
Deployment = 可恢复、可扩缩、可升级
kubectl create deployment nginx-deploy --image=nginx:alpine
pod的名字为deployment名字+随机生成的序列号
root@sunny-server:/home/sunny# kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-56b64874bf-w82mm   1/1     Running   0          8m1s

二、等 nginx 一跑起来，我们立刻进入“爽点”
下一步我会马上带你：
1️⃣ kubectl scale 到 3 副本
2️⃣ 删 Pod，看自愈
3️⃣ 用 Service 暴露
4️⃣ port-forward 本地访问
5️⃣ 一次性讲清 Pod / Service / Endpoint

三，k8s的自愈能力
删除pod
root@sunny-server:/home/sunny# kubectl delete pod nginx-deploy-56b64874bf-w82mm
pod "nginx-deploy-56b64874bf-w82mm" deleted

查看pod，新的 Pod 立刻创建 名字变了，但 Deployment 还在
root@sunny-server:/home/sunny# kubectl get pod -w
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-56b64874bf-nnn8j   1/1     Running   0          3s


四、扩容：一句命令，3 个副本
kubectl scale deployment nginx-deploy --replicas=3
root@sunny-server:/home/sunny# kubectl scale deployment nginx-deploy --replicas=3
deployment.apps/nginx-deploy scaled

pod变成了3个
root@sunny-server:/home/sunny# kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-56b64874bf-5pd4b   1/1     Running   0          33s
nginx-deploy-56b64874bf-brwwf   1/1     Running   0          33s
nginx-deploy-56b64874bf-nnn8j   1/1     Running   0          8m25s

五，让“集群里的服务”能被访问（Service）
现在 nginx 在集群里，但：
不能稳定访问 Pod IP
Pod 重建 IP 会变
👉 Service 就是“稳定入口”
root@sunny-server:/home/sunny# kubectl get pod -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-deploy-56b64874bf-5pd4b   1/1     Running   0          2m47s   10.244.0.9   kind-control-plane   <none>           <none>
nginx-deploy-56b64874bf-brwwf   1/1     Running   0          2m47s   10.244.0.8   kind-control-plane   <none>           <none>
nginx-deploy-56b64874bf-nnn8j   1/1     Running   0          10m     10.244.0.7   kind-control-plane   <none>           <none>

创建 Service（ClusterIP（默认，可以改为NodePort），不对外暴漏，只能在集群内访问，如果有多个pod，ClusterIP会自动做负载均衡，轮询转发到多个pod。如果是NodePort Service则对外暴漏）
kubectl expose deployment nginx-deploy \
  --port=80 \
  --target-port=80 \
  --name=nginx-svc
查看服务
root@sunny-server:/home/sunny# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   96m
nginx-svc    ClusterIP   10.96.206.47   <none>        80/TCP    24s
ClusterIp是由kube-porxy分配的吗？


六、第一次访问集群内的服务（port-forward）
kubectl port-forward svc/nginx-svc 8080:80

因为转发的是127.0.0.1，不是0.0.0.0，这里主要用于测试
root@sunny-server:/home/sunny# kubectl port-forward svc/nginx-svc 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

生产环境将使用NodePort或者Ingress

Pod  <-- Service (ClusterIP) <-- Ingress / NodePort <-- 浏览器


三种方式，目前平台使用NodePort
方式一：kubectl port-forward
    |
    |（仅本机，调试用）
    v
Service / Pod

方式二：NodePort   ← 今天主角
    |
    |（NodeIP:300xx）
    v
Service (NodePort)
    |
ClusterIP → Pod

方式三：Ingress


七 NodePort
NodePort 到底在干什么（核心机制）
当你创建一个 NodePort Service：
type: NodePort
ports:
- port: 80
  targetPort: 80
  nodePort: 30010
K8s 会在 每一台 Node 上 做三件事：

1️⃣ 打开一个端口 30010
2️⃣ 把流量转到 Service 的 80
3️⃣ 再由 Service 负载到 Pod

NodeIP:30010
   |
[kube-proxy]
   |
ClusterIP:80
   |
Pod:80

任何Node Ip都能访问到pod，对于如下平台，10.44.2.216:30010 和10.44.2.217:30010其实访问的东西是一样的
[root@master1 ~]# kubectl get node -o wide
NAME      STATUS   ROLES           AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
master1   Ready    control-plane   497d   v1.24.2   10.44.2.216   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
master2   Ready    control-plane   497d   v1.24.2   10.44.2.217   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
master3   Ready    control-plane   497d   v1.24.2   10.44.2.218   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker1   Ready    <none>          497d   v1.24.2   10.44.2.230   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker2   Ready    <none>          497d   v1.24.2   10.44.2.231   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker3   Ready    <none>          497d   v1.24.2   10.44.2.232   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker4   Ready    <none>          497d   v1.24.2   10.44.2.233   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker5   Ready    <none>          497d   v1.24.2   10.44.2.235   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker6   Ready    <none>          497d   v1.24.2   10.44.2.219   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20
worker7   Ready    <none>          497d   v1.24.2   10.44.2.220   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.20

创建一个 NodePort Service。假设已经有 nginx Pod（NodePort是转发到pod上的吗？这样不稳定）：
kubectl expose pod my-nginx \
  --type=NodePort \
  --port=80 \
  --target-port=80 \
  --name=my-nginx-np

Deployment	无状态服务
StatefulSet	有状态
DaemonSet	每节点一个
Job / CronJob	任务

这样是稳定的
kubectl expose deployment my-nginx \
  --type=NodePort \
  --port=80 \
  --target-port=80 \
  --name=my-nginx-svc
  
  然后访问Node Ip加端口号即可


## 常用命令：

Kubernetes（通常缩写为k8s）是一个开源的容器编排系统，用于自动化部署、扩展和管理容器化应用程序。kubectl是Kubernetes的命令行工具，用于与Kubernetes集群交互。

下面是一些常用的kubectl命令及其用途（如果查看某种资源的查看但是不想指定命名空间，可以增加参数`-A`表示所有、不限制命名空间）：

1. 命名空间管理：

  ```bash
kubectl get namespaces # 列出集群中的所有命名空间。
kubectl create namespace <命名空间名称> # 创建一个新的命名空间。
kubectl delete namespace <命名空间名称> # 删除指定的命名空间。
  ```

2. 查看资源：

  ```bash
  kubectl get pods # 列出当前命名空间下的所有 Pod。
  kubectl get pods -n <命名空间> # 列出指定命名空间中的所有 Pod。
  kubectl get pods -o wide # 显示更多详细信息，如 Pod 所在的节点、IP 地址等。
  kubectl get services # 列出当前命名空间下的所有服务（Services）。
  kubectl get nodes # 列出 Kubernetes 集群中的所有节点。
  kubectl get deployments # 列出当前命名空间下的所有部署（Deployments）。
  ```

3. 描述资源：

  ```bash
  kubectl describe pod <pod-name> -n <namespace> # 显示有关特定 Pod 的详细信息，包括事件、容器状态和配置详情。可以查看有哪些容器
  kubectl describe node <node-name> # 提供有关特定节点的详细信息。
  kubectl describe service <service-name> # 显示特定服务的详细信息。
  ```

4. 创建和删除资源：

  ```bash
  kubectl apply -f <file.yaml> # 根据 YAML 文件创建或更新资源。 
  kubectl create -f <file.yaml> # 根据 YAML 文件创建资源。
  kubectl delete pod <pod-name> -n <namespace> # 删除指定的 Pod。如果Pod 是由 ReplicaSet、Deployment 等控制器管理，控制器会自动创建一个新的 Pod替代被删除的 Pod，从而实现“重启”。
  kubectl delete -f <file.yaml> # 删除 YAML 文件中定义的资源。
  kubectl delete namespace <命名空间> # 删除指定的命名空间及其中的所有资源。
  ```

5. 与 Pod中的容器交互：

  ```bash
  # pod中只有唯一的容器，可以不指定容器
  # -- 符号是分隔符表示后面的参数是容器中要执行的命令。-it中i是交互式，t是伪终端，没有it参数不会打开交互式命令
  kubectl logs -n <namespace> pod-name # 显示特定 Pod 的日志。-f 跟踪 --tail= 指定行数；和docker一样
  kubectl exec -n <namespace> pod-name -- env # 在指定的 Pod 中打开一个 Shell，进行交互式调试。这是Pod中只有一个容器的情况
  kubectl exec -it -n <namespace> pod-name -- /bin/bash # 在指定的 Pod 中打开一个 Shell，进行交互式调试。这是Pod中只有一个容器的情况
  kubectl exec -it -n <namespace> pod-name -c my-container -- /bin/bash # 和上条命令类似，适用于有多个容器的情况，要指定容器
  kubectl port-forward pod-name <本地端口>:<pod端口> # 将本地端口转发到 Pod 的指定端口。
  ```

7. 扩展应用：

  ```bash
  kubectl scale deployment <deployment-name> --replicas=<副本数量> # 将部署的副本数量扩展到指定数量。
  ```

8. ConfigMaps 和 Secrets：

  ```bash
  kubectl get configmap # 列出当前命名空间中的所有 ConfigMap。
  kubectl describe configmap <configmap-name> # 提供特定 ConfigMap 的详细信息。
  kubectl get secret # 列出当前命名空间中的所有 Secret。
  kubectl describe secret <secret-name> # 显示特定 Secret 的详细信息。
  ```

9. 监控和调试：

  ```bash
  kubectl top nodes # 显示节点的资源使用情况（CPU、内存）。
  kubectl top pods # 显示 Pod 的资源使用情况。
  kubectl describe events # 列出并描述集群中的事件。
  ```

10. 查看 `Service` 的域名和 ClusterIP 映射

```bash
kubectl get svc -A
```




