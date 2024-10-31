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

### 修改终端编码为UTF-8

使用`chcp`命令可以显示或更改当前的代码页（Code Page）。代码页决定了在命令行窗口中使用的字符集，因此影响文本的显示和输入。

```cmd
chcp # 查看当前代码页
chcp 65001 # 将当前代码页设置为65001
```

**常见代码页**

- **437**：美国（默认）
- **65001**：UTF-8
- **936**：简体中文 (GBK)
- **950**：繁体中文 (BIG5)

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





