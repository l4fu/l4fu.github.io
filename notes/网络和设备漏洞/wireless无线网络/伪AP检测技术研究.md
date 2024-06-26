# 伪AP检测技术研究

0x00 简介
=======

* * *

随着城市无线局域网热点在公共场所大规模的部署，无线局域网安全变得尤为突出和重要，其中伪AP钓鱼攻击是无线网络中严重的安全威胁之一。

受到各种客观因素的限制，很多数据在WiFi网络上传输时都是明文的，如一般的网页、图片等；甚至还有很多网站或邮件系统在手机用户进行登陆时，将帐号和密码也进行了明文传输或只是简单加密传输（加密过程可逆）。因此，一旦有手机接入攻击者架设的伪AP，那么通过该伪AP传输的各种信息，包括帐号和密码等，就会被攻击者所截获。

本文主要从理论上探讨伪AP钓鱼攻击的检测和阻断技术。

0x01 无线网络基础知识
=============

* * *

### 基本术语

*   AP（Access point的简称，即访问点，接入点）：是一个无线网络中的特殊节点，通过这个节点，无线网络中的其它类型节点可以和无线网络外部以及内部进行通信。
*   Station（工作站）：表示连接到无线网络中的设备，这些设备通过AP，可以和内部其它设备或者无线网络外部通信。
*   Assosiate：连接。如果一个Station想要加入到无线网络中，需要和这个无线网络中的AP关联（即Assosiate）。
*   SSID：表示一个子网的名字，无线路由通过这个名字可以为其它设备标识这个无线路由的子网。设备进行扫描的时候，就会将相应SSID扫描到，然后就能够选择相应的SSID连接到相应的无线网络（当然不扫描，理论上也可以直接指定自己事先已经知道的ssid进行连接）。SSID可以和其它的重复，这样扫描的时候会看到两个同样SSID的无线网络。
*   BSSID：用来标识一个BSS，其格式和MAC地址一样，是48位的地址格式。一般来说，它就是所处的无线接入点的MAC地址。某种程度来说，它的作用和SSID类似，但是SSID是网络的名字，是给人看的，BSSID是给机器看的，BSSID类似MAC地址。
*   BSS（Basic Service Set）：由一组相互通信的工作站组成，是802.11无线网络的基本组件。主要有两种类型的IBSS和基础结构型网络。IBSS又叫ADHOC，组网是临时的，通信方式为Station<->Station，这里不关注这种组网方式；我们关注的基础结构形网络，其通信方式是Station<->AP<->Station，也就是所有无线网络中的设备要想通信，都得经过AP。在无线网络的基础网络中，最重要的两类设备：AP和Station。

### 802.11标准简介

802.11是IEEE在1997年为无线局域网(Wireless LAN)定义的一个无线网络通信的工业标准。此后这一标准又不断得到补充和完善，形成802.11x的标准系列。802.11x标准是现在无限局域网的主流标准，也是Wi-Fi的技术基础。本文只介绍一下平时应用最多的应该是802.11a/b/g三个标准。

| 协议 | 发布(年/月) | Op.标准频宽 | 实际速度（标准） | 实际速度（最大） | 半径范围（室内） | 半径范围（室外） |
| --- | --- | --- | --- | --- | --- | --- |
| 802.11a | 1999 | 5.15-5.35/5.47-5.725/5.725-5.875 GHz | 25 Mbit/s | 54 Mbit/s | 约30米 | 约45米 |
| 802.11b | 1999 | 2.4-2.5 GHz | 6.5 Mbit/s | 11 Mbit/s | 约30米 | 约100米 |
| 802.11g | 2003 | 2.4-2.5 GHz | 25 Mbit/s | 54 Mbit/s | 约30米 | 约100米 |

### WLAN拓扑结构

WLAN有以下三种网络拓扑结构:

1.  独立基本服务集(Independent BSS, IBSS)网络(也叫ad-hoc网络)
2.  基本服务集(Basic Service Set, BSS)网络
3.  扩展服务集(Extent Service Set, ESS)网络

**1) AD-Hoc网络**

images:  425716f55fd2e414dd8660827339bd5c0b0d96ef.jpg

无AP，站点间直接通信。

win7自带的AD-Hoc组建功能，可以让我们很方便的在一个小范围内快速组建"局域网"，联网打游戏啥的很方便。

**2) BSS网络**

images:  a1d1441821b9f49efc2f96db8f0d979738f01e44.jpg

又名Infrastructured网(基础设施网)：有AP(Access Point, 接入点)，无线站点通信首先要经过AP。对于个人PC来说，使用最多的所谓"无线Wi-Fi"指的就是BSS网络模式

**3) ESS网络**

images:  b2d69a9734e648972b84868c7b10598a8dc93d18.jpg

其中，ESS中的DS(分布式系统)是一个抽象系统，用来连接不同BSS的通信信道(通过路由服务)，这样就可以消除BSS中STA与STA之间直接传输距离受到物理设备的限制。

根据拓扑结构可以得出802.11的两类服务：

1.  站点服务SS(每个STA都要有的服务)
    
    1.  认证(Authentication)
    2.  解除认证(Deauthentication)
    3.  加密(Privacy)
    4.  MSDU传递(MSDU delivery)
2.  分布式系统服务DSS(DS特有服务)
    
    1.  关联(Association)
    2.  解除关联(Deassociation)
    3.  分布(Distribution)
    4.  集成(Integration)
    5.  重关联(Ressociation)

### 帧格式

**3.1 帧格式概述**

无线中的数据传播有如下相似的格式：

images:  718e6dc4c35b2a00ec272d1b49368010d53f443c.jpg

其中preamble是一个前导标识，用于接收设备识别802.11，而PLCP域中包含一些物理层的协议参数，显然Preamble及PLCP是物理层的一些细节。MAC层处理的是帧数据，截取上图中MAC头开始的部分构成MAC帧格式如下所示：

images:  87dd04219cafe6cc25773a55597880042ded1cd2.jpg

*   MAC Header（MAC头）：Frame Control（帧控制域），Duration/ID（持续时间/标识），Address（地址域），Sequence Control（序列控制域）、QoS Control（服务质量控制）；
*   Frame Body（帧体部分）：包含信息根据帧的类型有所不同，主要封装的是上层的数据单元，长度为0~2312个字节，可以推出，802.11帧最大长度为：2346个字节；
*   FCS（校验域）：包含32位循环冗余码。

**3.2 MAC Header**

1）Frame Control（帧控制域）格式如下所示：

images:  5d4b96b7227eda0f68f7827b9fa49d6826b3cea3.jpg

*   Protocol Version（协议版本）：通常为0；
*   Type（类型域）和Subtype（子类型域）：共同指出帧的类型；
*   To DS：表明该帧是BSS向DS发送的帧；
*   From DS：表明该帧是DS向BSS发送的帧；
*   More Frag：用于说明长帧被分段的情况，是否还有其它的帧；
*   Retry（重传域）：用于帧的重传，接收STA利用该域消除重传帧；
*   Pwr Mgt（能量管理域）：为1：STA处于power_save模式，0：处于active模式；
*   More Data（更多数据域）：为1：至少还有一个数据帧要发送给STA ；
*   Protected Frame：为1：帧体部分包含被密钥套处理过的数据；否则：0；
*   Order（序号域）：为1：长帧分段传送采用严格编号方式；否则：0。

2）Duration/ID（持续时间/标识）：表明该帧和它的确认帧将会占用信道多长时间；对于帧控制域子类型为：Power Save-Poll的帧，该域表示了STA的连接身份（AID, Association Indentification）；

3）Address（地址域）：源地址（SA）、目的地址（DA）、传输工作站地址（TA）、接收工作站地址（RA），SA与DA必不可少，后两个只对跨BSS的通信有用，而目的地址可以为单播地址（Unicast address）、多播地址（Multicast address）、广播地址（Broadcast address）；

4）Sequence Control（序列控制域）：由代表MSDU（MAC Server Data Unit）或者MMSDU（MAC Management Server Data Unit）的12位序列号（Sequence Number）和表示MSDU和MMSDU的每一个片段的编号的4位片段号组成（Fragment Number）。

**3.3 帧类型**

针对帧的不同功能，可将802.11中的MAC帧细分为以下3类：

1.  控制帧：控制帧主要用于协助数据帧的传递，所有控制帧都使用相同的Frame Control字段；
2.  管理帧：管理帧负责在工作站和AP之间建立初始的通信，提供认证和连接服务，包括了连接请求/响应、轮询请求/响应等；
3.  数据帧：用于在竞争期和非竞争期传输数据。

Frame Control（帧控制域）中的Type（类型域）和Subtype（子类型域）共同指出帧的类型，当Type的B3B2位为00时，该帧为管理帧；为01时，该帧为控制帧；为10时，该帧为数据帧。而Subtype进一步判断帧类型，如管理帧里头细分为关联和认证帧，管理帧的帧格式如下图所示：

images:  87c06d2e869daf5483333bf4f8d74743df298c40.jpg

管理帧的主体包含的固定字段与信息元素是用来运送信息的。管理帧主要以下几种，负责链路层的各种维护功能。

1.beacon信标帧

主要用来声明某个网络的存在。定期（默认100s、可自己设置）传送的信标可让station得知网络的存在，从而调整加入该网络所必需的参数。

2.Probe Request 探查请求帧

移动工作站利用Probe Request探查请求帧来扫描区域内目前有哪些802.11网络。

包含2个字段：

*   SSID：可被设定为特定网络的SSID或任何网络的SSID。
*   Support rates：移动工作站所支持的速率。

3.ProbeResponse探查响应帧

如果ProbeRequest所探查的网络与之兼容，该网络就会以ProbeResponse帧响应。送出最后一个信标帧的工作站必须负责响应所收到的探查信息。

Probe Request帧中包含了信标帧的所参数，station可根据它调整加入网络所需要的参数。

4.IBSS announcement traffic indication map (ATIM)

IBSS 的通知传输只是消息。

5.Disassociation and Deauthentication

取消关联、解除验证帧。

6.AssociationRequest

关联请求帧。

7.Reassociation Request

重新关联。

8.Association Response and Reassociation Response

关联响应、重新关联响应。

9.Authentication

身份验证帧

10.Action frame

帧传送、关联与身份验证的状态。共有三种状态，后面会详细介绍。

下面主要介绍下信标帧的格式：

信标帧被定期传送以便于移动STA能定位和识别BSS，信标帧中的信息允许移动STA能在任何时候定位BSS，同时信标帧还含有通知处于节能模式的STA取回缓冲的帧的信息。

images:  25e579395abaa18ab56362c12e40cb44ae22c88d.jpg

如图所示，一个信标帧包含有以下固定域：时间戳、信标间隔、能力信息。信标帧中的信息单元包括：服务集标识号(SSID)、支持的速率、一个或多个PHY参数集、一个可选的无竞争参数集、一个可选的IBSS参数集和一个可选的通信指示图。

1.  Timestamp (8 byte)：时间戳是64位长的域，它的值是传送该帧时的STA的同步时钟。
2.  信标 Interval (2 byte)：信标间隔的长度是16位，其值是以TU(1024μs)为单位的两次相邻信标帧之间的间隔时间。
3.  Capability info (2 byte)：能力信息域的长度是16位，标识的是STA的能力。
4.  SSID (variable size)
5.  Supported Rates (variable size)

0x02 伪AP钓鱼攻击
============

* * *

伪AP钓鱼攻击，国外又称为“双面恶魔攻击（evil twin attack）”，是通过仿照正常的AP，搭建一个伪AP，然后通过对合法AP进行拒绝服务攻击或者提供比合法AP更强的信号迫使无线客户端连接到伪AP。因为无线客户端通常会选择信号比较强或者信噪比（SNR）低的AP进行连接。为了使客户端连接达到无缝切换的效果，伪AP应该以桥接方式连接到另外一个网络。如果成功进行了攻击，则会完全控制无线客户端网络连接，并且可以发起任何进一步的攻击。

images:  e304a73e8f3c2778bb4cbf64303eb3956cce07d3.jpg

通常来说，发起无线钓鱼攻击的黑客会采取以下步骤：

1、获取无线网络的密钥。对于采用WEP或WPA认证的无线网络，黑客可以通过无线破解工具，或者采用社会工程的方法，来窃取目标无线网络的密钥，对于未加密的无线网络则可以省略这一步骤，使得无线钓鱼攻击更容易得手。

2、伪造目标无线网络。用户终端在接入一个无线网络之前，系统会自动扫描周围环境中是否存在曾经连接过的无线网络。当存在这样的网络时，系统会自动连接该无线网络，并自动完成认证过程；当周围都是陌生的网络时，需要用户手工选择一个无线网络，并输入该网络的密钥，完成认证过程。

而黑客在伪造该无线网络时，只需要在目标无线网络附近架设一台相同或近似SSID的AP，并设置上之前窃取的无线网络密钥。这台AP一般会设置成可以桥接的软AP，因此更加隐蔽，不容易被人发现。

这样，黑客伪造AP的工作就完成了，由于采用了相同的SSID和网络密钥，对用户来说基本上难辨真伪，并且由于伪造AP使用了高增益天线，附近的用户终端会接收到比较强的无线信号，因此用户会发现，在自己终端上的无线网络列表中，这个伪造的AP是排在靠前位置的。这样，用户就会很容易上钩，掉入这个精心构造的陷阱中。

images:  a6b05ae165d0489654e12732e1d36ffc5b76efc9.jpg图 无线钓鱼过程

3、干扰合法无线网络。对于那些没有自动上钩的移动终端，为了使其主动走进布好的陷阱，黑客会对附近合法的网络发起无线DoS攻击，使得这些无线网络处于瘫痪状态。这时，移动终端会发现原有无线网络不可用，重新扫描无线网络，并主动连接附近同一个无线网络中信号强度最好的AP。由于其他AP都不可用，并且黑客伪造的钓鱼AP信号强度又比较高，移动终端会主动与伪造的AP建立连接，并获得了IP地址。

至此，无线钓鱼的过程就已经完成了，剩下的就是黑客如何处理网中的这些"鱼"了。

4、截获流量或发起进一步攻击。无线钓鱼攻击完成后，移动终端就与黑客的攻击系统建立了连接，由于黑客采用了具有桥接功能的软AP，可以将移动终端的流量转发至Internet，因此移动终端仍能继续上网，但此时，所有流量已经被黑客尽收眼底。黑客会捕获流量并进一步处理，如果使用中间人攻击工具，甚至可以截获采用了SSL加密的Gmail邮箱信息，而那些未加密的信息更是一览无余。

更进一步，由于黑客的攻击系统与被钓鱼的终端建立了连接，黑客可以寻找可利用的系统漏洞，并截获终端的DNS/URL请求，返回攻击代码，给终端植入木马，达到最终控制用户终端的目的。此时，那些存储在终端上的资料已经是黑客囊中之物。

0x03 伪AP检测技术
============

* * *

对于伪AP钓鱼攻击，采用精准迅速的检测技术及时地发现并阻断攻击是非常必要的。目前的伪AP检测技术主要包括信标帧时钟偏差、时序特性、无线嗅探、指纹识别等方式。

### 通过时钟偏差检测

利用时钟偏差技术快速准确的检测出未授权的无线AP，主要通过收集AP的信标帧和探头响应消息来计算AP的时钟偏差，这个是根据IEEE802.11协议中的TSF（Time Synchronization Function，定时同步功能）时间戳来计算的。如果AP的时钟偏差值与特征库中储存的偏差值不一样，则可认定这个AP为一个无线钓鱼AP。

伪AP通常从源信标帧复制timestamp字段的值，所以它与授权AP几乎采用同一个时钟偏差。但是，当进行信标帧重构时，在形成帧并重传的过程中会产生几微秒的延迟。这将产生具有不同序列号的重复帧，这样就可以通过计算两个相邻信标帧之间的timestamp间隔差异，即时钟偏差，与预先设定的阈值进行比较来检测伪AP。检测步骤如下图所示：

images:  63db9f1b3f3d43957db355c53864653dc4c4f40f.jpg

首先，从无线网络中的授权接入点获取训练数据集。数据集包含从授权的AP（合法AP）中接收的信标帧。对于AP的时钟偏差的计算，我们使用信标帧的时间戳值。

从信标帧中获取到时间戳后使用LSF（最小二乘法拟合）算法计算出AP的时钟偏差，然后存储进数据库作为以供以后使用。

处理完授权AP后。接下来需要对未授权的AP（待检测AP）进行处理。首先，我们需要建立未授权的AP数据集，该数据集包含信标帧值。同样使用LSF算法计算未授权AP的时钟偏差。

之后，比较合法AP与待检测AP的时钟偏差值。根据比较结果可知待检测的AP是否为伪AP。

### 基于时序特性

基于时序特性的检测方法实际上就是通过检测两个顺序帧的接收时间间隔，即帧间到达时间来进行伪AP识别的。也可以测量认证帧和ACK帧之间的时序，即第一个确认帧被发送和相应的认证响应帧被发送之间的时间间隔。这种技术的缺点是在高流量的网络中检出率比较低。

### 使用无线嗅探解决方案

一些公司如 Air-Magnet 使用无线嗅探解决方案，将传感器部署在整个网络空间中。为了在分布式代理服务器结构中检测伪AP，使用传感器收集物理和数据链路层信息，所收集的信息中包含信道、SSID、MAC地址、信号强度和AP控制帧等信息。非法设备和安全威胁能够被追踪并通过无线锁定，或者锁定交换机端口。您也可以在平面图上看到非法设备准确的物理位置已确定您所处无线环境内的所有可能存在的安全威胁。

这些部署方案的代价是十分昂贵的，且这种无线方案一般不容易扩展，因为它涉及到大量的基础设施改建和大型网络改建，部署无线嗅探器要充分覆盖大型网络，例如，在城市建设公共热点。这种无线嗅探方式在两种情况下将会失效：第一种，无线钓鱼AP禁止向外界发送信标帧（隐藏SSID），降低信号强度，或使用非标准的协议和频率；第二种，攻击者使用定向天线，使得无线钓鱼AP覆盖范围很小，从而很难被发现。

### 采用TCP/IP指纹识别方法

这种方法运用请求-响应主动探测技术，可以通过网络探测和像nmap之类的安全审计工具实现。这种方法发送一个请求帧，并等待响应以确定设备如何对碎片帧做出回应或者如何处理帧。这种技术也有缺点：它使用了可以被大多数攻击者绕过的主动探测技术。此外，该技术可以被正常的WLAN流量所干扰。

### 基于设备指纹识别技术

论文《A Passive Fingerprint Technique to Detect Fake Access Points》中提出了一种使用PLL（the physical layer header length）作为指纹参数进行伪AP检测，文章指出这种方法在非高流量环境下可以在100ms内准确地识别到伪AP。

images:  123786884b085b6f30f8bfa4631b14c60ba0ed96.jpg

该检测流程首先捕获AP无线通信数据包，然后过滤出信标帧，从中提取出MAC、SSID，PLL存入到buffer中。

接下来需要设置一个阀值（TSV），然后比较每一个信标帧的PLL与TSV，如果PLL<TSV，则表明该AP是伪AP。

论文中通过训练大量合法AP与伪AP的样本数据得到TSV的计算公式为：

images:  4af8986222290cabe748c3f9322855afd86f847c.jpg

即取伪AP的最大PLL值与合法AP的最小PLL之和的平均值。

0x04 伪AP阻断技术
============

* * *

在检测到伪AP后，需要采用有效而精准的方式阻断客户端连接伪AP。我们可以采用dos攻击方式，使客户端无法连接到伪AP，从而达到阻断伪AP连接的目的。针对无线AP的dos攻击主要有以下几种：

*   认证洪水攻击（Authentication Flood Attack）
*   去认证洪水攻击（Deauthentication Flood Attack）
*   关联洪水攻击（Association Flood Attack）
*   去关联洪水攻击（Disassociation Flood Attack）
*   RF Jamming攻击

为了了解这几种攻击的原理，我们必须先了解IEEE802.11无线认证状态。

### 无线认证状态

IEEE 802.11将AP和STA在建立连接阶段的状态分为以下三种：

*   状态1：未认证未联结；
*   状态2：已认证未联结；
*   状态3：已认证已联结。

最初，STA和AP都处于状态1，AP在启动之后就会广播信标帧，告知周围的STA这里存在一个可用的无线网络，通常AP广播信标帧的间隔为100毫秒。在连接到AP之前，STA首先要扫描所有无线信道，选择要连接的AP，然后同选定的AP进行认证。完成认证后，STA和AP将进入状态2，STA将与AP建立连接，成功二者将进入状态3。

如果STA想断开与AP的连接，它将向AP 发送Disassociation帧。当然STA也可以向AP发送一个Deauthentication帧彻底地断开与AP的连接。同样，当AP想断开与客户端的连接时，它将发送一个Disassociation帧给客户端。AP也可以通过广播Disassociation帧来断开与所有STA的连接。当AP或STA收到一个Disassociation帧时二者的状态将回退到状态2，当收到一个Deauthentication帧时二者的状态将回退到状态1。DOS攻击就是基于无线客户端的这几种状态进行攻击的。

images:  404254d644fc4927f7bf036741f355730a9b625d.jpg图 IEEE802.11无线认证状态转换图

### 认证洪水攻击

认证洪水攻击(Authentication Flood Attack)，即身份认证洪水攻击，通常被简称为Auth攻击，是无线网络拒绝服务攻击的一种形式。该攻击目标主要针对那些处于通过验证、和AP建立关联的关联客户端，攻击者将向AP发送大量伪造的身份验证请求帧(伪造的身份验证服务和状态代码)，当收到大量伪造的身份验证请求超过所能承受的能力时，AP将断开其它无线服务连接。

images:  2218db4aa524791bcb3c7b175b5400a25b344b3c.jpg

一般来说，所有无线客户端的连接请求会被AP记录在连接表中。当连接数量超过AP所能提供的许可范围，AP就会拒绝其它客户端发起的连接请求。

### 关联洪水攻击

关联洪水攻击（Association Flood Attack），通常被简称为Asso攻击，是无线网络拒绝服务攻击的一种形式。它试图通过利用大量模仿的和伪造的无线客户端关联来填充AP的客户端关联表，从而达到淹没AP的目的。在802.11层，共享解密身份验证有缺陷，很难再用。仅有的其他备选项就是开放身份验证(空身份验证)，该身份验证依赖于较高级别的身份验证，如802.1x或VPN。

开放身份验证允许任何客户端通过身份验证然后关联。利用这种漏洞的攻击者可以通过创建多个到达已连接或已关联的客户端来模仿很多客户端，从而淹没目标AP的客户端关联表，具体情况如下图所示。客户端关联表溢出后，接入点将不再允许更多的连接，并会因此拒绝合法用户的连接请求。此类攻击和之前提及的Authentication Flood Attack表现很相似，但是原理却不同。

images:  7f0281739e75285e8f352c00711cd0b09580a03f.jpg

### 去认证洪水攻击

去认证洪水攻击(Deauthentication Flood Attack)，全称去身份认证洪水攻击或认证阻断洪水攻击，通常被简称为Deauth攻击，是无线网络拒绝服务攻击的一种形式，它旨在通过欺骗从AP到客户端单播地址的取消身份验证帧来将客户端转为未关联的/未认证的状态。对于目前广泛使用的无线客户端适配器工具来说，这种形式的攻击在打断客户端无线服务方面非常有效和快捷。一般来说，在攻击者发送另一个取消身份验证帧之前，客户端会重新关联和认证以再次获取服务。攻击者反复欺骗取消身份验证帧才能使所有客户端持续拒绝服务。

去身份验证洪水攻击具体步骤如下：

攻击者先通过扫描工具识别出预攻击目标(无线接入点和所有已连接的无线客户端)。

通过伪造无线接入点和无线客户端来将含有Deauthentication帧注入到正常无线网络通信。

此时，无线客户端会认为所有数据包均来自无线接入点。

在将指定无线客户端“踢出”无线网络后，攻击者可以对其他客户端进行同样的攻击，并可以持续进行以确保这些客户端无法连接AP。

尽管客户端会尝试再次连接AP，但由于攻击者的持续攻击，将会很快被断开。

下图为去身份验证洪水攻击原理图，可看到攻击者对整个无线网络发送了伪造的取消身份验证报文。

images:  3076c627f29147d2633e06209adae14b2253220e.jpg

### 去关联洪水攻击

去关联洪水攻击（Disassociation Flood Attack）的攻击方式和Deauthentication Flood Attack表现很相似，但是发送数据包类型却有本质的不同。它通过欺骗从AP到客户端的取消关联帧来强制客户端成为图所示的未关联／已认证的状态（状态2）。一般来说，在攻击者发送另一个取消关联帧之前，客户端会重新关联以再次获取服务。攻击者反复欺骗取消关联帧才能使客户端持续拒绝服务。

### RF Jamming攻击

RF干扰攻击（RF Jamming Attack），是通过发出干扰射频达到破坏正常无线通信的目的。

由于目前普遍使用的无线网络都工作在2.4GHz频带范围，此频带范围包含802.llb、 802.llg、802.lln、蓝牙等，所以针对此频带进行干扰将会有效地破坏正常的无线通信，导致传输数据丢失、网络中断、信号不稳定等情况出现。

当无线黑客使用射频干扰攻击来对公司或者家庭无线网络进行攻击时，无线路由器或无 线AP将会出现较为明显的性能下降，而当遇到针对2.4GHz整个频段的阻塞干扰时，整个无线网络中的AP及无线路由器甚至都将不能正常工作。

目前的无线阻断技术，用的比较多的是去关联以及去认证技术。对网络进行实时监听，当移动用户接入非法AP中时，捕获伪AP与终端相互传送的帧，获取伪AP以及终端的MAC地址，帧序列号等信息，然后构造去关联帧或去认证帧，模拟伪AP或终端发送数据包。通过持续不断地发送数据包，使终端无法与伪AP进行连接，从而达到了阻断终端接入伪AP的目的，如下图所示：

images:  ef60e089897ebcce36b5a34cf167b5922927a7d4.jpg

0x05 参考资料
=========

* * *

*   [http://blog.chinaunix.net/uid-9525959-id-3326047.html](http://blog.chinaunix.net/uid-9525959-id-3326047.html)
*   [http://my.oschina.net/u/994235/blog/220586?fromerr=NiPT4Bd2#OSC_h2_6](http://my.oschina.net/u/994235/blog/220586?fromerr=NiPT4Bd2#OSC_h2_6)
*   [http://www.cnblogs.com/LittleHann/p/3700357.html](http://www.cnblogs.com/LittleHann/p/3700357.html)
*   [http://netsecurity.51cto.com/art/201209/358531.htm](http://netsecurity.51cto.com/art/201209/358531.htm)
*   [http://www.woaidiannao.com/html/wlgz/wxwljs/10915.html](http://www.woaidiannao.com/html/wlgz/wxwljs/10915.html)
*   《无线钓鱼接入点攻击与检测技术研究综述》