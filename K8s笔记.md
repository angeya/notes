## 常用命令：

Kubernetes（通常缩写为k8s）是一个开源的容器编排系统，用于自动化部署、扩展和管理容器化应用程序。kubectl是Kubernetes的命令行工具，用于与Kubernetes集群交互。下面是一些常用的kubectl命令及其用途：
1. 查看资源：

  ```bash
  kubectl get pods # 列出当前命名空间下的所有 Pod。
  kubectl get pods -n <命名空间> # 列出指定命名空间中的所有 Pod。
  kubectl get pods -o wide # 显示更多详细信息，如 Pod 所在的节点、IP 地址等。
  kubectl get services # 列出当前命名空间下的所有服务（Services）。
  kubectl get nodes # 列出 Kubernetes 集群中的所有节点。
  kubectl get deployments # 列出当前命名空间下的所有部署（Deployments）。
  ```

2. 描述资源：

  ```bash
  kubectl describe pod <pod-name> # 显示有关特定 Pod 的详细信息，包括事件、容器状态和配置详情。
  kubectl describe node <node-name> # 提供有关特定节点的详细信息。
  kubectl describe service <service-name> # 显示特定服务的详细信息。
  ```

3. 创建和删除资源：

  ```bash
  kubectl apply -f <file.yaml> # 根据 YAML 文件创建或更新资源。
  kubectl create -f <file.yaml> # 根据 YAML 文件创建资源。
  kubectl delete pod <pod-name> # 删除指定的 Pod。
  kubectl delete -f <file.yaml> # 删除 YAML 文件中定义的资源。
  kubectl delete namespace <命名空间> # 删除指定的命名空间及其中的所有资源。
  ```

4. 与 Pod 交互：

  ```bash
  kubectl logs <pod-name> # 显示特定 Pod 的日志。
  kubectl exec -it <pod-name> -- /bin/bash # 在指定的 Pod 中打开一个 Shell，进行交互式调试。
  kubectl port-forward <pod-name> <本地端口>:<pod端口> # 将本地端口转发到 Pod 的指定端口。
  ```

5. 扩展应用：

  ```bash
  kubectl scale deployment <deployment-name> --replicas=<副本数量> # 将部署的副本数量扩展到指定数量。
  ```

6. ConfigMaps 和 Secrets：

  ```bash
  kubectl get configmap # 列出当前命名空间中的所有 ConfigMap。
  kubectl describe configmap <configmap-name> # 提供特定 ConfigMap 的详细信息。
  kubectl get secret # 列出当前命名空间中的所有 Secret。
  kubectl describe secret <secret-name> # 显示特定 Secret 的详细信息。
  ```

7. 命名空间管理：

  ```bash
  kubectl get namespaces # 列出集群中的所有命名空间。
  kubectl create namespace <命名空间名称> # 创建一个新的命名空间。
  kubectl delete namespace <命名空间名称> # 删除指定的命名空间。
  ```

8. 监控和调试：

  ```bash
  kubectl top nodes # 显示节点的资源使用情况（CPU、内存）。
  kubectl top pods # 显示 Pod 的资源使用情况。
  kubectl describe events # 列出并描述集群中的事件。
  ```