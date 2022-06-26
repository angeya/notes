# Shell编程

## 1.Shell 概述

shell是一个命令行解释器,它接收应用程序/用户命令，进而调用操作系统内核。

shell还是一个功能强大的编程语言，易编写、易调试、灵活性强。学习shell编程对应用部署有很大帮助。

一些注意事项：

1. 字符串使用单引号或者双引号都可以，区别是双引号可以解析内部变量，单引号不可以

    字符串内部还有字符串时，最好使用外层单引号内层双引号（外双内单awk工具语法错误）

2. 获取命令行的执行结果，可以将命令行使用反引号引起来，如p=\`ps -ef|grep idiom\`

3. echo # 打印 -n：不换行 -e：可以解析输出内容中的特殊字符

4. exist # exist 0 正常退出 exist 1不正常退出

5. sleep n # n为休眠的秒数

## 2.Shell 解析器

linux提供的shell解析器：

```shell
[root@VM-16-6-centos ~]# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
/bin/tcsh
/bin/csh
```

bash和sh的关系：

```shell
[root@cloud-host ~]# ll /bin/ | grep bash
-rwxr-xr-x    1 root root      922792 Jan 15 14:43 bash
lrwxrwxrwx    1 root root           4 Aug  7  2020 sh -> bash
```

查看默认解析器：

```shell
[root@cloud-host ~]# echo $SHELL
/bin/bash
```

## 3.shell脚本入门

- 脚本格式

脚本以 #!/bin/bash 开头，作用为指定解释器，如果没有指定解释器则使用系统默认解释器。

- 第一个shell脚本：test.sh

  ```shell
  #!/bin/bash
  echo "hello world!"
  ```

- 执行方式：

  1. 解释器 + 文件，通过ps命令可以查看进程

     ```shell
     bash test.sh
     sh test.sh
     ```

  2. ./文件，本质是文件自己执行，需要有执行权限，且ps命令无法查看进程，ssh会话结束脚本停止执行

     ```shell
     chmod +x test.sh
     ./test.sh
     ```

## 4. shell中的变量

### 1、系统变量

- 常用的系统变量

  $HOME，$PWD，$SHELL，$USER

- 查看系统变量的值

  ```shell
  [root@cloud-host ~]# echo $HOME
  /root
  ```

- 显示当前shell中所有系统变量

  ```shell
  [root@cloud-host ~]# set
  ABRT_DEBUG_LOG=/dev/null
  BASH=/bin/bash
  BASH_ALIASES=()
  ```

### 2、自定义变量

- 基本语法

  定义变量：变量名=值

  撤销变量：unset 变量名

  生命静态变量：readonly 变量名（只读，不能撤销）

  使用变量：$变量名

  

- 变量定义规则

  变量名规则一样，环境变量建议大写加下划线。

  赋值等号左右两边不能有空格。

  在bash中，变量默认类型是字符串，无法直接进行数值计算。

  如果变量的值有空格，需要使用双引号或者单引号包起来。

  通过export 变量名可以使变量变为全局变量。

```shell
[root@cloud-host test]# name=sunny
[root@cloud-host test]# echo $name
sunny
```

3. 特殊变量：$n

   $n  功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十个以上的参数，十以上的参数需要用大括号包含，如${10}。

4. 特殊变量：$#

   $#  功能描述：获取所有输入参数个数，常用于循环。

5. 特殊变量：$*、$@

   $*  功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体。

   $@ 功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待。

4. 特殊变量：$?

   $？ 功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令决定），则证明上一个命令执行不正确了。

   ```shell
   [root@cloud-host test]# vim demo.sh
   #!/bin/bash
   echo "$0-$1-$2"
   echo $#
   echo $*
   echo $@
   
   [root@cloud-host test]# sh demo.sh para1 para2
   demo.sh-para1-para2
   2
   para1 para2
   para1 para2
   ```
## 5. 运算式

1. 方式一（推荐）：$((运算式子)) 或 $[运算式子]

   ```shell
      [root@cloud-host ~]# a=$((1+2))
      [root@cloud-host ~]# echo $a
      3
      [root@cloud-host ~]# b=$(((1+3)*4))
      [root@cloud-host ~]# echo $b
      16
      [root@cloud-host ~]# c=$[(1+3)*5]
      [root@cloud-host ~]# echo $c
      20
   ```

2. 方式二：expr 运算式子（expr后有空格，式子中的运算要用空格隔开）

   ```shell
   [root@cloud-host ~]# expr 1 + 2
   3
   [root@cloud-host ~]# e=`expr 1 + 2`
   [root@cloud-host ~]# echo $e
   3
   ```

## 6. 条件判断

1. 语法

   [ condition ] (注意condition前后有空格)

2. 常用判断条件（判断符号要与其他符号有空格隔开）

   - 字符串比较

     = 等于                                   != 不等于

     ```shell
     [root@cloud-host ~]# [ "hi" = "hi" ]
     [root@cloud-host ~]# echo $?
     0
     ```

   - 整数比较

     -lt 小于（less than）          -le 小于等于（less equal）

     -gt 大于（great than）      -ge 大于等于（great equal）

     -eq 等于（equal）              -ne 不等于（not equal）

     ```shell
     [root@cloud-host ~]# [ 2 -gt 3 ]
     [root@cloud-host ~]# echo $?
     1
     [root@cloud-host ~]# [ 2 -lt 3 ]
     [root@cloud-host ~]# echo $?
     0
     ```

   - 文件权限判断

     -r 文件有读权限                   -w 文件有写权限

     -x 文件有执行权限

     ```shell
     [root@cloud-host ~]# [ -x ./test.sh ]
     [root@cloud-host ~]# echo $?
     0
     ```

   - 文件类型判断

     -e 文件存在（但可能是目录或者链接等） -f 文件存在且是一个普通文件

     -d 文件存在且是一个目录

     ```shell
     [root@cloud-host ~]# [ -d ./test.sh ]
     [root@cloud-host ~]# echo $?
     1
     ```
     
   - 变量判空（加双引号）
   
     -z  字符串长度为0则真
     -n  字符串长度不为0则真（是否有值），与-z相反
   
     ```shell
     [root@cloud-host ~]# b=2
     [root@cloud-host ~]# [ -z "$b" ]
     [root@cloud-host ~]# echo $?
     1
     [root@cloud-host ~]# [ -n "$b" ]
     [root@cloud-host ~]# echo $?
     0
     [root@cloud-host ~]# unset b
     [root@cloud-host ~]# [ -z "$b" ]
     [root@cloud-host ~]# echo $?
     0
     [root@cloud-host ~]# [ -n "$b" ]
     [root@cloud-host ~]# echo $?
     1
     ```

## 7. 流程控制

### 1、if判断

- 语法（if后面有空格）

  ```shell
  # 方式1
  if [ condition ];then
  	# do something
  fi
  # 方式2
  if [ condition ]
  then
  	# do something
  elif [ condition ]
  then
  	# do something
  fi
  ```

- 示例

  ```shell
  [root@cloud-host test]# vim if.sh
  #!/bin/bash
  name=sunny
  if [ $1 = $name ]
  then
  	echo "This is $name"
  elif [ $1 != $name ]
  then
  	echo "This is not $name"
  fi
  
  [root@cloud-host test]# sh if.sh tonny
  This is not sunny
  [root@cloud-host test]# sh if.sh sunny
  This is sunny
  ```

### 2、case判断

- 示例

  ```shell
  [root@cloud-host test]# vim case.sh
  #!/bin/bash
  case $1 in
  1)
          echo "1"
  ;;
  2)
          echo "2"
  ;;
  *)
          echo "other"
  ;;
  esac
  
  [root@cloud-host test]# sh case.sh 1
  1
  [root@cloud-host test]# sh case.sh 3
  other
  ```

### 3、for循环

- 方式1

  语法：

  ```shell
  for ((初始值;判断条件;变量变化)) # 括号内可以没有空格，条件判断可以使用<=等
  do
  	# do something
  done
  ```

  计算100累加：

  ```shell
  [root@cloud-host test]# vim for1.sh
  #!/bin/bash
  sum=0
  for ((i=0;i<=100;i++))
  do
          sum=$[$sum+i]
  done
  echo $sum
  
  [root@cloud-host test]# sh for1.sh
  5050
  ```

- 方式2

  语法：

  ```shell
  for 变量 in value1 value2 value3...
  do
  	# do something
  done
  ```

  打印shell脚本所有参数：

  ```shell
  [root@cloud-host test]# vim for2.sh
  #!/bin/bash
  for i in "$*"
  do
          echo "para $i"
  done
  echo "---- split line -----"
  for i in "$@"
  do
          echo "para $i"
  done
  
  [root@cloud-host test]# sh for2.sh 1 2 3 4 5
  para 1 2 3 4 5
  ---- split line -----
  para 1
  para 2
  para 3
  para 4
  para 5
  ```

  $*与$@都是包含所有参数，没有加双引号的时候它们效果一样

  加双引号后："$*"把所有参数当做一个整体，"$@"还是分开对待每一个参数

  

### 4、while循环

语法：

```shell
while [ condition ]
do
	# do something
done
```

计算100累加示例：

```shell
[root@cloud-host test]# vim while.sh
#!/bin/bash
n=1
sum=0
while [ $n -le 100 ]
do
        sum=$[$sum+$n]
        n=$[$n+1]
done
echo $sum

[root@cloud-host test]# sh while.sh
5050
```

## 8. 读取控制台输入

- 语法

  ```shell
  read options 变量
  ```

  options:

  -p 设置输入时的提示符

  -t 设置读取值时的等待时间 

- 示例

  提示5秒内输入值

  ```shell
  [root@cloud-host test]# vim input.sh
  #!/bin/bash
  read -p "Delete this file? yes/no" -t 5 input
  if [ $input = "yes" ]
  then
          echo "yes"
  elif [ $input = "no" ]
  then
          echo "no"
  fi
  
  [root@cloud-host test]# sh input.sh
  Delete this file? yes/noyes
  yes
  ```

## 9. 函数

### 1、系统函数

- basename函数

  用于通过文件路径字符串获取文件名（去掉路径，可选去掉后缀）

  语法：basename filePathStr [suffix]

  ```shell
  [root@cloud-host test]# basename /root/test/demo.sh
  demo.sh
  [root@cloud-host test]# basename /root/test/demo.sh .sh
  demo
  ```

- dirname函数

  获取文件绝对路径中的路径（去掉文件名）

  语法：basename filePathStr

  ```shell
  [root@cloud-host test]# dirname /root/test/demo.sh
  /root/test
  ```

### 2、自定义函数

语法：参数的传递和脚本一样，只能通过$?获取函数返回值，可以通过return n（0-255）进行返回，如果没有return语句则返回值为最后一条命令的执行结果。

```shell
[function] funname[()]
{
	# do something
	[return int;]
}
funname # 函数调用
```

加法计算的函数示例

```shell
[root@cloud-host test]# vim func.sh
#!/bin/bash
function addition()
{
        sum=$[$1+$2]
        return $sum
}
addition 2 3
result=$?
echo $result

[root@cloud-host test]# sh func.sh
5
```

## 10. shell工具

### 1、cut

cut的主要功能是分割。分割文件中每一行的字符并将它们输出。当输出的列处于两个分隔符之间，则此列返回空。

- 语法

  cut [options] fileName

  options说明：

  -f 输出列号，如-f1输出第一列、-f2,3输出第二、三列（可解决分隔符不整齐的情况）

  -d 用于切割的分隔符，只能是单个字符，如-d"-"，-d" "，默认是制表符

- 示例

  ```shell
  [root@cloud-host test]# cat cut.txt
  1 shenzhen
  2 beijing
  3 hangzhou
  [root@cloud-host test]# cut -d" " -f2 cut.txt
  shenzhen
  beijing
  hangzhou
  
  [root@cloud-host test]# echo "1-3"|cut -d'-' -f 2
  3
  ```

  查看Kafka的进程号

  ```shell
  [root@cloud-host test]# jps |grep Kafka|cut -d" " -f1
  20209
  [root@cloud-host test]# pid=`jps |grep Kafka|cut -d" " -f1`
  [root@cloud-host test]# echo $pid
  20209
  ```

### 2、sed

sed是一种流编辑器，每次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，成为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完之后把缓冲区输出到终端（可以通过重定向输出到文件中）。接着处理下一行，知道文件末尾。被处理的文件不会发生改变。

- 语法：

  sed [options] "command" fileName

- options：

  -e 直接在指令列模式上进行sed的动作编辑

- command说明：

  a 新增行，在a前面可以添加参数n代表在第几行新增，a的后面接要新增的字符串，如"2a你好"在第二行后面新增“你好”。如果没有参数n则每一行后面都增加新字符串。

  d 删除，删除包含某个字符串的行，如"/sunny/d"删除包含sunny的行。

  s 查找替换，如"s/sunny/angeya/g"将sunny替换为angeya，g代表全局替换。

- 示例

  数据源sed.txt

  ```shell
  [root@cloud-host test]# cat sed.txt
  sunny love you
  you love sunny
  you are angeya
  ```

  第三行之后增加一行“good morning”

  ```shell
  [root@cloud-host test]# sed "3a good morning" sed.txt
  sunny love you
  you love sunny
  you are angeya
  good morning
  ```

  删除包含“angeya”的行

  ```shell
  [root@cloud-host test]# sed "/angeya/d" sed.txt
  sunny love you
  you love sunny
  ```

  将“love”替换成“hate”

  ```shell
  [root@cloud-host test]# sed "s/love/hate/g" sed.txt
  sunny hate you
  you hate sunny
  you are angeya
  ```

### 3、awk

一个强大的文本分析工具，把文件逐渐行读入，以空格为默认分隔符将每行分割，对分割的部分再进行分析处理。

- 语法

  awk [options] 'pattern1{action1} pattern2{action2}...' fileName

  pattern：表示awk在数据中查找内容（正则表达式），返回符号模式的行

  actions：在找到匹配内容时执行的命令

- option

  -F 指定输入文件拆分分隔符

  -v 赋值一个用户定义变量

- awk内置变量

  FILENAME 文件名

  NR 当前处理（匹配）的行号

  NF 分割之后的列数

- 示例

  数据源（手机和价格）

  ```shell
  [root@cloud-host test]# vim awk.txt
  honor10-2399
  iphone12-5299
  iphone13-5999
  redmiK40-2199
  oppoA73-1999
  ```

  获取以iphone开头的行，输出第一、第一列

  ```shell
  [root@cloud-host test]# awk -F- '/^iphone/{print $1"->"$2}' awk.txt
  iphone12->5299
  iphone13->5999
  ```

  显示第一、二列，使用“:”做分隔符，且在第一行加入列名，结尾加入诺基亚手机（2099元）。

  ```shell
  [root@cloud-host test]# awk -F- 'BEGIN{print "phone:price"} {print $1":"$2} END{print "nokia:2099"}' awk.txt
  phone:price
  honor10:2399
  iphone12:5299
  iphone13:5999
  redmiK40:2199
  oppoA73:1999
  nokia:2099
  ```

  所有手机打五折

  ```shell
  [root@cloud-host test]# awk -F- '{print $1"-"$2*0.5}' awk.txt
  honor10-1199.5
  iphone12-2649.5
  iphone13-2999.5
  redmiK40-1099.5
  oppoA73-999.5
  ```

  统计awk.txt文件名，每行的行号，每行的列数

  ```shell
  [root@cloud-host test]# awk -F- '{print "fileName:" FILENAME",rowNum:" NR "colunms:"NF}' awk.txt
  fileName:awk.txt,rowNum:1colunms:2
  fileName:awk.txt,rowNum:2colunms:2
  fileName:awk.txt,rowNum:3colunms:2
  fileName:awk.txt,rowNum:4colunms:2
  fileName:awk.txt,rowNum:5colunms:2
  ```

  统计一个文件中空行所在的行号

  ```shell
  [root@cloud-host test]# vim emptyLine.txt
  111
  
  2222
  
  333
  
  [root@cloud-host test]# awk '/^$/{print NR}' emptyLine.txt
  2
  4
  ```

### 4、sort

sort命令是在Linux里非常，它将文件进行排序，并将排序结果标准输出。

- 语法：

  sort options filePath

  -n 按照数值大小排序

  -r 相反顺序（默认是升序）

  -t 设置排序时所用的分割字符

  -k 指定需要排序的列

- 示例：

  分别按照手机名称和价格排序

  ```shell
  [root@cloud-host test]# vim sort.txt
  honor10-2399
  iphone12-5999
  redmiK40-2199
  oppoA73-1999
  
  [root@cloud-host test]# sort -t- -nk 1 sort.txt
  honor10-2399
  iphone12-5999
  oppoA73-1999
  redmiK40-2199
  [root@cloud-host test]# sort -t- -nrk 2 sort.txt
  iphone12-5999
  honor10-2399
  redmiK40-2199
  oppoA73-1999
  ```

## 11. 常用shell脚本

### 1. 配置守护进程：

1. 将demo.service文件放入/usr/lib/systemd/system目录
2. 将demo-daemon.sh文件放入/etc/rc.d/init.d目录
3. 增加执行权限：chmod +x /etc/rc.d/init.d/demo-daemon.sh
4. 重新加载系统服务：systemctl daemon-reload
5. 开启守护进程：systemctl start demo
6. 配置守护进程开机启动：systemctl enable demo

demo.service文件

```shell
[Unit]
Documentation=demo-service
 
[Service]
User=root
Group=root
Type=forking
Restart=no
KillMode=process
ExecStart=/etc/rc.d/init.d/demo-daemon.sh

[Install]
WantedBy=multi-user.target
```

demo-daemon.sh文件

```shell
#!/bin/bash
while true; do
RUNNING=`pgrep -f data-export-1.0-SNAPSHOT.jar`
if [ -n "$RUNNING" ];
then
echo "data-export is runing"
else
echo "starting data-export"
/usr/bin/java -jar -Xmx4g /data/data-export/data-export-1.0-SNAPSHOT.jar &
sleep 30
fi
sleep 30 
done &
```

### 2. 查找包含某字符串的文件：

在home路径下查找包含“iphone”的文件

```shell
grep -r "iphone" ~/ | cut -d":" -f1 # 递归查找，通过:分割，输出第一列
```















































