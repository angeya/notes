## Nginx常用配置

### 简介

- 常用命令

```shell
nginx -s stop     快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。
nginx -s stop filename    快速关闭使用某个配置文件开启的Nginx。
nginx -s quit     平稳关闭Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload   因改变了Nginx相关配置，需要重新加载配置而重载。
nginx -s reopen   重新打开日志文件。
nginx -c filename 为 Nginx 指定一个配置文件，来代替缺省的。
nginx -t          不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法
nginx -v 显示      nginx 的版本。
nginx -V 显示      nginx 的版本，编译器版本和配置参数
```



### 基本配置项

```nginx
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别依次为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。
    client_max_body_size 50m; #设置允许请求体的最大大小，默认为1M。如果需要上传文件，可能需要设置大一点

    upstream mysvr {   
        server 127.0.0.1:7878;
        server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;  #监听地址    
        gzip on; #可配置是否使用压缩的方式来发送数据
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```

### 静态资源

```nginx
server {
    listen 83;
    server_name localhost;

    location  / { #静态页面
        root html;  #使用相对路径(windows绝对路径：D:\dev-env\nginx-1.20.2\html\) 
        index /index.html; #配置首页
    }

    location  /image { #图片静态资源
        alias html/image; #如果使用root(则会拼接路径)，配置为：root html/
        autoindex on; # 请求不指定文件时自动显示目录内容
    }

    location /pdf{ #图片静态资源
        alias html/pdf;
        try_files $uri $uri/ /index.html; #顺序检查文件是否存在，返回第一个找到的文件或文件夹，如果都不存在，则会重定向到最后一个参数
    }
}
```

### 正向代理

```nginx
http{
	resolver 8.8.8.8; # DNS解析器
	server{ # 没有server_name的server
		listen 8888;
		location / {
			proxy_pass http://$http_host$http_uri; #转发请求地址
		}
	}

}
# 配置电脑代理ip和端口
```



### 反向代理

```nginx
http{
    include       mime.types;
    default_type  application/octet-stream;
    
    server {  #代理服务1
		listen 81;
		server_name localhost;
		location / {
			proxy_pass http://172.1.1.136:8080; # 代理地址
		}
	}

    server { #代理服务2
		listen 82;
		server_name localhost;
		location / {
			proxy_pass http://172.1.1.170:8080;
		}
	}
}
```

### 负载均衡

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;
	
	# 代理（可以配置n个服务 负载均衡，通过weight=1设置权重）
	upstream servers {
		server 127.0.0.1:9091 weight=2;
		server 127.0.0.1:9092 weight=1;
	}
    
    # 代理（如果A失败，则会转到B处理，用户无感知。A失败两次后，将会不会被分发请求，直到60s后）
    upstream servers2 {
    	server 127.0.0.1:8001 fail_timeout=60s max_fails=2; # Server A
    	server 127.0.0.1:8002 fail_timeout=60s max_fails=2; # Server B
	}
    
    #每个server是一个服务
    server {
        listen       8088; #监听的端口号
        server_name  localhost;

        location / {
            proxy_pass http://servers; # 代理到servers
        }
		# 对image转发(实现动静分离)
        location /image/ {
        	root html; #静态文件所在路径
			autoindex on;
        }
    }
}
```
### 支持WebSocket

本质上是添加请求头，并设置http版本为1.1。

```nginx
location /index/ {
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection 'upgrade';
	proxy_pass http://127.0.0.1:80/home.html;
}
```



### Nginx配置常用指令说明

#### proxy_set_header

设置请求头：`add_header header_name header_value [always];`可选的 `always` 参数表示无论响应状态码是什么，都应该添加该头部。

```nginx
# 设置并传递原始请求的host
proxy_set_header Host $host;
```



#### proxy_hide_header

去掉响应头：`proxy_set_header header_name header_value;`

```nginx
# 不设置内容处理方式，例如有的服务通过设置此响应头来强制下载文件
proxy_hide_header Content-Disposition;
# 隐藏Mime类型，比如 text/html 类型，这时候浏览器会显示html的代码，而不是渲染
proxy_hide_header Content-Type;
```



#### add_header 

设置响应头：`add_header header_name header_value [always];`可选的 `always` 参数表示无论响应状态码是什么，都应该添加该头部。

```nginx
# 设置响应的内容类型
add_header Content-Type applicaiton/json;
```

