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
# 如果出现如下报错，可能是因为权限不够，需要使用管理员打开cmd再操作
错误: 无法终止 PID 为 xxx 的进程。
原因: 拒绝访问。
```

查看端口占用进程

```cmd
netstat -ao | findstr 9025
```



### 网络请求 CURL

curl 是一个功能强大的命令行工具，用于传输数据，支持多种协议（如HTTP、HTTPS、FTP等）。它广泛用于测试API、下载文件、发送请求等场景。

Windows 和 Linux 上的 curl 命令在功能和用法上总体上是**一致的**，因为它们都来源于同一个开源项目（由 Daniel Stenberg 开发），核心代码和参数是相同的。

> Linux 支持单引号，Windows只支持双引号，特别是提交请求体 json 的时候注意需要转义双引号。

#### 基本语法

```bash
curl [选项] [URL] # 默认是get请求
```

- **[选项]**：控制curl的行为，例如指定请求方法、添加头信息等。
- **[URL]**：目标地址，例如 https://example.com。

#### 指定请求方法 -X

通过语法 `-X, --request <METHOD>` 指定请求方法，默认是GET

```bash
curl -X POST https://example.com/api
```

#### 指定请求体数据

如果是普通请求体，通过命令 `-d, --data <DATA>` 发送POST请求时附带数据，格式通常为键值对。

```bash
curl -X POST -d "key1=value1&key2=value2" https://example.com/api
```

如果数据是JSON格式，需要配置 `-H`设置请求头指定内容为json

```bash
curl -X POST -H "Content-Type:application/json" -H "Authorization: Basic xxxx"  -d "{\"name\":\"John\"}" http://127.0.0.1/getSequenceByCode?code=Demo
```

如果数据中包含文件，需要通过`-F`指定为`multipart/form-data`类型

```bash
curl -X POST -H "Authorization: Basic xxxx" -F "file=@D:\Desktop\images\test.jpg" -F "businessId=12345678" http://127.0.0.1/oss/uploadFile
```

> 多个请求头`-H`和多个请求体`-F`参数的时候，都是需要多次指定的
> `@`符号可以用来指定本地文件，不止文件上传，json请求体也可以使用本地文件

#### 指定User-Agent头

通过`-A, --user-agent <STRING>`命令可以指定User-Agent请求头

```bash
curl -A "Mozilla/5.0" https://example.com
```

#### 将响应保存到指定文件

使用`-o, --output <FILE>`参数将响应保存到指定文件，响应内容太多时好用

```bash
curl -o output.txt https://example.com
```

使用`-O, --remote-name`参数使用URL中的文件名保存文件，常用于下载。

```bash
curl -O https://example.com/file.zip
```

#### 自动跟随重定向

使用``参数自动跟随重定向（例如301、302）

```bash
curl -L https://example.com
```

#### 超时参数

参数`-m, --max-time <SECONDS>`设置请求最大超时时间（秒）

```bash
curl -m 10 https://example.com
```

参数`--connect-timeout <SECONDS>`设置连接超时时间（秒）

```bash
curl --connect-timeout 5 https://example.com
```

#### 显示详细的请求和响应信息

使用`-v, --verbose`参数显示详细的请求和响应信息，便于调试

```bash
curl -v https://example.com
```

#### 跳过SSL证书验证

参数`-k, --insecure`设置跳过SSL证书验证（用于自签名证书或测试环境）

```bash
curl -k https://example.com
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





