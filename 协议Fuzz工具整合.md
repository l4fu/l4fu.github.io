# 协议Fuzz工具整合
概括整理协议fuzz相关的工具，针对其实现原理和过程进行总结，分析各工具优势和不足，进行横向和纵向的对比。

  
因为有些论文工具暂时没有包括进来，所以会长期更新完善，争取覆盖掉所有的主流协议fuzz工具。而且针对论文性质的工具，后续会出论文解读的详细内容，该文章中只做工具的整理。


## AFLNet

  

source：https://github.com/aflnet/aflnet

##   

## 1、Description

  

AFLNet是一款灰盒协议fuzz工具，采用了代码覆盖率反馈、种子变异以及状态反馈等技术。

与传统的基于生成的协议fuzz工具，它采用了server和client之间的通信消息数据作为种子，无需任何的协议规范。本质上是模拟一个client来发送一系列消息到server，并保留可以触发新的代码执行路径或者响应状态的变异数据。

AFLNet使用server端的响应码来识别消息序列触发的不同状态，根据这种反馈，AFLNet可以尽可能得向有效的状态区域靠近。

  

## 2、Installation

###   

### （1）环境

  

system：Ubuntu 20.04 64-bit

dependencies：clang, graphviz-dev

（对于环境要求，只要支持AFL和graphviz工具即可，对版本没有特殊要求。）

  

### （2）AFLNet

  

下载源码，并编译，最后设置环境变量：

```
# First, clone this AFLNet repository to a folder named aflnet
```

##   

## （3）Usage

  

样例：

```
afl-fuzz -d -i in -o out -N <server info> -x <dictionary file> -P <protocol> -D 10000 -q 3 -s 3 -E -K -R <executable binary and its arguments (e.g., port number)>
```

  

选项说明：

*   \-N netinfo：server信息（例如 tcp://127.0.0.1/8554）
    
*   \-P protocol：待测试的应用协议（例如：RTSP, FTP, DTLS12, DNS, DICOM, SMTP, SSH, TLS, DAAP-HTTP, SIP）
    
*   \-D usec：可选，server完全初始化的等待时间，微秒为单位
    
*   \-K：可选，处理完所有请求消息后，向server发送SIGTERM信号以终止server
    
*   \-E：可选，开启状态感知模式
    
*   \-R：可选，开启region-level变异
    
*   \-F：可选，启用假阴性消除模式
    
*   \-c script：可选，server cleanup的脚本或完整路径
    
*   \-q algo：可选，状态选择算法（1. RANDOM\_SELECTION, 2. ROUND\_ROBIN, 3. FAVOR）
    
*   \-s algo：可选，种子选择算法（1. RANDOM\_SELECTION, 2. ROUND\_ROBIN, 3. FAVOR）
    

##   

## （4）Example

  

Fuzzing Live555流媒体服务器

Live555是一个为流媒体提供解决方案的跨平台的C++开源项目，它实现了标准流媒体传输，是一个为流媒体提供解决方案的跨平台的C++开源项目，实现了对标准流媒体传输协议如RTP/RTCP、RTSP、SIP等的支持。

Live555实现了对多种音视频编码格式的音视频数据的流化、接收和处理等支持，包括MPEG、H.263+ 、DV、JPEG视频和多种音频编码。同时由于良好的设计，Live555非常容易扩展对其他格式的支持。Live555已经被用于多款播放器的流媒体播放功能的实现，如VLC(VideoLan)、MPlayer。

###   

### Server和client的编译和安装

  

为了展示fuzz成果，这里使用的Live555为2018年的一个旧版本，在这个版本中，AFLNet发现了4个漏洞，其中2个是0day。Live555的编译和安装如下：

```
cd $WORKDIR
```

  

在上述命令中，使用了一个patch来使得目标更容易被fuzz。patch文件的主要功能是设置编译器为afl-clang-fast\\fast++以实现覆盖反馈：

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

禁用Live555中的随机会话id生成：

![图片](_v_images/310382617242321.webp)

这里的随机会话id主要是在原生的Live555版本中，会为每个连接生成一个session id，该id会用在从以连接的client发送的后续的request中，如果没有合法的id则会被server拒绝，这样就会导致fuzz过程中即使是相同的消息序列也会产生不同的执行路径，不同部分主要是session id部分的更改，所以这里就直接硬编码session id来消除这种干扰变量。

Live555编译成功后，在testProgs目录下会看到server端testOnDemandRTSPServer和client端testRTSPClient。可以执行以下命令进行测试：

```
# Move to the folder keeping the RTSP server and client
```

  

client能成功发送reqeust并接收从server发来的数据即可。

![图片](_v_images/307312617214644.webp)

  

### ① 准备种子

  

AFLNet使用发送的消息序列作为种子输入，所以需要先获取消息序列。基本方法就是首先捕获一些testRTSPClient和server之间的交互数据作为种子输入，这里以请求一个wav格式的音频文件为例：

首先启动server：

```
cd $WORKDIR/live555/testProgs
```

  

使用tcpdump捕获8554端口的所有流量并保存到pcap文件：

```
tcpdump -w rtsp.pcap -i lo port 8554
```

  

运行client发起请求：

```
./testRTSPClient rtsp://127.0.0.1:8554/wavAudioTest
```

  

交互完成后，停下tcpdump，对抓到的流量进行分析。直接跟踪TCP流查看所有发往server的数据：

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

这就是在整个的交互过程中，client发出的所有的request消息序列，将它作为种子即可（需要使用的是原始数据二进制格式）。种子路径为$AFLNET/tutorials/live555/in-rtsp：

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

###   

### ② 进行fuzz

```
cd $WORKDIR/live555/testProgs
```

  

触发crash的消息序列的测试用例会放在crashes或replayable-hangs目录中。在fuzz过程中，AFLNet状态机会不断推断server的已实现状态，并相应地更新.dot文件，可以查看该文件来监控AFLNet在协议推理过程中的情况。

![图片](_v_images/304242617225180.webp)

  

![图片](_v_images/300192617232938.webp)

   

（备注：不做任何修改的话，这个fuzz速度慢到令人发指，具体原因暂时还未深入研究。）

  

### ③ crash重现

  

AFLNet自带重现工具：afl-replay

```
./afl-replay tutorials/live555/CVE_2019_7314.poc RTSP 8554
```

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

（备注：官方并没有说明命令执行完成后会怎么样，反正我执行完没有任何反应，目标程序也没有发生crash）

获取更多的crash信息可以使用GDB调试工具调试server，或者加上Address Sanitizer-Enabled patch。

##   

  

二

  

**StateAFL**

##   

## 1、Description

  

沿用AFL的基本思路，但是提升了代码覆盖率，并且寻求最大化协议状态覆盖。StateAFl会自动推测server的当前协议状态。在编译阶段，会想目标server的内存分配和网络I/O操作插入探针；在运行阶段，会为每个协议迭代拍摄进程内存中长期驻留的数据的快照，并使用fuzzy hashing将内存状态映射到唯一的协议状态。

![图片](_v_images/295132617228155.webp)

（感觉本质上还是AFLNet的思路，发送请求序列，只不过在状态监控上做了另外一种方案。）

  

## 2、Installation

  

与AFLNet的安装方式基本相同：

```
# Install clang (required by afl-clang-fast)
```

##   

## 3、Usage

  

与AFLNet一模一样。

  

## 4、Example

  

测试环境与AFLNet的一样，这里直接执行fuzz，看看效果如何：

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](_v_images/291072617212364.webp)

速度上比AFLNet会快一丢丢，在覆盖率上也会高一点。

（自带的语料库没法触发漏洞，忽略这个吧。毕竟挖不到洞才是常态）

##   

  

三

  

**ProFuzzBench**

  

这是一个对于网络协议相关fuzz工具的测试集合，可以方便地对流行的几款协议fuzz工具进行使用测试。

ProFuzzBench是一个有状态的网络协议测试的基准，包含了多个用于测试流行网络协议（TLS, SSH, SMTP, FTP, SIP等）的server。

ProFuzzBench提供了自动化执行fuzz的脚本，截止目前包含的fuzz工具有：AFLnwe（基于网络的AFL版本，使用TCP/IP sockets替代文件作为输入）、AFLNet（一款针对有状态网络协议的fuzz工具），而StateAFL（另一款针对有状态网络协议的fuzz工具）尚未提供自动化执行脚本，可直接使用原生的StateAFL进行fuzz。

其项目结构如下：

```
protocol-fuzzing-benchmark
```

  

接下来以LightFTP为例进行fuzz。

  

## Installation and Setup

  

首先，下载ProFuzzBench的源码并设置环境变量：

```
git clone https://github.com/profuzzbench/profuzzbench.git
```

  

然后，创建lightFTP server的docker ：

```
cd $PFBENCH
```

##   

## 1、Run Fuzzing

  

执行profuzzbench\_exec\_common.sh脚本开启环境，这里涉及到8个参数：

*   Doc：指定docker 
    
*   Runs：runs的实例数，每个container跑一个fuzz过程，类似于并行fuzz
    
*   Saveto：结果的保存路径
    
*   Fuzzer：指定使用的fuzzer name
    
*   Outdir：docker container中创建的output文件夹名称
    
*   Options：除了目标特定的 run.sh 脚本中编写的标准选项之外，fuzz所需的所有选项
    
*   Timeout：fuzz时长，秒为单位
    
*   Skipcount：用于计算覆盖率时间，例如Skipcount=5就是说每5个test case后执行一次gcov
    

  

以下指令就是在10分钟内执行4个AFLNet实例和4个AFLnwe实例来对LightFTP进行fuzz：

```
cd $PFBENCH
```

  

AFLNet的fuzz结果如下：

![图片](_v_images/286032617238564.webp)

输出结果保存在了\`results-lightftp目录中：

![图片](_v_images/282982617223266.webp)

一个fuzzer实例产生一个.tar.gz文件，每个.tar.gz文件包含了这次fuzz过程产生的所有数据。

类似的，AFLnwe的输出一样：

![图片](_v_images/278932617215245.webp)

这两个工具对比起来，fuzz效果都一般般，但AFLNet的速度会快一些，相对而言，比AFLnwe效果也好一些。cov\_over\_time数据对比如下：

![图片](_v_images/277872617226788.webp)

  

## 2、Collect the results

  

对于覆盖率信息（也就是上面的表格）的处理，ProFuzzBench提供了处理覆盖率的自动化工具profuzzbench\_generate\_cssv.sh，参数说明如下：

*   prog：subject program的名字，例如lightftp
    
*   runs：runs数量
    
*   fuzzer：fuzzer的name，例如AFLNet
    
*   covfile：CSV格式的输出文件
    
*   append：append模式
    

  

可以使用下面的命令来处理fuzzer跑出来的覆盖率信息文件：

```
cd $PFBENCH/results-lightftp
```

  

输出结果为results.csv文件：

![图片](_v_images/273832617234869.webp)

  

文件中的第一列为时间戳，第二列为目标程序，第三列为fuzzer，第四列为运行的index，第五列为coverage type，第六列为type的值。文件包含了随时间变化的line coverage和branch coverage信息，每个coverage type都包含两个值：百分比 (\_per) 和绝对数量 (\_abs)。

##   

## 3、Analysis the results

  

前面生成的results.csv文件可以进一步进行绘图显示，使用脚本profuzzbench\_plot.py。

```
cd $PFBENCH/results-lightftp
```

  

处理完成后，可以看到处理结果：

![图片](_v_images/270792617235981.webp)

从这里也能看出AFLNet比AFLnwe的效果要好太多。其实这个脚本的实现也不难，python中有很多成熟的库。

  

## 4、Conclusion

  

该工具支持添加自定义的fuzzer，按照stateafl的添加方式来进行添加即可，但是个人感觉以docker模式运行，速度上肯定要变慢。该工具值得肯定的是将常见的几个网络协议的fuzz环境能实现自动化搭建，可以节省一定的环境搭建时间成本。看个人需求吧，侧重效率的话还是要自行根据需求搭建独立环境。

该工具目前支持的协议环境有：

  

![图片](_v_images/267662617233569.webp)

  

支持自定义添加协议，官方的开发文档也算完善，还是比较值得尝试的（毕竟也没有其他更好的选择了）。