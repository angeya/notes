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
df -h # 磁盘剩余空间(disk free) -h 以人类友好的单位方式显示
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
```

#### 软件包管理

##### yum

1，设置阿里 yum 源

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

/etc/yum.repos.d 目录为 yum 源配置目录，只要文件后缀为 repo 就会被加载。

##### rpm



linux系统运行级别说明：

init级别	systemctl target   说明 （重置密码1，常用3，图形5）

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