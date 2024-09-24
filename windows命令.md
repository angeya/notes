## Windows 开发环境管理

### 命令详情

使用以下格式可以查看命令的使用详情

命令 /?；如 ipconfg /？

###  系统服务

- 查看服务

  运行 services.msc

- 开启服务

  net start 服务名，如net start mysql

- 停止服务

  net stop 服务名

- 删除服务

  sc delete 服务名

### 进程管理

- 查看进程

  所有进程：tasklist

  筛选：tasklist | findstr chrome

- 停止进程

  taskkill /F /PID 进程id

- 查看端口占用进程

  netstat -ao | findstr 9025

## Java相关

- 后台运行 jar 包

  ```bash
  start javaw -jar xxx.jar > log
  exit
  ```


## SSH客户端

在windows中，可以在cmd或者powerShell中使用ssh命令连接到ssh服务器。示例如下

```cmd
ssh 127.0.0.1 -l root # 参数-l 可以指定登录的用户名。-p 可以指定端口号。
```





