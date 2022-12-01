# Jenkins使用教程

## 常见问题

### 插件总是下载失败

1. 将官方插件镜像修改为国内的镜像，如清华源：`http://mirror.esuni.jp/jenkins/updates/update-center.json`

   修改位置：Manage Jenkins -> Plugin Manager -> Advanced -> Update Site -> URL

2. 找到Jenkins工作目录，在 updetes/default.json 文件中：
   把：`http://www.google.com/` 全部替换成 `http://www.baidu.com/`
   把：`https://updates.jenkins.io/download` 全部替换成 `http://mirrors.tuna.tsinghua.edu.cn/jenkins`

3. 重启Jenkins服务

### Git获取代码失败

1. 如果是超时，则使用VPN或者稍后再试。

2. 如果是无法获取，可以试着将默认分支 */master 改为 */main