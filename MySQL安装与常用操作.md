# MySQL安装与常用操作

 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#mysql "Permanent link")

## 【目的】 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#_1 "Permanent link")

让学员通过该实验的练习主要掌握：

*   了解MySQL的相关知识
*   掌握MySQL的安装，并完成安装过程中的相关配置
*   熟练掌握MySQL中常用操作

## 【环境】 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#_2 "Permanent link")

*   操作机：Windows10

## 【工具】 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#_3 "Permanent link")

*   MySQL

## 【原理】 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#_4 "Permanent link")

### 一、 MySQL数据库简介

  MySQL是一种开放源代码的关系型数据库管理系统（RDBMS），MySQL数据库系统使用最常用的数据库管理语言--结构化查询语言（SQL）进行数据库管理。具有体积小、速度快、总体拥有成本低，并且开放源码的特点。

### 二、 MySQL常用操作语法

  （1） 查看数据库支持的存储引擎

语法格式：mysql>slow engines;

  （2） 查看表的结构等信息

语法格式：mysql>Desc\[ribe\] tablename; //查看数据表的结构。
         mysql>show table status like ‘tablename’;//显示表的当前状态值。
         mysql>show table status like ‘tablename’\\G;//显示表的当前状态值。

  （3） 修改数据表存储引擎

语法格式：mysql>alter table tableName engine =engineName;

  （4） 创建数据库

语法格式：mysql>create database 数据库名;

  （5） 创建数据库表时设置存储引擎

语法格式：mysql>Create table tableName(
          columnName(列名1)  type(数据类型)  attri(属性设置),
          columnName(列名2)  type(数据类型)  attri(属性设置),
          ……) engine = engineName;

  （6） 显示数据库

语法格式：mysql>show databases;

  （7） 显示数据库中的表

语法格式：mysql>use数据库名;
         mysql>show tables;

  （8） 查看MySQL数据库服务器和数据库MySQL字符集

语法格式：mysql>show variables like 'character\_set\_%';

  （9） 查看MySQL数据表（table）的MySQL字符集

语法格式：mysql>show table status from 库名  like ‘%表名%';

  （10） 查看MySQL数据列（column）的MySQL字符集

语法格式：mysql>show full columns from 表名;

  （11） 显示数据库中表的结构

语法格式：mysql>describe 表名;

  （12） 建立数据库与表

语法格式：mysql>use 库名；
         mysql>create table 表名(列属性);

  （13） 删库及表

语法格式：mysql>drop database 库名; 
         mysql>drop table 表名；

  （14） 增加记录

语法格式：mysql>insert into 表名 values(value1,value2,…);

  （15） 修改记录

语法格式：mysql>update 表名 set 列属性值='value' where 列属性值='valude0';

  （16） 删除记录

语法格式：mysql>delete from表名where列属性值='value’;

  （17） 查询记录

语法格式：mysql>select \* from 表名 where列属性值='value';

## 【步骤】 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#_5 "Permanent link")

1.打开桌面【C:\\tools\\MySQL的安装与常用操作】找到MySQL安装攻击，点击开始安装。

![](_v_images/36931418238489.png)

2.选择安装类型，这里选择【Server only】，点击【Next】继续。

![](_v_images/31311418251149.png)

3.点击【Execute】开始安装。

![](_v_images/29091418252138.png)

4.之后进行数据库的配置，可以更改端口，默认为3306，点击【Next】继续。

![](_v_images/27061418244627.png)

5.输入“Root”用户的密码，这个密码需要记住，之后需要使用，点击【Next】继续。

![](_v_images/24861418249519.png)

6.最后点击【Execute】使所有的配置生效。

![](_v_images/23901418230710.png)

7.搜索程序“MySQL”打开“MySQL 5.6 Command Line Client”,打开MySQL命令行。

![](_v_images/22811418237851.png)

8.打开MySQL命令行，输入之前设置的密码。

![](_v_images/21661418247115.png)

9.接下来可以执行SQL语句，输入：`select version();`查看MySQL版本信息。

![](_v_images/19601418246483.png)

10.输入：`show databases;`查看当前存在的数据库。

![](_v_images/18571418240728.png)

11.新创建一个数据库，名为“mytest”：`create database mytest;`。

![](_v_images/17541418222685.png)

12.输入：`use mytest`切换到新建的数据库，创建一个名为“students”的表，包含“id”，“name”，“sex”，“birthday”四个字段：

mysql>create table students (
       id int auto\_increment not null primary key, 
       name char(20),
       sex char(2),
       birthday date);

![](_v_images/16251418232354.png)

13.输入：`show tables`查看当前数据库中的所有表，可以看到刚新建的“students”表。

![](_v_images/14991418235587.png)

14.输入：`describe students`查看“students”表的信息。

![](_v_images/13761418242542.png)

15.如果数据库不支持中文字符，可以输入`alter database mytest character set utf8;`讲编码格式设置为utf8。

![](_v_images/12651418226263.png)

16.在“students”表中添加一条记录：`insert into students valuse('1','张三','男','1971-10-01');`.（若 中文插入有问题 可以输入 ：`alter table students convert to character set utf8;`）

![](_v_images/11681418240711.png)

17.同样的方式添加另一条记录：`insert into students valuse('2','白云','女','1972-05-20');`。

![](_v_images/10541418240534.png)

18.输入：`select * from students;`查看“students”表中的所有记录。

![](_v_images/9621418239532.png)

19.也可以更新表中的某一条记录，输入`update students set bithday='1971-01-10' where name='张三';`将学生张三的出生日期修改。

![](_v_images/8371418238237.png)

20.输入：`select * from students where name='张三';`查看更改后的记录信息。

![](_v_images/7361418233198.png)

21.使用delete命令删除记录：`delete from students where name='张三';`可以看到该条记录已被删除。

![](_v_images/6331418221108.png)

22.使用drop命令可以删除表和数据库：

drop table students;
drop database mytest;

![](_v_images/4201418220969.png)

## 【总结】 (http://study.ctfcaict.com/media/course/html/16/bd1b9ebd-04ec-4905-837c-5dd925572404.html#_6 "Permanent link")

  本实验主要介绍MySQL的安装，讲解MySQL数据库的相关操作和配置方法。介绍数据库及数据库表的创建、删除、更新等常用操作，让学习者更好的了解MySQL的相关知识以及MySQL的安装与常用操作。