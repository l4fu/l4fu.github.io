**目录**

常见共享命令

IPC$

IPC$的利用条件

1：开启了139、445端口

2：目标主机开启了IPC$共享

3：IPC连接报错

IPC空连接 

空连接可以做什么?(毫无作用)

IPC$非空连接

IPC$非空连接可以做什么？

dir命令(查看文件和目录)

tasklist命令(查看进程)

at命令(计划命令，可反弹shell)

schtasks(计划任务)

Impacket中的atexec.py

关闭IPC$共享及其他共享

IPC$连接失败的原因及常见错误号

连接失败原因

常见错误号

***

## 常见共享命令  

```
net use                               #查看本机建立的连接(本机连接其他机器)
```

  

## IPC$

**IPC$** (Internet Process Connection) 是共享“命名管道”的资源，它是为了让进程间通信而开放的命名管道，通过提供可信任的用户名和口令，连接双方可以建立安全的通道并以此通道进行加密数据的交换，从而实现对远程计算机的访问。IPC$是NT2000的一项新功能，它有一个特点，即在同一时间内，两个IP之间只允许建立一个连接。NT2000在提供了 IPC$ 共享功能的同时，在初次安装系统时还打开了默认共享，即所有的逻辑共享(C$、D$、E$……)和系统目录共享(Admin$)。所有的这些初衷都是为了方便管理员的管理。但好的初衷并不一定有好的收效，一些别有用心者会利用IPC$，访问共享资源，导出用户列表，并使用一些字典工具，进行密码探测。

为了配合IPC共享工作，Windows操作系统（不包括Windows 98系列）在安装完成后，自动设置共享的目录为：C盘、D盘、E盘、ADMIN目录（C:\\Windows）等，即为ADMIN$、C$、D$、E$等，但要注意，这些共享是隐藏的，只有管理员能够对他们进行远程操作。

输入 net share 可以查看开启的共享。

  

输入 net share 可以查看开启的共享。

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

所有的共享都依赖于139或445端口。

## IPC$的利用条件

### 1：开启了139、445端口

首先我们来了解一些基础知识：

*   SMB: (Server Message Block) Windows协议族，用于文件打印共享的服务；
    
*   NBT: (NETBios Over TCP/IP)使用137（UDP）138（UDP）139（TCP）端口实现基于TCP/IP协议的NETBIOS网络互联。
    
*   在WindowsNT中SMB基于NBT实现，即使用139（TCP）端口；而在Windows2000中，SMB除了基于NBT实现，还可以直接通过445端口实现
    

**对于win2000客户端（发起端）来说：**

*   如果在允许NBT的情况下连接服务器时，客户端会同时尝试访问139和445端口，如果445端口有响应，那么就发送RST包给139端口断开连接，用455端口进行会话，当445端口无响应时，才使用139端口，如果两个端口都没有响应，则会话失败；
    
*   如果在禁止NBT的情况下连接服务器时，那么客户端只会尝试访问445端口，如果445端口无响应，那么会话失败。
    

**对于win2000服务器端来说：**

*   如果允许NBT, 那么UDP端口137, 138, TCP 端口 139, 445将开放（LISTENING）；
    
*   如果禁止NBT，那么只有445端口开放。
    

我们建立的IPC会话对端口的选择同样遵守以上原则。显而易见，如果远程服务器没有监听或端口，会话对端口的选择同样遵守以上原则。显而易见，如果远程服务器没有监听139或445端口，IPC会话是无法建立的。

### 2：目标主机开启了IPC$共享

默认共享是为了方便管理员进行远程管理而默认开启的，包括所有的逻辑盘(C$、D$等)和系统目录 winnt 或 windows(admin$)以及IPC$。这些共享默认是开启的。可以使用net share命令查看这些共享是否开启。

### 3：IPC连接报错

如果目标主机没有开放139或445端口，我们去使用IPC$连接的话，会提示找不到网络名。

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

IPC空连接  

在介绍空会话之前，我们有必要了解一下一个安全会话是如何建立的。在Windows NT中，是使用 NTLM挑战响应机制认证。传送门——>  NTLM认证方式(工作组环境中)

空会话是在没有信任的情况下与服务器建立的会话（即未提供用户名与密码）。那么建立空会话到底可以做什么呢？  
利用IPC$，黑客甚至可以与目标主机建立一个空的连接，而无需用户名与密码(当然,对方机器必须开了IPC$共享,否则你是连接不上的)，而利用这个空的连接，连接者还可以得到目标主机上的用户列表(不过负责的管理员会禁止导出用户列表的)。建立了一个空的连接后,黑客可以获得不少的信息(而这些信息往往是入侵中必不可少的),访问部分共享,如果黑客能够以某一个具有一定权限的用户身份登陆的话,那么就会得到相应的权限。

**建立IPC$空连接**

```
建立IPC空连接
```

  

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 空连接可以做什么?(毫无作用)

在Windows2003以后，空连接什么权限都没有，也就是说并没有太大实质的用处。有些主机的 Administrator 管理员的密码为空，那么我们可以尝试使用下面的命令进行连接，但是大多数情况下服务器都阻止了使用空密码进行连接。

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

以前建立空会话可以获取一些有用的信息，但是现在空会话的权限很低，访问都被拒了

![图片](_v_images/382184917235862.webp)

## IPC$非空连接

```
建立IPC$非空连接
```

  

![图片](_v_images/381134917226107.webp)

## IPC$非空连接可以做什么？

*   使用管理员组内用户(administrator或其他管理员组内用户均可)建立IPC$连接，可以执行以下所有命令。
    
*   使用普通用户建立IPC$连接，仅能执行查看时间命令：net time \\192.168.10.131 ，其他命令均执行不了。
    

### dir命令(查看文件和目录)

![图片](_v_images/380084917214942.webp)

也可以直接在文件管理用命令：\\192.168.10.131\\c$ 查看对应的文件及目录，也可以增删改查  

![图片](_v_images/379034917211681.webp)

### tasklist命令(查看进程)

```
tasklist /S 192.168.10.131 /U administrator -P 密码
```

  

![图片](_v_images/377994917229341.webp)

### at命令(计划命令，可反弹shell)

*   查看目标系统时间：net time \\192.168.10.131
    
*   将本目录下的指定文件复制到目标系统中：copy vps.exe \\192.168.10.131\\c$
    
*   使用at创建计划任务：at \\192.168.10.131 17:00:00 C:\\vps.exe
    
*   清除at记录：at \\192.168.10.131 作业ID /delete
    
*   使用at命令执行，将执行结果写入本地文本文件，再使用type命令查看该文件的内容：at \\192.168.10.131 17:00:00 cmd.exe /c "ipconfig > C:/1.txt "
    
*   查看生成的1.txt文件：type \\192.168.10.131\\C$\\1.txt
    

  

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### schtasks(计划任务)

Windows Vista、Windows Server 2008及之后版本的操作系统已经弃用at命令，而转为用schtasks命令。schtasks命令比 at 命令更灵活。在使用schtasks命令时，会在系统中留下日志文件：C:\\Windows\\Tasks\\SchedLgU.txt

```
在目标主机上创建一个名为test的计划任务，启动程序为C:\vps.exe，启动权限为system，启动时间为每隔一小时启动一次
```

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### Impacket中的atexec.py

Impacket中的atexec.py脚本，就是利用定时任务获取权限，该脚本的利用需要开启ipc$共享。这个脚本仅工作Windows>=Vista的系统上。这个样例能够通过任务计划服务（Task Scheduler）来在目标主机上实现命令执行，并返回命令执行后的输出结果 。

```
 ./atexec.py  xie/hack:x123456./@192.168.10.130  whoami
```

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

关闭IPC$共享及其他共享

既然ipc$有一定的危险性，而且对于我们大多数人来说是没啥用的，所以我们执行以下命令关闭共享

1、使用命令关闭：

```
net  share  ipc$    /delete              关闭ipc默认共享
```

  

![图片](_v_images/367934917215002.webp)

2、修改注册表关闭

限制IPC$缺省共享：

*   HKEY\_LOCAL\_MACHINE/SYSTEM/CurrentControlSet/Control/Lsa
    
*   Name：restrictanonymous
    
*   Type：REG\_DWORD
    
*   Value：0x0(缺省) 0x1 匿名用户无法列举本机用户列表 0x2 匿名用户无法连接本机IPC$共享 说明:不建议使用2，否则可能会造成你的一些服务无法启动，如SQL Server。
    

## IPC$连接失败的原因及常见错误号

### 连接失败原因

*   用户名或密码错误
    
*   目标主机没有开启IPC$共享
    
*   不能成功连接目标主机的139、445端口
    
*   命令输入错误
    

### 常见错误号

*   错误号5：拒绝访问
    
*   错误号51：Windows无法找到网络路径，及网络中存在问题
    
*   错误号53：找不到网络路径，包括IP地址错误、目标未开机、目标的lanmanserver服务未启动，目标防火墙过滤了端口
    
*   错误号67：找不到网络名，包括 lanmanworkstation 服务未启动，IPC$已被删除
    
*   错误号1219：提供的凭据与已存在的凭据集冲突。例如已经和目标建立了IPC$连接，需要在删除后重新连接
    
*   错误号1326：未知的用户名或错误的密码
    
*   错误号1792：试图登录，但是网络登录服务没有启动，包括目标NetLogon服务未启动(连接域控制器时会出现此情况)
    
*   错误号2242：此用户的密码已经过期。
    


  
         Windows系统安全|135、137、138、139和445端口