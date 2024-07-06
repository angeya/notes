### Linux常用命令（CentOS_7）

#### 文件管理

```shell
 ls # 简单查看当前目录内容
 ll # 列出目录详细信息 为ls -al的简写 可自定义命令
 cd 目录 # 改变目录(change directory) 特殊参数：..上级目录 -上一次目录 ~用户目录
 mv 源文件 目标文件 # 移动文件(move) 源文件与目标文件目录一致，文件名不一致可以实现重命名
 rm <-rf> 文件 # 删除文件 -r递归(目录使用) -f强制删除，避免提问
 mkdir 目录 # 创建目录 参数-p 递归创建目录
 touch 文件名 # 创建空文件
 vim 文件名 # 打开文件，如果文件不存在则创建空文件饼打开
 find 路径 -name 文件名 # 在指定路径下查找文件 参数-user可以根据用户查文件
 tail -20 文件名 # 查看文件的最后20行内容 参数-f 表示一直显示文件尾
 tailf -20 文件名 # 查看文件最后20行内容，并跟随文件增加显示（滚动屏幕）
 stat 文件名 # 文件状态 显示最近时间aTime:访问时间 mTime:内容改变时间 cTime:文件属性改变时间 linux没有文件创建时间的概念
 wc 文件名 # 显示文件行数  ll 文件夹|wc 可以显示文件夹有多少个文件(需要减去ll命令带来的1)
 grep hello test.txt # 查找test.xtx文件中的hello字符串
 scp test.txt root@172.1.1.136:/root/ # 使用scp命令传输文件到其他机器，scp -r 传输目录
 sha1sum file # 获取文件sha1算法校验和，也可以通过md5sum file获取md5算法的校验和
```

#### 部署相关

```shell
java -jar XXX.jar # 运行jar包
java -Dfile.encoding=utf-8 -jar xxx.jar # 设置文件编码，一般在用IO流的时候使用
nohup java -jar XXX.jar > log & # 后台运行jar包，控制台输出到log文件 输出到/dev/null不输出日志  windows中后台运行jar包：start javaw -jar xxx.jar
ps -aux | grep XXX # 查找名为xxx的进程
ps -ef | grep XXX # 查找名为xxx的进程
pgrep -f XXX # 查找名为XXX的进程的pid
grep -r --include 'fileNamePattern' 'keyword' /root/data # 递归在指定文件中查找关键字
service xxx status # 查看某服务的状态  active为运行 inactive为停止，可用systemctl代替
top -d 3 | grep XXX # 显示当前系统运行进程对资源的占用 -d 3表示mei3秒刷新 grep进行筛选
kill 进程id # 结束进程 参数 -9 为强制结束进程
-Dserver.port=8088 # 启动jar包时 设置覆盖配置文件的属性
--server.port=8088 # 与上一条相同
java -XX:+PrintCommandLineFlags -version # 查看jvm默认参数
jmap -heap pid # 查看某个java进程的内存参数
jstack -pid # 查看java进程中各个线程运行的堆栈信息
# idea打包普通jar包时，Directory for META-INFO/MANIFEST.MF路径包含项目根路径即可
./xxx.sh # 前台启动sh脚本,没有生成相对应进程，使用ps命令查不到，通过sh xxx.sh方式或者运行方式启动可以用ps查到。如果ps查不到可以通过ctrl+D停止，如果包含死循环则关闭控制台即可关闭
```

#### 用户与权限

```shell
useradd 用户名 # 新增用户
adduser 用户名 # 新增用户
userdel 用户名 # 删除用户 不删除home目录
passwd 用户名 # 修改用户密码
su 用户名 # 切换用户
passwd -l user # 密码锁定用户
passwd -u user # 密码解除用户
vim /etc/ssh/sshd_config # 新增一行DenyUsers test，禁止test用户登录ssh，重启sshd生效
cat /etc/passwd # 查看用户信息 passwd文件保存用户信息
chown -R root:ftpuer ftpuserhome # 将ftpuserhome路径拥有者设置为root组(可选)ftpuser用户
who # 用户最近登录信息
last # 查看用户登录历史
whoami # 查看当前登录用户
source # 在当前bash环境下读取并执行文件中的命令 如source test.sh
```

#### 系统维护

```shell
free -h # 查看内存和交换区使用情况
df -h # 磁盘剩余空间(disk free) -h 以人类友好的单位方式显示
hostnamectl set-hostname name # 设置主机名
du -sh * # 查看当前目录下各个文件(夹)所占存储空间大小
date -s "2020-02-02 08:30:30" # 按照日期 时分秒设置系统时间
date -s "08:30:30" # 按照时分秒设置系统时间
mysql -u用户名 -p # 登录mysql
service 服务名 restart # 重启服务 如mysql，vsftpd
rpm -ivh xxx.rpm # rpm包安装
rpm -qa | grep mysql # 查看是否安装mysql 或者 rpm -qa mysql
rpm -e --nodeps `rpm -qa | grep mysql` # 卸载mysql，如果安装有的话
whereis java # 查找java命令路径 whereis mysql查看mysql的路径 
systemctl isolate multi-user.target  #更改为命令模式启动
systemctl isolate graphical.target  #更改为图形界面启动
journal -xe # 查看系统日志，存放systemctl等的日志，日志文件存放于/var/log/messages
netstat -ntlp # 当前网络正在使用的端口
firewall-cmd --list-ports # 查看防火墙开发的端口
firewall-cmd --zone=public --add-port=2000-2005/tcp --permanent # 添加防火墙 移除remove
systemctl stop filrewalld # 关闭防火墙
nc -l -p 9999# 打开socket监听端口，可以使用telnet命令连接 nc —ul 999是udp的方式
nc ip 9999 < test.txt # 发送文本文件test.txt的内容到对应主机的端口,也可以发送数据进行通信
echo $JAVA_HOME # 打印系统变量，可用于查找安装目录
env # 查看当前用户环境变量，包括JAVA_HOME等
netstat -anp|grep 8080 # 查看8080端口是否正在使用
cat /etc/redhat-release # 查看RedHat系列发行版本
mysql -h127.0.0.1 -P3306 -uroot -p # mysql数据库登录，本地主机和3306端口可以省略
mysqldump -h 127.0.0.1 -uroot -proot book_sharing user > /user.sql # 转储到sql文件 mysqldump [-h地址] [-u用户] [-p密码] 库名 [表名] > sql文件路径
psql -h127.0.0.1 -Upostgres -p # -U是大写
psql -h 127.0.0.1 -U postgres -f xxx.sql # 导入sql脚本
export HISTTIMEFORMAT="%F %T " # 设置history命令格式（增加时间列），只对当前ssh session有用，全局可以在/etc/profile中设置
!! # 执行上一条命令
ntsysv # 查看当前系统开启启动的进程有哪些，如果仅仅想查看一个进程服务，可以使用systemctl status即可
lsof -i :8086 # 查看8086端口的使用情况
alias # 显示所有命令的别名 设置别名：alias ll='ls -l'。删除别名 unalias ll。（命令方式为临时方式，写在文件 ~/.bashrc 中则永久生效）
```

#### 软件包管理

##### yum

Yum(全称为 Yellow dogUpdater, Modified)是一个在Fedora和RedHat以及CentOS中的Shell前端软件包管理器。基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

1，设置阿里 yum 源

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

/etc/yum.repos.d 目录为 yum 源配置目录，只要文件后缀为 repo 就会被加载。

2，常用命令

```bash
# 清空缓存 yum 安装一个软件的时候会把软件包下载到本地指定的目录中，为了节省磁盘空间，可以使用以下命令清空缓存
yum clean packages 														# 清除缓存目录下的软件包，清空的是(/var/cache/yum)下的缓存
yum clean headers 														# 清除缓存目录下的 headers
yum clean oldheaders 													# 清除缓存目录下旧的 headers
yum clean, yum clean all (= yum clean packages; yum clean oldheaders) 	     # 清除缓存目录下的软件包及旧的headers

# 生成缓存
yum clean all
yum makecache

# yum 显示信息
yum list          												    # yum list显示所有已经安装和可以安装的程序包   
yum list installed                                                      # 显示已经安装的软件包
yum list <package_name> 											# 显示安装包信息rpm，显示installed ，这里是包名，版本和仓库名
yum list repolist all												# 查询所有的yum仓库
yum info <package_name>                           					   # 显示安装包rpm的详细信息
yum groupinfo <group_name>             								  # 显示程序组group信息

# yum 搜索查看
yum search string 													# 根据关键字string查找安装包
yum deplist <package_name>											 # 仅仅 查看程序rpm依赖情况
yum provides */命令												   # 查看命令是由哪个包提供的（这个命令很有帮助）

# yum 安装
yum -y install <package_name>           							# 不加-y则会询问是否安装，想控制哪些包安装，则不要加-y，否则可以加-y
yum install --downloadonly --downloaddir=/xx/xxx/xx/				 # 只下载软件但不安装

# yum 卸载删除软件
yum remove <package_name>						# 卸载程序包，此卸载命令会对 yum 或 rpm 安装的包生效，如果是编译安装的，则不受yum控制
yum groupremove <group_name>					# 删除程序组group

# yum 包升级
yum check-update 							    # 检查可更新的软件有哪些
yum update 									    # 更新升级所有软件包
yum update <package_name> 						 # 更新指定程序包package   
yum upgrade <package_name> 						 # 升级指定程序包package

# 安装 yum 扩展源
yum  install -y epel-release # 我们安装的网络yum源基本都是基础的yum源，有些软件不一定能在其找得到，而epel源是一个扩展源，里面有很多软件，所以安装epel扩展源是一个很好的选择              
```

##### rpm



### 无法识别域名(一般出现在虚拟机中)

比如ping www.baidu.com 的时候报错：Name or service not known。但是ping 百度的ip可以ping通。

大概率是DNS配置的问题。解决步骤如下：

1. 设置DNS服务器

   编辑文件`/etc/resolv.conf`，在文件中增加如下DNS服务器配置：

   ```
   nameserver 8.8.8.8
   nameserver 8.8.4.4
   ```

   保存退出，重启服务器（经确认不重启也可以！！），之后再ping 一次试一试。

2. 配置以太网卡

   如果以上方法不行。

   编辑文件`/etc/sysconfig/network-scprits/ifcfg-ens33`,文件名可能不是-ens33而是其他编号，或者类似eth0等。

   在文件中 找到 ONBOOT=NO 改成 ONBOOT=yes

   重启网络： `systemctl restart network`

### 查看linux的IP地址

使用ifconfig命令会输出多个信息，本机ip的信息一般在ens33中。

### Linux系统运行级别

init级别	systemctl target   说明 （重置密码1，常用3，图形5）

```bash
systemctl get-default # 查看当前运行级别
systemctl set-default target # 设置运行级别，重启生效
```



0	shutdown.target	系统停机状态
1	emergency.target	单用户工作状态
2	rescure.target	多用户状态（没有NFS）
3	multi-user.target	多用户状态（有NFS）
4	无	系统未使用，留给用户
5	graphical.target	图形界面
6	无	系统正常关闭并重新启动

#### shell脚本基础

```shell
# 赋值时等号左右两边不能有空格
a="hello"

# 引用变量时，变量名前面加上$符号
a=$param

# 使用linux命令时，两边使用反引号引起来 ``
pid=`pgrep -f data-export`

-z $p # 判断参数是否为null或者空字符串
-n $p # 判断参数是否已经被赋值
-d $p # 判断参数是否为目录
-f $p # 判断参数是否为文件

$ # $#：参数个数 $0：脚本本身的名字 $1：传递给脚本的第一个参数 $2类似  
echo # 打印 -n：不换行 -e：可以解析输出内容中的特殊字符
exist # exist 0 正常退出  exist 1不正常退出
sleep n # n为休眠的秒数

```

#### 文件存在但是shell提示 No such file or directory

通过vim编辑器，:set ff查看文件格式，:set ff=unix修改格式