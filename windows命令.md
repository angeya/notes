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

## Java相关

- 后台运行 jar 包

  ```bash
  start javaw -jar xxx.jar > log
  exit
  ```

  

- 

