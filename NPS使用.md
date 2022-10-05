# NPS使用
下载地址：[https://github.com/ehang-io/nps/releases](https://github.com/ehang-io/nps/releases)  
文档说明：[https://ehang-io.github.io/nps/#/?id=nps](https://ehang-io.github.io/nps/#/?id=nps)

# nps简介

nps是一款轻量级、高性能、功能强大的内网穿透代理服务器。与ngrok、frp等老牌内网穿透工具相比，nps可以算是一匹黑马。

其优势主要有两点：一是强大的网页管理面板，nps可以在服务端通过网页管理所有用户行为以及映射记录；二是它集成了多种协议，包括tcp/udp隧道，socks5以及p2p，可以满足多种需求。  
在渗透测试方面，支持<mark>无配置文件模式</mark>，方便进行内网探测。

# Nps启动

NPS分为服务端和客户端

下载地址：[https://github.com/ehang-io/nps/releases](https://github.com/ehang-io/nps/releases)

## 服务端

```bash
执行安装命令
    对于linux|darwin sudo ./nps install
    对于windows，管理员身份运行cmd，进入安装目录 nps.exe install
启动
    对于linux|darwin sudo nps start
    对于windows，管理员身份运行cmd，进入程序目录 nps.exe start
    安装后windows配置文件位于 C:\Program Files\nps，linux和darwin位于/etc/nps
停止和重启可用，stop和restart
    如果发现没有启动成功，可以使用nps(.exe) stop，然后运行nps.(exe)运行调试，或查看日志(Windows日志文件位于当前运行目录下，linux和darwin位于/var/log/nps.log)
访问web服务端：
    访问服务端ip:web服务端口（默认为8080）
    使用用户名和密码登陆（默认admin/123，正式使用一定要更改）
创建客户端
``` 

## 客户端

```bash
点击web管理中客户端前的+号，复制启动命令
执行启动命令，linux直接执行即可，windows将./npc换成npc.exe用cmd执行
Tcp代理
``` 

# 统一准备工作（必做）

```bash
开启服务端，假设公网服务器ip为1.1.1.1，配置文件中bridge_port为8024，配置文件中web_port为8080
访问web客户端1.1.1.1:8080
在客户端管理中创建一个客户端，记录下验证密钥
内网客户端运行（windows使用cmd运行加.exe）
./npc -server=1.1.1.1:8024 -vkey=客户端的密钥
注意：运行服务端后，请确保能从客户端设备上正常访问配置文件中所配置的bridge_port端口，telnet，netcat这类的来检查
``` 

# tcp隧道

```bash
适用范围： ssh、远程桌面等tcp连接场景
假设场景： 想通过访问公网服务器1.1.1.1的8001端口，连接内网机器10.1.50.101的22端口，实现ssh连接
使用步骤
在刚才创建的客户端隧道管理中添加一条tcp隧道，填写监听的端口（8001）、内网目标ip和目标端口（10.1.50.101:22），保存。
访问公网服务器ip（1.1.1.1）,填写的监听端口(8001)，相当于访问内网ip(10.1.50.101):目标端口(22)，例如：ssh -p 8001 root@1.1.1.1
``` 

eg:在其客户端位置处，创建客户端  
主要包括Basic 认证用户名(test)，Basic 认证密码(test)，唯一验证秘钥(123456)  
其中用户名、密码为连接Socks5、Web、HTTP转发代理使用的账号密码，唯一验证秘钥为veky参数，是服务端和客户端连接的凭证。  

[![](_v_images/211523920226024.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105719208-154307225.png)

在控制面板创建tcp隧道  
Tcp隧道为:  
1、服务器监听端口:7777  
2、目标ip和端口：内网ip 192.168.30.130和端口3389  

[![](_v_images/209483920244783.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105729301-188579246.png)

在靶机ip上执行npc客户端命令  
npc.exe -server=172.100.30.173:8024 -vkey=123456  

[![](_v_images/207423920247281.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105735543-1714767012.png)

想通过访问公网服务器172.100.30.173的7777端口，连接内网机器192.168.30.130的3389端口，实现rdp连接。

# Socks5代理

```bash
适用范围： 在外网环境下如同使用vpn一样访问内网设备或者资源
假设场景： 想将公网服务器1.1.1.1的8003端口作为socks5代理，达到访问内网任意设备或者资源的效果
使用步骤
在刚才创建的客户端隧道管理中添加一条socks5代理，填写监听的端口（8003），保存。
在外网环境的本机配置socks5代理(例如使用proxifier进行全局代理)，ip为公网服务器ip（1.1.1.1），端口为填写的监听端口(8003)，即可畅享内网了
``` 

eg:  
利用上面设置的客户端，在控制面板设置socks5代理，设置服务器监听端口12345  

[![](_v_images/205363920249677.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105744843-98864340.png)  

[![](_v_images/203303920231797.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105751156-1636059833.png)

运行npc客户端：npc.exe -server=172.100.30.173:8024 -vkey=123456  
利用proxifer代理工具进行socks5连接  
协议 ip 端口 用户名 密码  
socks5 172.100.30.173 12345 test test  

[![](_v_images/201273920236043.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105759589-193659493.png)

# http正向代理

```bash
适用范围： 在外网环境下使用http正向代理访问内网站点
假设场景： 想将公网服务器1.1.1.1的8004端口作为http代理，访问内网网站
使用步骤
在刚才创建的客户端隧道管理中添加一条http代理，填写监听的端口（8004），保存。
在外网环境的本机配置http代理，ip为公网服务器ip（1.1.1.1），端口为填写的监听端口(8004)，即可访问了
注意：对于私密代理与p2p，除了统一配置的客户端和服务端，还需要一个客户端作为访问端提供一个端口来访问
``` 

eg:  
利用上面设置的客户端，在控制面板设置http正向代理，设置服务器监听端口6666  

[![](_v_images/199223920239488.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105810389-1227929788.png)

运行npc客户端：npc.exe -server=172.100.30.173:8024 -vkey=123456  
利用proxifer代理工具进行http正向代理连接

# 私密代理

```bash
适用范围： 无需占用多余的端口、安全性要求较高可以防止其他人连接的tcp服务，例如ssh。
假设场景： 无需新增多的端口实现访问内网服务器10.1.50.2的22端口
使用步骤
在刚才创建的客户端中添加一条私密代理，并设置唯一密钥secrettest和内网目标10.1.50.2:22
在需要连接ssh的机器上以执行命令
./npc -server=1.1.1.1:8024 -vkey=vkey -type=tcp -password=secrettest -local_type=secret
如需指定本地端口可加参数-local_port=xx，默认为2000
注意： password为web管理上添加的唯一密钥，具体命令可查看web管理上的命令提示
假设10.1.50.2用户名为root，现在执行ssh -p 2000 root@127.0.0.1即可访问ssh
``` 

[![](_v_images/197163920246819.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201105900357-29598482.png)

需要进行客户端服务端连接:  
npc.exe -server=172.100.30.173:8024 -vkey=123456  
然后在需要连接ssh的机器上以执行命令  
npc.exe -server=172.100.30.173:8024 -vkey=123456 -type=tcp -password=888888 -local\_type=secret  

[![](_v_images/195123920226653.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201110003343-1561286865.png)

# p2p服务

```bash
适用范围： 大流量传输场景，流量不经过公网服务器，但是由于p2p穿透和nat类型关系较大，不保证100%成功，支持大部分nat类型。nat类型检测
假设场景：
想通过访问使用端机器（访问端，也就是本机）的2000端口---->访问到内网机器 10.2.50.2的22端口
使用步骤
在nps.conf中设置p2p_ip（nps服务器ip）和p2p_port（nps服务器udp端口）
注：若 p2p_port 设置为6000，请在防火墙开放6000~6002(额外添加2个端口)udp端口
在刚才创建的客户端中添加一条p2p代理，并设置唯一密钥p2pssh
在使用端机器（本机）执行命令
./npc -server=1.1.1.1:8024 -vkey=123 -password=p2p ssh -target=10.2.50.2:22
如需指定本地端口可加参数-local_port=xx，默认为2000
注意： password为web管理上添加的唯一密钥，具体命令可查看web管理上的命令提示
假设内网机器为10.2.50.2的ssh用户名为root，现在在本机上执行ssh -p 2000 root@127.0.0.1即可访问机器2的ssh，如果是网站在浏览器访问127.0.0.1:2000端口即可。
``` 

[![](_v_images/193073920238786.png)](https://img2020.cnblogs.com/blog/2224145/202012/2224145-20201201110017448-1787239174.png)

# 增强模式：通过代理连接nps

```bash
有时候运行npc的内网机器无法直接访问外网，此时可以可以通过socks5代理连接nps
对于配置文件方式启动,设置
[common]
proxy_url=socks5://111:222@127.0.0.1:8024Copy to clipboardErrorCopied
对于无配置文件模式,加上参数
-proxy=socks5://111:222@127.0.0.1:8024Copy to clipboardErrorCopied
支持socks5和http两种模式
即socks5://username:password@ip:port
``` 

# 注册到系统服务(开机启动、守护进程)

```bash
对于linux、darwin
    注册：sudo ./npc install 其他参数（例如-server=xx -vkey=xx或者-config=xxx）
    启动：sudo npc start
    停止：sudo npc stop
    如果需要更换命令内容需要先卸载./npc uninstall，再重新注册
对于windows，使用管理员身份运行cmd
    注册：npc.exe install 其他参数（例如-server=xx -vkey=xx或者-config=xxx）
    启动：npc.exe start
    停止：npc.exe stop
    如果需要更换命令内容需要先卸载npc.exe uninstall，再重新注册
注册到服务后，日志文件windows位于当前目录下，linux和darwin位于/var/log/npc.log
```