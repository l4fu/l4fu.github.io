# 智能模糊测试工具 Winafl 的使用与分析
[Winafl](https://github.com/ivanfratric/winafl)是Linux下的智能模糊测试神器[afl-fuzz](http://lcamtuf.coredump.cx/afl/)的Windows版本。afl-fuzz从2013年发布到现在，发现了很多真实的漏洞，被广大网络安全从业人员所使用，现在，Windows版本也同样问世，我们同样可以利用它对Windows下的软件进行测试并尝试发现漏洞。

Winafl是一个文件格式及协议漏洞的半自动发现工具，可以帮助我们发现各种使用特定格式文件的应用软件漏洞，如文件编辑软件（doc、xls、pdf等）、图片查看软（bmp、jpg、png等）、视频播放软件（avi、mp4、mpg等）等。

## 一、编译
Winafl的编译一般情况下不会出现什么问题，过程如下：

可以从https://github.com/ivanfratric/winafl直接下载所有代码的打包文件，也可以使用git for Windows从站点Clone所有代码。下载的代码本身包含编译好的版本，包括32位和64位，你也可以自己编译，这样方便理解、修改和使用Winafl。

1. 下载CMake（http://www.cmake.org），Winafl的编译需要CMake，CMake是一个跨平台的编译软件。

2. 下载DynamoRIO（http://dynamorio.org/），一个跨平台开源的分析代码动态instrumentation工具软件。

3. 然后你需要一个编译器，可以用Visual Studio C++，版本的问题是跟CMake有关的，下载最新的CMake，支持VS的2005-2015版本，但由于VS 2010前的版本并不跟C99完全兼容，所以编译的时候会出现stdint.h没有找到的错误，可以从别的和C99兼容的编译器复制一个，比如MinGW、或者直接从网上搜索下载一个。  
安装CMake，自动建立环境变量。

4. 建立一个目录，如C:test，解压Winafl、DynamoRIO到C:test目录。

5. 启动Visual Studio命令行编译环境，如果要编译32位版本进入32位环境、否则进入64位环境，编译什么版本取决于我们需要fuzzing的软件版本。

6. 进入Winafl目录，可以发现Bin32和Bin64目录，这2个目录是编译好的Winafl，我们可以重新编译  
32位下键入如下命令完成编译：  

md b32
cd b32
cmake .. -DDynamoRIO_DIR=C:testDynamoRIOcmake（DynamoRIO在你机器上的路径）
cmake --build . --config Release（也可以是Debug）

64位下键入如下命令完成编译：  

md b64
cd b64
cmake -G"Visual Studio 10 Win64" .. -DDynamoRIO_DIR=C:testDynamoRIOcmake（DynamoRIO在你机器上的路径）
cmake --build . --config Release（也可以是Debug）

Winafl编译完成可以使用了，编译好的文件在当前目录下的Release（或Debug）目录。

##  二、使用
代码编译后生成的主要文件有：  

```
afl-fuzz.exe主程序
winafl.dll  注入进fuzzing程序的模块
```

afl-fuzz.exe命令行格式如下：  

`afl-fuzz [afl options] — [instrumentation options] — target_cmd_line`

其中[afl options]常用的参数包括（这些参数由afl-fuzz.exe处理）：  

```
-i dir  – 测试用例存放目录
-o dir – fuzzing过程和结果存放目录
-D dir– 二进制动态Instrumentation工具执行文件路径
-t msec – 超时设置
-x dir – 字典文件
```

…更多参数请查阅afc-fuzz.c的main函数。

[instrumentation options]常用的参数包括（这些参数由winafl.dll处理）：  
```

-coverage_module  – fuzzing对象程序会调用到的模块
-target_module – fuzzing对象模块，也就是-target_offset所在的模块
-target_offset – fuzzing的偏移地址，也就是会被instrumentation的偏移
-fuzz_iterations – 再fuzzing对象程序重新启动一次内运行目标函数的最大迭代数
-debug – debug模式

```
…更多参数请查阅winafl.c的options_init函数。

target_cmd_line参数就是你要fuzzing对象的启动程序。  
那么，如果fuzzing winafl默认带的例子，整个命令行如下：

```
afl-fuzz.exe -i in -o out -D C:\testDynamoRIO\bin32 -t 20000 -- -coverage_module gdiplus.dll -coverage_module WindowsCodecs.dll -fuzz_iterations 5000 -target_module test_gdiplus.exe -target_offset 0x1650 -nargs 2 -- test_gdiplus.exe @@
```
```

-i in 测试用例存放在当前目录下的in目录
-o out 测试结果和过程输出到当前目录下的out目录
-D c:DynamoRIO-Windows-6.1.1-3bin32 Instrumentation工具执行文件路径
-t 20000 运行fuzzing对象文件的超时
```
— 分割

-coverage_module gdiplus.dll -coverage_module WindowsCodecs.dll为fuzzing对象程序会调用的模块，也就是说你fuzzing的偏移地址的函数会调用到这些模块里面的函数，这个可以通过一些PE工具来查看fuzzing对象调用了那些模块，或者通过winafl.dll的-debug模式获得，然后根据反汇编代码判断调用了那些模块，在无法自行判断的情况下我们写的跟-target_module一样即可。  

```
-target_module test_gdiplus.exe -target_offset所在的模块，也是fuzzing的对象模块

-target_offset 0x1650 fuzzing模块instrumentation的偏移地址，如test_gdiplus.exe是一个很简单的程序，所以对main函数进行fuzzing，而通过ida或类似的工具发现main函数相对于test_gdiplus.exe基地址的偏移为0x1650

```
test_gdiplus.exe @@ fuzzing对象程序的启动命令行  
运行上面命令出现如下界面，afl-fuzz.exe开始工作。

[![智能模糊测试工具 Winafl 的使用与分析](_v_images/20200427162614354_32168.png)](http://blog.jowto.com/wp-content/uploads/2016/09/%E5%9B%BE%E7%89%871.png)

## 三、使用分析
afl-fuzz.c的大体流程：  
建立整个out目录结构及相关文件  
处理测试用例并写到out目录  
通过drrun.exe运行fuzzing对象程序并插入winafl.dll，使用变形后的测试用例进行测试，并返回结果。那么具体out目录结构如下：  
```

out/
/crashes 使应用程序crash的用例，也就是Winafl存放最终结果的目录
/hangs 存放使应用程序挂起的用例，这个目录的文件同样可能使应用程序crash
/queue fuzzing的一些过程文件
/.cur_input 当前fuzzing的用例
/fuzz_bitmap 被fuzzing函数的映射

```
 3.1、测试用例文件变形处理
afl-fuzz.exe通过-i参数指向的目录读取用户提供的测试文件，并通过各种变形存储到out/.cur_input文件，供被fuzzing的应用程序调用打开，查看afl_fuzz.c可以发现其对测试用例都做过哪些变形。

afl-fuzz.c的主要函数是fuzz_one(char  argv)，对测试文件的变形操作也通过这个函数完成，在fuzz_one函数中会调用trim_case(char  argv, struct queue_entry* q, u8* in_buf)函数，先对测试用例进行裁剪，测试用例太大会影响测试速度，这是肯定的，所以理论上说测试用例越小那么全部fuzzing完成的时间就越短，那么怎么裁剪才不会影响fuzzing的结果呢，afl-fuzz.c靠如下手段完成。

首先，Winafl会建立一个64KB的共享内存，用于存储被fuzzing的应用程序的执行路径，如：A -> B -> C -> D -> E (tuples: AB, BC, CD, DE) ，当程序路径发生改变时，如变成A -> B -> D -> C -> E (tuples: AB, BD, DC, CE)，Winafl会更新共享内存，发现新的执行路径更有助于发现代码的漏洞，因为大多数安全漏洞经常是一些没有预料到的状态转移，而不是因为没有覆盖那一块代码。

trim_case函数会用测试用例的总长度除以16，并删除这个长度的数据交给被fuzzing的应用程序打开并记录执行路径到共享内存中，执行完成后trim_case函数对这块共享内存进行hash计算，如果这块共享内存没有发生变化，那么说明没有发现新的执行路径，也同时说明被裁剪的这段数据对整个测试结果影响不大，于是对以后的测试用例都删除掉这段内容，这样，测试用例就会减小到一个合理的大小而并不会影响fuzzing结果。

在fuzz_one函数里，接着就会对测试用例文件做大量的变形操作，包括以下几种类型的变形：bit flips、byte flips、arithmetics、known ints、dictionary、havoc。

在bit flips模式下，对测试用例文件的内容进行按每1、2、4位的长度进行顺序变形，在进行1、2、4位的长度变形时使用如下算法：  

```
#define FLIP_BIT(_ar, _b) do { 
u8* _arf = (u8*)(_ar); 
u32 _bf = (_b); 
_arf[(_bf) >> 3] ^= (128 >> ((_bf) & 7)); 
} while (0)

```
在byte flips模式下，对测试用例文件的内容进行按每8、16、32位的长度进行顺序，变形时使用语句：out_buf[stage_cur] ^= 0xFF;、*(u16*)(out_buf + i) ^= 0xFFFF;、*(u32*)(out_buf + i) ^= 0xFFFFFFFF;来进行变形，并把变形后的测试用例交给common_fuzz_stuff函数进行后续处理。

在arithmetics模式下，8位的运算运用变形语句：out_buf[i] = orig + j;、16位的运算运用变形语句：*(u16*)(out_buf + i) = orig – j;、32位的运算运用变形语句：*(u32*)(out_buf + i) = orig + j;。并把变形后的测试用例交给common_fuzz_stuff函数进行后续处理。

在known ints模式下、8位的运算运用变形语句：out_buf[i] = interesting_8[j];、16位的运算运用变形语句：*(u16*)(out_buf + i) = interesting_16[j];、32位的运算运用变形语句：*(u32*)(out_buf + i) = interesting_32[j];。并把变形后的测试用例交给common_fuzz_stuff函数进行后续处理。其中interesting相关定义如下：

```
static s8 interesting_8[] = { INTERESTING_8 };
static s16 interesting_16[] = { INTERESTING_8, INTERESTING_16 };
static s32 interesting_32[] = { INTERESTING_8, INTERESTING_16, INTERESTING_32 };

```
在dictionary模式下，允许用户使用字典，这样可以用字典文件对测试用例的相关部分进行替换并可以对一些有格式的文本、协议进行fuzz，如html、js、php等。如果使用字典，参数是-x，后面可以是目录或者是一个字典文件，如：-x dictxml.dict、-x dict，装载字典的函数是load_extras，如果参数是一个文件，那么用load_extras_file打开并格式化，用于对一些文本文件处理程序进行fuzz，如php、sql等、如果是一个目录，那么load_extras函数把所有文件内容读到extras数组，对二进制文件处理程序进行fuzz，如doc、xls等，此时对字典文件的大小限定为小于128B，但这个版本的load_extras函数写的有问题，没办法使用，但自己可以简单的修改下，如把load_extras_file(dir, &min_len, &max_len, dict_level);goto check_and_sort;这2条语句上移，不进行多余的判断，直接执行load_extras_file函数，至少能用字典跑一些带格式的文本文件，文本格式文件的一些字典可以参考testcases的例子。

在havoc模式下，通过如下表达式UR(15 + ((extras_cnt + a_extras_cnt) ? 2 : 0))计算出一个值，并通过switch…case进行不同的匹配变形。

 3.2、通过DynamoRIO进行instrumentation
afl-fuzz对测试用例进行变形后，调用DynamoRIO的相关程序对被fuzz的目标程序进行instrumentation并把目标程序的参数设置为变形后的测试用例，监视目标程序的运行，如果目标程序crash，那么复制变形后的测试用例到outcrashes目录下，如果目标程序无响应，那么复制变形后的测试用例到outhangs目录下。这里负责运行DynamoRIO的函数是run_target函数。

run_target首先检查目标进程是否存在，如果存在终止进程，并通过drconfig.exe -nudge_pid %d 0 1命令将一个参数为1的nudge事件送到客户端回调，既目标fuzz程序的参数为1。然后通过drrun.exe -pidfile %s -no_follow_children -c winafl.dll %s — %s命令启动目标程序，并注入winafl.dll到目标程序进程，之后的操作可以通过查看winafl.c文件进行分析，winafl.c的入口函数为dr_client_main函数，先继续一些必要的初始化工作后，首先通过options_init函数处理winafl的相关命令行参数，包括coverage_module、target_module、target_offset等，接着挂载各种回调函数：event_exit，退出事件回调；onexception，异常回调；

instrument_bb_coverage，instrument_edge_coverage，两种模式instrumentation的回调；event_module_load，event_module_unload模块装载、卸载回调。在event_module_load函数中，可以看到winafl通过drwrap_wrap(to_wrap, pre_fuzz_handler, post_fuzz_handler);完成在给定偏移（offset）的指向函数进入时运行pre_fuzz_handler函数，退出时运行post_fuzz_handler函数，监控偏移指向函数的运行。

 3.3、参数选择
coverage_module可以有多个，target_module只有一个，而target_offset指定的偏移是target_module相对于基地址的偏移，coverage_module和target_module必须是目标被fuzz程序所调用的模块或其自身，确定的办法可以通过-debug参数完成，如运行如下命令行：  

```
C:testDynamoRIObin32drrun.exe -c winafl.dll -debug -target_module test_gdiplus.exe -target_offset 0x1650 -fuzz_iterations 10 -nargs 2 — test_gdiplus.exe input.bmp
```

winafl会在当前目录下生成log文件，文件名类似afl.test_gdiplus.exe.13280.0000.proc.log，内容如下图所示：  

[![智能模糊测试工具 Winafl 的使用与分析](_v_images/20200427162614037_9740.png)](http://blog.jowto.com/wp-content/uploads/2016/09/%E5%9B%BE%E7%89%872.png)

可以看到很多行开始都是Module loaded，也就是说参数coverage_module和target_module指定的模块必须出现在这里。  
target_offset的设定简单的命令行程序可以直接设置为程序的main函数，如上面例子的test_gdiplus.exe，复杂的带界面的程序查找偏移可以如下操作：

1. 启动ollydbg并加载目标程序，参数设置为测试用例文档，按F9，目标程序停止在程序入口；

2. 在Command输入框中建立断点：bp CreateFileA、bp CreateFileW，一般情况下应用程序如果打开文件肯定会调用这2个API；

3. 按F9，观察参数是否为输入的测试用例文档，不是继续F9（函数调用太频繁可以使用条件断点 bp kernel32.CreateFileW, [UNICODE [ESP+4]]==”C:test1.mov”），如果是按F7进入，按Ctrl+F9执行到返回，记录下EIP地址；

4. 继续Ctrl+F9执行到返回，记录下记录下EIP地址，如此反复多次，得到多个偏移地址；

5. 用IDA反汇编目标程序，找到所有记录的编译地址所在函数的偏移，这样我们得到了一个执行路径，把每个偏移地址用上面提到的带-debug参数的命令行测试，如果生成的log文件中包含语句：Everything appears to be running normally，那么说明这个偏移地址是可以用的。接着，我们就可以用类似下面的命令行来对目标程序进行fuzz了。

afl-fuzz.exe -i in -o out -D C:testDynamoRIObin32 -t 20000 -- -coverage_module gdiplus.dll -coverage_module WindowsCodecs.dll -fuzz_iterations 5000 -target_module test_gdiplus.exe -target_offset 0x1650 -nargs 2 -- test_gdiplus.exe @@

当然，可以直接alt+k查看Call stack，但有时候想查看更深的调用可以这样操作。这个简单的办法找到的偏移地址极有可能不是最好的偏移地址或者是无用的，而这是跟目标程序写的方法和习惯有关系的，但无论如何确定的target_offset偏移地址必须是目标应用程序打开测试用例肯定会调用到的偏移，否则fuzz程序无法继续运行。

## 四、一个例子
真实的情况是通过winafl想短时间发现流行软件的bug还是比较难的，甚至发现不了。我用winafl跑个很流行的文字处理软件跑了一天，一个crash没有出现，可能时间不够长，呵呵，还是先找个不流行的小众软件测试下好了。我下载了一些不流行的软件来跑，于是winafl终于有了用武之地，每个软件都跑了不太长的时间，就出现了很多crash。

[![智能模糊测试工具 Winafl 的使用与分析](_v_images/20200427162613624_16308.png)](http://blog.jowto.com/wp-content/uploads/2016/09/%E5%9B%BE%E7%89%873.png)

[![智能模糊测试工具 Winafl 的使用与分析](_v_images/20200427162613071_11902.png)](http://blog.jowto.com/wp-content/uploads/2016/09/%E5%9B%BE%E7%89%874.png)

跑了很多软件后，出现了很多crash，但确定这些crash是否可利用还要花大量的时间，而这些软件的价值都不是很大，只粗略的看了下，基本上都是读异常，被利用的可能性很小，当然仔细分析的话还是可能存在被利用的可能，但由于上述的原因没有仔细分析。其中有一个软件的crash很像能利用，于是简单的分析了下，最终认为也不能利用，下面把这个例子贴出来。  
程序出错位置：

[![智能模糊测试工具 Winafl 的使用与分析](_v_images/20200427162612282_11439.png)](http://blog.jowto.com/wp-content/uploads/2016/09/%E5%9B%BE%E7%89%875.png)

异常的语句：rep movsd dword ptr es:[edi], dword ptr [esi]，明显是一个memcpy操作，检查下esi是否可以控制，向上一层找：

[![智能模糊测试工具 Winafl 的使用与分析](_v_images/20200427162611847_7862.png)](http://blog.jowto.com/wp-content/uploads/2016/09/%E5%9B%BE%E7%89%876.png)

esi就是从上图中黄色显示的eax经过运算得来的，默认情况下这个值应该是50，当修改测试文件相关偏移的内容后，这个值是可以有限控制的，但看完esi后继续看edi，发现完全无法控制，edi是函数内部malloc出来的，而复制的长度是固定的，也就是说基本上这个crash不太可能利用了。

无论如何，Winafl是一个开源的优秀半自动化fuzz测试工具，相比较其他的公开fuzz测试工具还是有很大的优势的，那么让我们去发现漏洞吧。 ^_^