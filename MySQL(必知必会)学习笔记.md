### MySQL 学习笔记

#### 1，了解MySQL

数据库：数据库是一个以某种有组织的方式存储的数据集合，是一个容器，通常形式为一个或一组文件。不过通常我们会把DBMS也称为数据库。

表：一种结构化文件，用来存储某种特定类型的数据。（关系型数据库中一般由行和列组成）。

SQL：结构化查询语言（Structured Query Language）。一种用来与数据库通信的语言。几乎所有重要的DBMS都支持SQL（如SQL Server，Oracle等）；

DBMS：数据库管理系统（Database Management System），对数据库进行管理（增删查改等）的大型软件。

MySQL：一款免费、开源、快速的DBMS。

MySQL基本特点：语句不区分大小写，SELECT 和 select一样。查询中字符串匹配也默认不区分大小写。

登录

```sql
mysql -h127.0.0.1 -P3306 -uroot -p
```

显示数据库/表

```sql
show databases/tables;
```

显示数据库/表创建语句

```sql
show create database/table xxx;
```

使用数据库

```sql
use database xxx;
```

显示用户授权

```sql
show grants;
```

显示服务器错误或警告信息

```sql
show errors/warnnings;
```

MySQL是多用户多进程的，查看命令

```sql
show processlist;
```



#### 2，数据检索

##### 基本查询 select：

​		select用来查询数据，至少指定哪张表的哪些列。还可以用来做判断，结果为true则查询结果为1，否则为0。

`select * from student;`查询 student 表中所有列，在实际开发中，尽量不使用通配符*，因为会影响性能，除非每一个字段都要使用。

`select 2 = 3`，可用于判断 2=3 是否为true。一般调试的时候会用到。

##### 不包含重复行 distinct：

`select distinct teacher_id from student;`查询结果中teacher_id不能包含相同的值，即每个id只出现一次，一般用来统计teacher的数量。distinct关键字放在列的前面即可，但是distinct不能用于部分列。

##### 限制查询 limit：

`select name from student limit 5;`限制最多查询5条结果，不够5条时返回实际结果。

`select name from student limit 0, 1;`从第一条数据开始检索，最多检索一条数据，本例返回第一条数据，一般用于分页查询中。mysql 5.0 之后 limit 子句还可以写成 limit 1 offset 0;

##### 排序 order by：

`select * from student order by id asc;`根据id字段升序排序。升序（默认）：ascending，降序：descending。可以指定多列排序：order by name asc, age desc。首先按照name进行降序排序，如果有多个name相等的情况，才会对age进行降序排序。

##### 过滤 where + 操作符：

`select * from student where age > 20;`查找年龄大于20的学生。过滤出的操作符还有 =，<>（!=），<，>，>=，<=，between...and（包含边界）等

##### 过滤 where + 条件组合 and or in：

and优先级大于or。同时使用and和or时，最好加上优先级括号，提高可读性。

`select * from student where teacher_id = 2 and age > 20;`查找某个老师的20岁以上的学生；

`select * from student where age = 20 or age = 18;`查找年龄为20或者18的学生

`select * from student where age in (18, 20);`使用in实现上面or的功能，in和or两者可以替换，但是根据具体情况来使用，方便就OK。可以使用not in(18, 20)查找age不等于18和20的学生。

##### like + 通配符（ % _ ） 过滤：

如果能够使用上面的其他方法就不要使用通配符，通配符匹配的时间开销比以上其他方法大。

`select * from student where name like '王%'；`%可以匹配任意个任意字符。本例为找出所有姓王的学生。

`select * from student where anme like '张_';`_可以匹配一个任意字符。本例为查找姓张的且名字只有两个字的学生。

##### 正则表达式过滤：

MySql中支持的正则表达式只是全部正则表达式的一部分，且转移字符需要两个反斜杠\\\。匹配的结果只要包含就能匹配，如果想要指定开头或者结尾需要使用字符^和$

`select * from student where name regexp 'du';`可以匹配baidu，duck，product。

##### 创建字段：

使用数据表已有的列进行简单处理创建出需要的字段，这样无需等到应用端处理，速度更快。

`select concat(id, '--', name) as info from student;`查找学生数据时使用concat()函数将id和name拼接并命名为info，因此本例info的值可以为1023--张三。

`select order_id, price * num as cost from orders;`查找订单号，订单的总金额信息。总金额通过单价和数量计算出来。类似的还支持+，-，/ 等操作。

##### 数据处理函数：

每种DBMS的数据处理函数都没有很强的移植性，自己支持的函数DBMS不一定支持，在更换DBMS时需要注意他们之间函数的区别。

文本处理函数



新鲜：soundex()函数可以尝试对对发音处理，

`select * from student where Soundex(name)=Soundex('lee');`本例可以查询到name为 li 的学生。

日期处理函数



直接使用日期字符串查询并不靠谱，如：select * from orders where order_date = '2020-06-08'，因为日期类型为datetime时，2020-06-08 00:00:00 与 2020-06-08不相等，无法匹配出结果。这时使用日期函数就显得很重要。

`select * from orders where Date(order_date) = '2020-06-08';`靠谱地实现了上面需求。

`select * from orders where Date(order_date) between '2020-06-01‘ and '2020-06-30';`可以查询出6月份的订单，但是必须要知道每个月的天数，优化如下。

`select * from orders where Year(order_date) = 2020 and Month(order_date) = 6;`

##### 数据汇总函数：

SQL的汇总函数，（MySQL还支持其他的一些），以下函数都只返回一条记录。



`select avg(age) as avg_age from student;`获取学生的平均年龄。

`select count(distinct(grade) grade_num from student;`获取年级的数量。

`select avg(score) as avg_age from student where grade = 1;`获取一年级学生的平均分数。

`select sum(score) as total, avg(score) as avg from student;`获取所有学生的总分和平局分。

##### 数据分组

在查询结果中可以使用group by进行分组，having进行分组排序。where支持的操作符having都支持。

`select grade, count(grade) from student group by grade;`统计每个年级有多少个学生。根据grade分组，相同的grade是一个组，然后通过count函数统计每个不同grade的数量。

`select grade, count(*) as grade_num from student group by grade having grade_num > 1 order by grade_num;`统计学生数超过1的各个年级的学生数，升序排序（一般不用使用默认排序）。

`select grade, count(grade) as grade_num from student where grade > 1 group by grade having grade_num > 1;`统计1年级以上的，人数超过1人的各个年级的学生人数。

##### 子查询：%%可以将一个查询的结果作为一个查询的筛选条件。但是子查询嵌套太多时会影响性能。

`select * from student where school_id in(select id from university where name='北京大学');`查找在北京大学的所有学生。本例中in可以换做=；子查询中还可以使用子查询。

`select name, (select count(*) from student where student.school_id = university.id) as student_num from university;`查找每个学校包含的学生数。

##### 简单联结（内联结，也叫等值联结）：

联结是数据库查询的重要内容，内联结比外链接更常用。

`select * from university, student where student.school_id = university.id;`查询每一个学生所在的学校，满足条件的行才返回。此方式必须要有where子句，否则查询的结果就是两个表做笛卡尔乘积。当然，可以联结多个表，但是会影响性能。

`select * from university inner join student on student.school_id = university.id;`上一句SQL语句的替代，显式地表达内联结。两种方法都可以。inner可以省略。

##### 高级联结（自联结、自然联结和外部联结）：

自联结

`select s2.* from student s1, student s2 where s1.name='李四' and s1.school_id=s2.school_id;`查找和李四在同一所学校的所有学生。使用别名之后，同一个表可以出现两次了。

自然联结：使得做联结的时候，每列只出现一次。

左外部联结

`select * from university left outer join student on university.id = student.school_id;`查询每一个大学中的学生，没有学生的大学也列出来。左联结保证了左表的每一列都查询出来。右外联结与左外联结类似，也可以通过交换两个表的顺序实现替换。

##### 组合查询

UNION：组合查询操作符，放在多个select语句之间，表示将多个查询的结果（结构须一致）放在一起。虽然where子句也可以实现，但是复杂查询使用UNION可能会简化一些，排序使用最后一个select的排序。使用UNION组合时，MySQL会自动只保留多个select查询到的相同结果中的一个，如果想要相同的结果每个都保留，需要使用UNION ALL关键字。

`select * from student where age > 24 union select * from student where grade > 1 order by id;`查询年龄大于24且年级大于1的学生，按照学生id进行排序，等同于``select * from student where age > 24 or grade > 1 order by id;``

##### 全文搜索

全文搜索目前只有MyISAM引擎支持。

通配符和正则表达式进行文本搜索需要对数据表的每一行进行匹配，因此数据量越大的时候搜索性能越低；而用于全文搜索的列，建立了索引，可以更快地进行匹配关键词，提高搜索效率。

#### 3，插入数据

`insert into student (name, school_id, age, grade) values('中白', 3, 20, 3)，('大白', 3, 21, 4);`将中白、大白两行数据插入到student表，列名中没有给id，MySQL会忽略掉，使用自增的值。into student后面可以不接表名，但是列值需要按照数据表列的顺序给出，而且每个列值都要给出，这种方式在数据表结构改变后就不安全了。

`intsert into student (id, name, school_id, age, grade) select * from student2;`insert select语句，表示将student2中的数据全部插入到student中。

`insert low_priority into student ... ;`在insert和into之间添加low_priority关键字，可以降低插入数据的优先级，因为插入数据涉及到索引更新，性能较低，降低优先级可以保证查询查询得到快速响应。

#### 4，更新与删除数据

MySQL没有撤销操作，索引应该非常小心地使用update和delete，否则搞错了数据没法恢复。

`update student set name = '张仨', age = 23 where name = '张三';`将张三的名字改为张仨，年龄改为23。如果缺少where子句，则对所有用户的姓名和年龄进行修改。在update后面加入ignore关键字，表示更新数据时如果发生错误，则恢复到更新之前的数据。

`update user set role = case when role=1 then 10 when role =2 then 20 else 0 end where id = 2`有条件更新数据库记录。

`delete from student where grade = 4;`删除大四的所有学生。如果没有where子句，将删除所有表中所有数据。如果需要删除所有数据，使用`truncate table student;`会更快，因为它会删除表并重新创建。

#### 5，创建和操作表

创建表：

```mysql
drop table if exists student;
create table if not exists student(
    id int not null auto_increment,
    name varchar(50) not null comment '学生姓名',
    school_id int not null default 1,
    score float(3,2) null,
    primary key(id)
) engine=InnoDB
# ceate 中if not exists去掉，但是如果表存在会报错
# auto_crement：自增，每个表只允许一列，并且必须被索引。如果插入时给了一个合法值，下次会从这个值开始自增
# not null default 1，指定默认值为1
# float(3,2)，浮点数长度为3，精确到小数点后2位
# MyISAM引擎性能高，支持全文搜索，但是不支持事务；InnoDB能够良好支持事务，不支持全文搜索。外键不能用于关联不同引擎的表
# 整数中的位数指的是数值的位数，如998为3位
```

更新表：

一般在表创建好后不修改表，所以前期需要好好设计表结构。

``alter table student add score float(3,2);`添加score列

``alter table student drop column score;`删除score列

删除表：

`drop table student;`删除student表，删除后不能恢复，一般建议做好备份。

重命名表：

`rename table student to student2, teacher to teacher2;`

#### 6，视图

视图可以理解为一个虚拟表，它们包含的不是数据而是需要检索数据的查询，也就是对select语句的封装，可以简化数据处理。一般视图只用来查询，不用来更新。

创建视图：`create view sttdent_info as select s.name, u.name from student s left outer join university u on u.id = s.school_id;`创建一个查询学生姓名和所在学校的视图。

使用视图：

`select * from student_info;`视图的使用方式与表类似。

删除视图：

`drop view student_info;`

#### 7，存储过程

存储过程将处理封装在容易使用的处理单元中，简化复杂的操作，同时可以提高性能，类似于函数。

存储过程的执行，`call get_stu_age(@min_age, @avg_age, 5);`调用名为get_stu_avg_age 的存储过程，全局变量都需要使用@开头，一般情况下：全局变量为出参，具体值为。查询全局变量语句`select @min_age;`；查看存储过程的语句`show create procedure get_stu_age;`；存储过程内部可以根据入参进行分支判断等操作，可以满足多种应用场景。

创建存储过程get_stu_age()

```mysql
delimiter //
create procedure get_stu_age(out min_age int, out avg_age float, in id_ int)
begin
	select min(age) into min_age from student where id > id_;
	select avg(age) into avg_age from student where id > id_;
end //
delimiter ;
# delimiter // 表示设置mysql的结束符为 //，如果不这么做，存储过程内部的 ; 会被认为是结束符，导致发生语法错误。end // 表示存储过程结束位置，delimiter ; 将mysql结束符重新设为;
# 存储过程仅支持真实表的操作，不支持视图（虚拟表）
```

删除存储过程

`drop procedure get_stu_age;`

游标：游标用于在存储过程中一行一行地读取数据。

#### 8，触发器

触发器在对表的操作前或后出发相应操作，触发器只能响应delete，insert，update操作。每张表每个操作最多可以定义两个触发器，before和after。如果触发器的语句执行失败，会导致之后的语句也不能被正常执行。

创建触发器：

`create trigger add_student after insert on student for each row select 'student added' into @stu_add;`定义一个add_student触发器，在student表每增加一行的时候都将字符串‘student added’赋值给@stu_add变量。

删除触发器：

`drop triggeradd_student;`

#### 管理事务处理

事务：原子性，持久性，

开启事务`start transaction;`到提交事务`commit;`之前的SQL语句，如果发生错误，则不会提交（自动撤销`rollback;`，rollback只能在一个事务内使用）。一般的的SQL语句都是直接针对数据库表执行和编写的。提交是隐式的（自动进行的）。

```mysql
start transaction;
delete from student where id = 2;
delete from counse where id = 2;
commit;
# 开除一个学生的时候，也需要把他所选的课程删除掉，如果其中一个失败了，都不提交，会撤销到事务开始之前
```

部分提交：有时候我们虽然开启了事务，但是不想回退到事务开始之前，可以设置若干保留点`savapoint delete1;`，以便可以回退到这些保留点`rollback to delete1`。

事务和保留点会在rollback或者commit之后自动关闭。

#### 字符集与排序规则

字符集：设置字符编码，为了适应不同的语言，一般使用utf-8编码的字符集。

排序规则：某一字符集下不同字符的比较队则。_ci（case insensitive）大小写不敏感，_cs（case sensitive）大小写敏感，而且不同的排序规则的比较顺序和准确性也不一样，utf8字符集一般使用默认编码utf8_general_ci排序队则。

字符集和排序规则可以对数据库指定，数据表指定甚至表列也可以指定。

```mysql
create table person(
	id int not null auto_increment,
    name varchar(30) not null,
    primary key(id)
) default character set utf8
collate utf8_general_ci;
# 如果创建表时没有指定字符集和排序规则，则使用数据库默认的字符集和排序规则
```

#### 用户和权限管理

MySQL是支持多用户的，在现实中最好不要直接使用root账户，应该创建一些用户并给定相应权限。

创建用户：

`create user sunny indentified by password 'sunny';`创建一个用户名密码都为sunny的用户。但是这样一般会报错，因为使用的明文密码不符合hash值的类型，这里需要使用`select passwrd('sunny')`将明文密码加密，然后使用加密后的密码创建用户，也可以先创建用户`create user sunny;`不设密码，然后再修改密码。刚创建的用户除了登录权限之外，是没有其他权限的，登录只能看到一个information_schema库。

用户重命名：`rename user sunny to angeya;` 将sunny重命名为angeya

修改密码：`set password for sunny = Password(‘sunny2’);`将密码改为sunny2

删除用户：`drop user sunny;`删除sunny用户

设置访问权限：

`grant select on mybatis_demo.* to sunny;`给sunny用户授予mybatis_demo库所有表的select权限。撤销该权限只需将语句的`grant`改为`revoke`即可。

`show grants for sunny;`查看sunny用户的所有权限。

#### 转储与导入sql文件

使用命令行比图形化界面速度快很多

```mysql
# 转储到sql文件 mysqldump [-h地址] [-u用户] [-p密码] 库名 [表名] > sql文件路径
mysqldump -h 127.0.0.1 -uroot -proot book_sharing user > /user.sql


```

#### flush hosts

如果一个ip连接数据库失败太多次，并且超过了允许次数，数据库服务器会记录该ip，不让其进行连接。解决办法有两种：

1，增加最大失败连接数

2，使用flush hosts命令刷新ip记录



#### 数据类型

​		定长则每个值都分配相同长度的内存空间，变长会根据实际的值长度分配空间。那么为什么需要定长呢？因为性能，定长列的处理比变长列的处理快的多！

字符串类型：

| 类型       | 说明                                                 |
| ---------- | ---------------------------------------------------- |
| char       | 1-255个字符的定长串。创建时不指定长度，默认为char(1) |
| enum       | 接受最多64K个字符串组成的一个预定义集合的某个串      |
| text       | 最大支持长度为64K的变长文本                          |
| mediumtext | 与text相同，最大支持长度为16K                        |
| longtext   | 与text相同，最大支持长度为64G                        |
| tinytext   | 与text相同，最大支持长度为255Byte                    |
| set        | 接受最多64个字符串组成的一个预定义集合的某个串       |
| varchar    | 长度可变，0-255                                      |

​		数值类型默认都是有符号的，可以声明为无符号（unsigned），最大值可增长一倍

数值类型：

| 类型      | 说明                  |
| --------- | --------------------- |
| bit       | 位字段，1-64位        |
| bigint    | 64位，与java long类似 |
| int       | 32位，与java int类似  |
| mediumint | 24位                  |
| smallint  | 16位                  |
| tinyint   | 8位，与java byte类似  |
| boolean   | 布尔标志，0或者1,     |
| real      | 4字节的浮点数         |
| float     | 32位，类似java float  |
| double    | 64位，类似java double |
| decimal   | 精度可变的浮点数      |

时间日期类型：

| 类型      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| date      | 格式为YYYY-MM-DD                                             |
| time      | 格式为HH:MM:SS                                               |
| datetime  | date和time的组合                                             |
| timestamp | 功能与datatime相同，但范围较小（1970.01.01之后）             |
| year      | 用2位数表示，范围70（1970）- 69（2069），4位数表示，范围1901-2155 |

二进制数据类型：

| 类型       | 说明                |
| ---------- | ------------------- |
| tinyblob   | Blob最大长度255字节 |
| blob       | Blob最大长度64K     |
| mediumblob | Blob最大长度16MB    |
| longblob   | Blob最大长度4GB     |



