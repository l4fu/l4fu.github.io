# windows反调试实现与绕过姿势汇总

这篇文章应该更加详细
https://bbs.pediy.com/thread-262200.htm#msg_header_h1_0

## 调试器检测

###  PEB相关

####  BeingDebugged

BeingDebugged是位于PEB（Process Environment Block，进程环境块）偏移0x2处的标志。

####  

`IsDebuggerPresent()`是Windows的API，这个函数通过查询PEB中的BeingDebugged标志来判断当前进程是否处于被调试状态。

####  NtGlobalFlag

NtGlobalFlag位于PEB的0x068h处，如果BeingDebugged被设置为TRUE，则NtGlobalFlag的值为70h。

####  

CheckRemoteDebuggerPresent( )与IsDebuggerPresent( )类似，同为Windows API，可以直接调用，`CheckRemoteDebuggerPresent()`实际调用了NtQueryInformationProcess()，查询某个进程的ProcessDebugPort即系统与调试器通信的端口句柄，CheckRemoteDebuggerPresent()通过查询这个值来确定程序是否处于调试状态。

####  ProcessHeap

ProcessHeap位于PEB的0x018，处正常情况下，系统为进程创建一个堆，会将进程堆位于+0x00c处的Flags设置为2，将位于+0x010处的ForceFlags设置为0，但在调试状态下这两个位置的值会发生改变。

###  基于系统痕迹

####  父进程检测

一个进程被正常启动时，其父进程一般是Explore.exe文件资源管理器、cmd.exe、或者Services.exe系统服务。如果一个进程的父进程不是这些进程，可以怀疑其被调试了。

####  堆数据检测

由于BeingDebugged被设置为TRUE，NtGlobalFlag设置了FLG\_HEAP\_VALIDATE_PARAMETERS，RtlCreateHeap函数用RtlDebugCteateHeap函数创建堆，与此同时在堆中填充数据：`BA AD F0 0D`、`FE EE FE EE`、`AB AB AB AB`，如果这些数据出现的次数较多（大于10次）则说明被调试了。

####  注册表检测

下面是调试器在注册表中的一个常用位置。

SOFTWAREMicrosoftWindows NTCurrentVersionAeDebug(32位系统)

SOFTWAREWow6432NodeMicrosoftWindowsNTCurrentVersionAeDebug(64位系统)

该注册表项指定当应用程序发生错误时，触发哪一个调试器。默认情况下，它被设置为Dr.Watson。如果该这册表的键值被修改为OllyDbg，则恶意代码就可能确定它正在被调试。

####  进程遍历

枚举进程中是否有调试器进程。

####  窗口遍历

枚举主窗口的标题，判断是否有调试器窗口，与上面的进程检测方法一样，这样的反调试很容易绕过。

###  基于调试器行为的检测

####  硬件断点检测

调试器使用DR0~Dr3作为硬件断点，通过检查这几个寄存器的值是否为空来确定当前程序是否被调试。

####  软件断点检测

调试器软件断点是将断点处的指令替换为INT 3，当程序运行到这一条指令时会调用异常处理例程，从而检查内存中INT 3指令的机器码0xCC可以检查软件断点。除了INT 3还有INT 2D、CD03。

####  检测DBGHELP模块

调试器一般使用微软提供的DBGHELP库来装载调试符号，如果一个进程装载了DBGHELP.DLL那么这个进程很可能是一个调试器。

####  代码CRC值校验

通过对需要保护的代码进行CRC校验或者MD5值校验可以保证这部分代码不被篡改且无法下软断点。

####  SetUnhandledExceptionFilter

在进程发生异常的时候若SEH未处理或者注册的SEH不存在，则会调用UnhandledExceptionFilter，它会运行系统最后的异常处理器，UnhandledExceptionFilter会判断当前进程是否被调试，如果处于被调试状态则将异常传递给调试器，若进程没有被调试则将异常传递给系统最后的异常处理器。使用SetUnhandledExceptionFilter可以修改系统最后的处理器，我们可以修改系统最后的异常处理器为正常逻辑中的一个过程，经由这个正常逻辑过程运行的程序才能正常运行，达到反调试效果。

###  其他检测方法

####  TrapFlag检测

CPU中有一个eflags标志位叫做Trap Flag，如果TF为1，CPU执行指令后会产生一个单步异常，因此可以提前在程序中设置可以跳转到正确程序逻辑的SEH，在触发异常之后，如果进入SEH中则程序可以正常运行，如果没有进入SEH则说明程序处于调试状态。

####  

在函数ZwQuerySystemInformation( )中，当SystemInformation=SystemKernelDebuggerInformation，判断DebuggerEnabled和DebuggerNotPresent()的值可以探测系统调试器是否存在。

####  SeDebugPrivilege权限检测

正常进程不具有SeDebugPrivilege权限，但是调试器具有此权限，当进程从调试器加载时，进程会继承调试器的SeDebugPrivilege权限。可以通过打开CSRSS.EXE进程间接地检查进程是否有SeDebugPrivilege权限，因为默认权限无法对CSRSS.EXE进行OPENPROCESS。

####  TLS回调函数

实际上并不是程序在加载到调试器后，会让第一条指令执行之前而暂停程序的运行，而是调试器从程序PE头部指定的入口点开始。TLS回调被用来在程序入口点执行之前运行代码，因为这些代码可以在调试器中秘密地执行。

TLS是Windows的一个存储类，其中数据对象不是一个自动的堆栈变量，而是代码中运行的每个线程的一个本地变量，TLS允许每个线程维护一个用TLS声明的专有变量，在应用程序实现TLS的情况下，可执行程序的PE头部会包含一个`.tls`段。TLS提供了初始化和终止TLS对象的回调函数。使用`PEView`可以看到`.tls`段，正常程序不会使用这个段。可以使用调试器在TLS回调函数运行之前下断点来分析这些函数。

####  DebugObject

在调试器附加到一个进程的过程中会调用ZwCreateDebugObject创建DebugObject，正常的进程中DebugObject为NULL，如果不是NULL则说明有一个用户态调试器的进程。使用ZwQueryObject查询所有对象的类型，如果DebugObject的数目不为零则说明系统中存在调试器。

####  运行时间差检测

当一个程序运行过程中出现异常时，会将异常传递给调试器由调试器决定处理异常的过程，但是在这个过程中程序运行所需的事件比程序自身直接执行的时间要长很多，因此可以计算一个操作运行的时间来判断当前程序是否正在被调试，常用的方法是RDTSC指令和GetTickCount()函数。

###  已失效的方法

####  

虽然这个方法在较新版本的Windows中不再使用，但是我们还是可以了解一下原理。

使用GetLastError( )可以得到程序运行过程中的出现的错误的原因，在程序出现错误时会改变此函数的返回值，而可以利用此原理首先使用SetLastError()设置LastError为某值，然后通过故意的与调试器相关的错误调用如OutputDebugString()使得程序中出现错误，如果当前程序被调试则OutputDebugString()成功调用因此LastError的值不会改变，但是如果没有被调试则LastError的值会因为OutputDebugString()没有成功调用而发生改变，通过前后LastError的值的对比来判断程序被调试。

##  干扰调试器

####  Drx寄存器清理

OllyDbg在捕获到一个异常时，会将Dr0~Dr7清零，但是如果设置SEH使用这些寄存器中的数据参与运算，通过对比运算结果是否正确可以判断这些寄存器是否被清空即可判断程序是否被OllyDbg调试。

####  OutputDebugStringA

此函数被用来向调试器发送一个格式化的串，但是OllyDbg1.1版本之前存在一个格式化字符串漏洞，使用类似如下的调用形式会触发此漏洞使OllyDbg崩溃：

| <br> | <br> |
| --- | --- |
| 1   | OutputDebugStringA("%s%s%s%s%s%s%s%s%s");   |

####  ThreadHideFromDebugger

设置ThreadHideFromDebugger可以禁止为某个线程产生调试事件。

####  EnableWindow

调用这个API可以暂时锁定前台的窗口：

| <br> | <br> |
| --- | --- |
| 1   | EnableWindow(GetForegrounfWindow(),FALSE);   |

####  BlockInput

调用`BlockInput(TRUE)`锁住窗口，完成工作之后使用`BlockWindow(FALSE)`恢复。锁住期间可以通过`Ctrl+Alt+Del`组合键强制解除输入锁。

####  防止被调试器附加

Ring3调试器的附加使用的是DebugActiveProcess函数，在附加相关进程时，会首先执行到ntdll.dll下的ZwContinue函数，最后停留在ntdll.dll的DbgBreakPoint处。因此Hook一下ZwContinue函数可以实现防止进程被调试器附加的效果。