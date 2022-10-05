# winAFL使用

## winAFL的环境搭建


### 使用预编译的版本
需要下载 DynamoRIO6.2.0-2 ，git上下不下来，所以不能用啊。

### 定制自己的winAFL版本

使用git for Windows从站点Clone所有代码。下载的代码本身包含编译好的版本，包括32位和64位，你也可以自己编译，这样方便理解、修改和使用Winafl。

- 下载CMake（http://www.cmake.org），Winafl的编译需要CMake，CMake是一个跨平台的编译软件。安装CMake，自动建立环境变量。
- 下载DynamoRIO（http://dynamorio.org/），一个跨平台开源的分析代码动态instrumentation工具软件。
-  然后你需要一个编译器，可以用Visual Studio C++，版本的问题是跟CMake有关的，下载最新的CMake，支持VS的2005-2015版本，但由于VS 2010前的版本并不跟C99完全兼容，所以编译的时候会出现stdint.h没有找到的错误，可以从别的和C99兼容的编译器复制一个，比如MinGW、或者直接从网上搜索下载一个。

-  建立一个目录并解压Winafl、DynamoRIO到目录中
-  启动Visual Studio命令行编译环境，如果要编译32位版本进入32位环境、否则进入64位环境，编译什么版本取决于我们需要fuzzing的软件版本。
-  进入Winafl目录，可以发现Bin32和Bin64目录，这2个目录是编译好的Winafl，我们可以重新编译

32位 64位下键入如下命令完成编译：
```
md b32
cd b32
cmake .. -DDynamoRIO_DIR=C:\testDynamoRIOcmake（DynamoRIO在你机器上的路径）
cmake --build . --config Release（也可以是Debug）
md b64
cd b64
cmake -G"Visual Studio 10 Win64" .. -DDynamoRIO_DIR=C:\testDynamoRIOcmake（DynamoRIO在你机器上的路径）
cmake --build . --config Release（也可以是Debug）
```

Winafl编译完成可以使用了，编译好的文件在当前目录下的Release（或Debug）目录。
**注意：** 这个文章的文件夹中编译的环境已经具备了。
查看third_party 下面的源代码是否下载，没有的话请按照git源代码的链接进去下载，放到对应的目录下。
```
third_party\processor-trace\libipt   
--https://github.com/intel/libipt/tree/257202c1fc41687d998e1a82ba1076f816c5c3d5
third_party\winipt
--https://github.com/ifratric/winipt/tree/517c01eef48880fade7b55256e76370769840196
```
编译过程非常慢，需要等待很久。
x86 32位应用程序版本
```
mkdir build32
cd build32
cmake -G"Visual Studio 15 2017" .. -DDynamoRIO_DIR=..\DynamoRIO-Windows-7.0.0-RC1\cmake -DINTELPT=1
cmake --build . --config Release
```
x64 64位应用程序版本
```
mkdir build64
cd build64
cmake -G"Visual Studio 15 2017 Win64" .. -DDynamoRIO_DIR=..\DynamoRIO-Windows-7.0.0-RC1\cmake -DINTELPT=1
cmake --build . --config Release
```
编译完成的程序在 build32\bin\Release  build64\bin\Release 下

### afl-fuzz命令信息
afl-fuzz.exe命令行格式如下：  
`afl-fuzz [afl options] — [instrumentation options] — target_cmd_line`
其中[afl options]常用的参数包括（这些参数由afl-fuzz.exe处理）：  
```
-i dir  – 测试用例存放目录
-o dir – fuzzing过程和结果存放目录
-D dir– 二进制动态Instrumentation工具执行文件路径
-t msec – 超时设置
-x dir – 字典文件

-i in 测试用例存放在当前目录下的in目录
-o out 测试结果和过程输出到当前目录下的out目录
-D c:DynamoRIO-Windows-6.1.1-3bin32 Instrumentation工具执行文件路径
-t 20000 运行fuzzing对象文件的超时
```
[instrumentation options]常用的参数包括（这些参数由winafl.dll处理）：  
```
-coverage_module  – fuzzing对象程序会调用到的模块
-target_module – fuzzing对象模块，也就是-target_offset所在的模块
-target_offset – fuzzing的偏移地址，也就是会被instrumentation的偏移
-fuzz_iterations – 再fuzzing对象程序重新启动一次内运行目标函数的最大迭代数
-debug – debug模式
```
target_cmd_line参数就是你要fuzzing对象的启动程序。  

## 使用winAFL测试fuzz一个demo程序
这里使用一个别人已经做的例子，省略掉前期的库寻找，函数定位的过程。教程参考：https://www.freebuf.com/articles/system/216437.html  这里使用的和参考中的示例dll版本相同，只是来做第一个fuzz的例子，对其中出现的crash也不做深度分析和利用。后期熟练了再进一步完善。
其实也向我们展示了如何去fuzz一个dll库的方法。

### 测试 WinAFL 是否可以正常使用。-debug 表示设置为调试模式。
```
..\DynamoRIO7.0.0\bin32\drrun.exe -c winafl.dll -debug -coverage_module FreeImage.dll -target_module freeImage_fuzz.exe -target_method main -fuzz_iterations 10 -nargs 2 -- freeImage_fuzz.exe samples_pic\a.jpg
```
![测试winAFL是否可以正常运行](_v_images/20200429160740227_29507.png =1010x)
### 筛选测试用例文件
测试用例文件是从网络上搜集的，包含 jpg、png,还可以包含tiff，bmp，ico 等格式，这一步我处理后结果是空的，遂 拷贝了几个PNG文件进去试试。
```
python2 winafl-cmin.py --working-dir C:\fuzz_winALF_RIO\fuzz_demo -D E:\DynamoRIO-7.0.0\bin32 -t 100000 -i C:\fuzz_winALF_RIO\fuzz_demo\samples_pic -o C:\fuzz_winALF_RIO\fuzz_demo\fuzz -coverage_module FreeImage.dll -target_module freeImage_fuzz.exe -target_method main -nargs 2 -- freeImage_fuzz.exe @@
```
### 开始模糊测试
```
afl-fuzz.exe -i fuzz -o fuzz_result -D  C:\fuzz_winALF_RIO\DynamoRIO7.0.0\bin32 -t 20000 -- -coverage_module FreeImage.dll -target_module freeImage_fuzz.exe -target_method main -nargs 2 -- freeImage_fuzz.exe @@
```
![运行10分钟后](_v_images/20200429161129139_16321.png =672x)
发现了 0 个导致程序 Crash 的 Bug，并且挂起计数为 0。速度方面，平均9.00 次每秒，很慢。


```
afl-fuzz.exe -i fuzz -o fuzz_result -D  C:\fuzz_winALF_RIO\DynamoRIO7.0.0\bin32 -t 20000 -- -coverage_module FreeImage.dll -target_module freeImage_fuzz.exe -target_method FreeImage_test -nargs 2 -- freeImage_fuzz.exe @@
```
有pdb文件，上面那个也没法运行。
```
afl-fuzz.exe -i fuzz -o fuzz_result -D  C:\fuzz_winALF_RIO\DynamoRIO7.0.0\bin32 -t 20000 -- -coverage_module FreeImage.dll -target_module freeImage_fuzz.exe -target_offset 0x12510 -nargs 2 -- freeImage_fuzz.exe @@
```
![执行过程](_v_images/20200429172709482_14404.png =672x)
![最后结果](_v_images/20200430075319531_2244.png =672x)
可能是选择的都是小于3KB的文件，所以崩溃的结果较少。

### 衡量 WinAFL 是否正常运行的一个标准
以下图为例，第一个就是 exec speed，少于 20 次每秒算慢的（zzzzz…），60 次左右每秒算中等（slow），正常是 100 次左右每秒，越快模糊测试效率越高。第二个是 fuzzing strategy yields 板块，这个表示样本变异的计数，数字计数越大表示越模糊，如果你执行了 10 多个小时还显示为 0 的话，就要考虑考虑是哪一流程设置错了。
![正常的测试](_v_images/20200429161703142_13132.png!small =672x)

![崩溃](_v_images/20200429181058402_6242.png =762x)
**说明：**
相比以前的手工挖掘漏洞，工具挖掘漏洞的好处就是快，如果你使用 WinAFL 只有 0.1 次每秒的速度，那你还用它干什么。提升 WinAFL 的执行速度有很多种方法，除了更改源码之外（我可不想花时间去更改源码，只想等着作者更新）最常用的方法就是编写更高效的测试文件，比如上面的测试文件，我将 GetProcAddress 函数放到了 FreeImage_test 函数之外执行，这样就不需要每次都执行一边，也就是说将不怎么重要且浪费时间的代码统统写到测试函数之外的地方。不常用的方法就是修改实例文件（-i 参数），这一点真的非常的重要，实例文件的大小会严重影响 WinAFL 的运行速度，比如使用 154 字节的图片作为实例文件输入比使用 4MB 的实例文件输入要快得多。
刚接触这款 Fuzz 工具的时候，它的命令行参数确实多的吓人，这使我足足花了一周的时间去搞清楚了这个软件的各种怪异的错误以及使用的方法和技巧，这里我分享出来，以便使你可以愉快的使用 WinAFL 进行模糊测试。首先是 afl-fuzz.exe 命令，这个命令的参数有很多，其中 -o 参数所指向的文件夹必须是不存在的，因为 WinAFL 会自动建立，不然会报错。-D 参数所指向的二进制插桩工具的路径必须和当前版本的 winafl.dll 版本所匹配，这也是我为什么不用 7.1 而选用 7.0 的原因，而且传入的是 bin32 文件夹的路径而不是 bin32\drrun.exe。还有就是要想使用 -target_method 必须能够查到符号，不然白搭；要想使用 -target_offset 必须为函数的地址，比如 0×1547，多一个少一个都不行。-coverage_module 参数必须在测试文件中包含，也就是使用 -debug 调试所生成的文件中载入的模块。最后就是 -t 参数必须要有，每次我都忘了。反正记住一点，WinAFL 使用中的绝大多数错误和你输入的参数有关。


**参考：**
[0vercl0k-winafl](https://devhub.io/repos/0vercl0k-winafl)
[winafl](https://github.com/ivanfratric/winafl)


