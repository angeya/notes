## 基础

### 命令详情

使用以下格式可以查看命令的使用详情

```cmd
command /?  # 如 ipconfg /？
```

### 命令注释

在cmd中命令的注释符号是 `#`，而在 bat 批处理文件中注释符号是 `--`。

### 命令路径带空格

```cmd
# 需要执行如下命令时，因为目录中包含空格，命令会被拆分解析，导致提示Program命令不存在
D:\Program Files\2345Soft\HaoZip\HaoZipC
# 这时候应该使用引号包裹起来，如下
"D:\Program Files\2345Soft\HaoZip\HaoZipC"
```

### 常见基础命令

```cmd
robocopy .\ .\simple-service\src\main\resources\static\ /E /XD "simple-service" /XD "ok" /XF "start.bat"
-- pushd .\simple-service\src\main\resources\static\
-- ren *.* *.doc
-- popd
cd .\simple-service
call mvn package
echo package finished....
cd .\target
"D:\Program Files\2345Soft\HaoZip\HaoZipC" x simple-service-1.0-SNAPSHOT.jar -y
-- del /Q simple-service-1.0-SNAPSHOT.zip
-- rename simple-service-1.0-SNAPSHOT.jar simple-service-1.0-SNAPSHOT.zip
-- rmdir /S /Q .\output\
-- powershell -Command "Expand-Archive -Path 'simple-service-1.0-SNAPSHOT.zip' -DestinationPath 'output\' -Force"
cd ..\..\
robocopy .\simple-service\target\BOOT-INF\classes\static .\ok
pause
```



## Windows 开发环境管理

###  系统服务

查看服务

```cmd
运行 services.msc
```

开启服务

```cmd
net start 服务名 # 如net start mysql
```

停止服务

``` cmd
net stop 服务名
```

删除服务

```cmd
sc delete 服务名
```

### 进程管理

查看进程

```cmd
tasklist # 所有进程

tasklist | findstr chrome # 进程过滤
```

停止进程

```cmd
taskkill /F /PID 进程id
```

查看端口占用进程

```cmd
netstat -ao | findstr 9025
```



### 网络请求

curl命令，Windows内置的，用法和linux的curl类似。

```cmd
# 访问接口或者页面
curl https://www.baidu.com
# 下载文件
curl -O https://www.demo.com/demo.pdf
```



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





