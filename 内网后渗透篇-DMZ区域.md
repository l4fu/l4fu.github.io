# 内网后渗透篇:DMZ区域

前言：内网后渗透第一篇，DMZ区域

**1.DMZ区域**

当我们拿到首个权限的时候，我们一般都在DMZ区域，什么是DMZ区域呢？

DMZ是英文“demilitarized zone”的缩写，中文名称为“隔离区”，也称“非军事化区”。它是为了解决安装防火墙后外部网络的访问用户不能访问内部网络服务器的问题，而设立的一个非安全系统与安全系统之间的缓冲区。**DMZ介于外网和内网之间，一般同时拥有一个或者多个内网IP和外网IP。**    

![](images/2020_05_28/Intranet/15906550653054.jpg)


**2.HOW TO DO?**

DMZ作为内网的"入口",普遍部署的防御设备比内网其他区域要多。同时DMZ作为我们窥探内网的窗口，我们内网漫游的开始，我们该怎么做呢？

**2.1查看当前有没有管理员在线**

往往这个步骤会被大家所遗忘,但是这是非常有必要的。

linux命令w，Windows命令 query user

![](images/2020_05_28/Intranet/15906550806370.jpg)


尽量不要跟管理员同时操作，尽管webshell或者远程执行命令的动静不怎么大，但是因为各种情况都可能发生，为了不让自己好不容易搞下来的权限丢了，需要谨慎再谨慎。当然一些特殊情况，无法避免的时候该干还是干！！！

**2.2探测防御设备**

通过tasklist/ps/netstat等命令查看进程，端口，或者通过查看一些目录名来判断已控的设备开启了哪些杀毒软件和可能有的防御设备（HIDS，流控，日志等），再根据设备进行指定绕过。

**2.3信息收集**

信息收集不管在内网和外网都是一项必备的技能，如何更好地进行信息收集呢？

Windows：

* 1.各类密码（点击可跳至文章）
* 2.当前与被控主机所连接的服务器,ARP缓存，ip信息，路由信息目前服务器所在位置（tracert）
* 3.日志，历史登陆记录等
* 4.当前服务器所特有的信息，如是web服务器是否连接数据库，数据库连接串是啥？或者POP3服务器，邮件存哪了,能否直接被读取？

linux:

Linux不同于Windows，Linux的信息收集就简单多了，网上也有很多脚本，这边根据环境直接来用就行了。

sh:

https://github.com/rebootuser/LinEnum.git

https://github.com/pentestmonkey/unix-privesc-check.git

python:

http://www.securitysift.com/download/linuxprivchecker.py

Perl:

https://github.com/PenturaLabs/Linux_Exploit_Suggester.git

**2.4后门留存**

为了不把鸡蛋放在一个篮子里面，当我们拿到一个权限的时候可以多加几个后门，如：

* 1.把webshell写进源码，然后拼接（记得更改文件时间）
* 2.在图片里插马，然后在源码里写包含
* 3.LinuxHOOK命令，Windows ATT&CK的后门姿势集

**2.5痕迹清除**

Linux 的history，last,lastlog，w

https://github.com/re4lity/logtamper（删除脚本）

Windows的安全日志（单条），最近文件等信息

https://3gstudent.github.io/3gstudent.github.io/渗透技巧-Windows日志的删除与绕过/