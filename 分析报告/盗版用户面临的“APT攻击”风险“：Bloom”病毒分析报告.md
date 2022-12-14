# 盗版用户面临的“APT攻击”风险 “：Bloom”病毒分析报告

0x00 概述
=======

* * *

1. 盗版软件用户和“APT 攻击”
------------------

我国电脑用户当中,使用盗版软件是非常普遍的现象,从盗版的 Windows 系统到各种收费软件的“破解版”等等。

互联网上也充斥着各种帮助用户使用盗版的“激活工具”、“破解工具”,投其所好地帮助用户使用盗版软件。但是“天下没有免费的午餐”,除了一部分破解爱好者提供的无害的免费激活工具之外,病毒制造者也瞄准了盗版人群,他们利用提供激活工具的机会,将恶性病毒植入用户电脑。

前段时间爆发的“小马激活病毒”为例,其以系统激活工具的身份为掩护,利用其“入场”时间早的天然优势,在用户电脑上屏蔽安全软件、肆意劫持流量,危害极大。同理,许多病毒制造者病毒用 PE 工具箱、系统激活工具等形式进行包装,不但加快了病毒的传播速度,也加强了其隐蔽性,从而躲避安全软件的查杀。

提到 APT,很多人会首先想到那些针对大型企业甚至政府部门的特定攻击或威胁,以及 0day、挂马、间谍等。实际上,APT(Advanced Persistent Threat,即高级持续性威胁),所指非常宽泛,即“通过较为高级的手段这对特定群体产生的持续性威胁”,因此针对盗版用户的病毒威胁,也在 APT 攻击之列。

本报告为大家揭示的病毒,就属于这类恶意威胁。该病毒在系统安装阶段 “先人一步”释放出恶意程序,并通过时间优势提前安置安全软件白名单库将病毒自身 “拉白”,再通过众多功能单一、目标明确的程序相互配合,针对 “盗版操作系统用户”这一特定群体形成了持续性的威胁。

这种针对“盗版用户”的 APT 攻击迷惑性极强,用户很难发现恶意程序。更要命的是,即使安全软件报毒,用户通常也会认为是误报了“系统程序”从而选择放过。

2. “Bloom”病毒何以长期“幸存”?
---------------------

近期,火绒在一个浏览器首页遭劫持的用户现场中提取到了一组具有“锁首”功能的恶意程序,该程序会通过修改用户浏览器快捷方式的手段劫持“灰色流量”。在获取样本后,火绒便当即对其进行了查杀,因其注册的服务名将其命名为“Bloom”病毒。

随后,我们在火绒样本库中进行关联查询,发现了该病毒在互联网中流行的两个主要版本,两个版本时间相差一年,而“第一代”病毒更是在 2014 年就已经出现。然而,截止目前为止,国内的大部分主流安全软件仍未能有效对其进行查杀。随着我们对这组样本的分析我们发现,这组样本中每个程序功能极为单一、独立,如果仅仅通过对单个样本进行分析,并不能完整的获得该病毒的整个逻辑,从而无法认定其为恶意样本。

正是因为上述原因,使得这个病毒在安全软件的眼皮底下堂而皇之地生存了至少两年之久。

0x01 样本分析
=========

* * *

“Bloom”病毒是一组通过修改浏览器快捷方式的手段劫持用户流量的恶意程序。通过火绒的“威胁情报分析系统”,我们发现该病毒通常会藏匿于以下几个路径名中:

```
C:\Program Files\Bloom Services
C:\Program Files\Supervise Services
C:\Program Files\WinRAR\RARDATA
C:\Windows\Microsoft.NET\Framework\v4.0.3032018
C:\Program Files\Windows Defender\DATA\Supervisory

```

观察上述路径名,我们发现该病毒通常会将自己伪装到一些常见的系统目录下,以此来蒙蔽用户,致使安全软件即使发现了该病毒,用户也会误以为是系统程序选择放过。

经过我们一段时间的追踪,我们发现在当前互联网环境中活跃“Bloom”病毒的两个版本,我们暂且称他们为“Bloom”病毒的“第一代”和“第二代”。如下图所示:

![](http://drops.javaweb.org/uploads/images/e7e508c11c5ee086f3126e327d08e847425bf381.jpg)

图 2-1. “Bloom”病毒“第一代”(左)与“第二代”(右)概览

两代病毒都是通过名为“360.bat”的批处理脚本进行病毒相关的初始化操作。如下图所示:

![](http://drops.javaweb.org/uploads/images/1d47da609074e8700b250c6817d9541d5267c43b.jpg)

图 2-2. “360.bat”内容概览

“Supervisory.exe”用来注册服务,服务名为“Bloom Services”,该服务每隔一分钟就会启动“Moniter.exe”,由该程序调用“Prison.exe”将浏览器快捷方式中加入网址参数,达到其劫持用户流量的目的。其在“Prison.exe”中使用的网址如下:

```
http://bd.33**38.cc –> https://web.sogou.com/?12315
http://hao.33**38.cc -> http://www.2345.com/tg21145.htm
http://www.33**38.cc -> https://web.sogou.com/?12242-0001
http://bd.4**z.com/?fw -> https://web.sogou.com/?12242-0001
http://bd.i**8.com/?fw –> http://www.2345.com/?33883
http://hao.4**z.com/?fw -> https://web.sogou.com/?12315
http://hao.i**8.com/?fw -> http://www.2345.com/tg21145.htm
http://www.4**z.com/?fw -> http://www.2345.com/tg16937.htm
http://www.i**8.com/?fw -> https://web.sogou.com/?12242-0001

```

由于“Prison.exe”中指向的网址是固定的,病毒作者为了增加“锁首”的“灵活性”,还为“第一代”病毒设置了更新程序“360TidConsole.exe”。每隔一分钟就会检测“Prison.exe” ,如果本地的病毒版本过旧,就会通过访问`http://update.qido***ashi.com/`下的“version.txt”获取最新的病毒版本。如果版本不统一,则会通过该站点下的“download.txt”获取病毒下载地址进行更新。为了增加其病毒的隐蔽性,“第一代”病毒还试图仿冒微软签名欺骗用户,如下图:

![](http://drops.javaweb.org/uploads/images/58ea08c00a473d000daa2ca62dff653a1ea1e781.jpg)

图 2-3. “第一代”病毒仿冒的微软签名展示

“第二代”病毒较前者去掉了病毒的更新功能,添加了“preservice.exe”用于结束一些主流安全软件的安装程序进程。由于病毒“入场”时间早于安全软件,在与安全软件对抗的时候大大降低的技术成本。如下图:

![](http://drops.javaweb.org/uploads/images/7e855eb87e0b0b38ec4550aa7700e2984c1b4e86.jpg)

图 2-4. “preservice.exe”结束进程概览“第二代”病毒为了绕过国内个别安全软件的查杀,每个可执行文件都加上自己的签名。如下图:

![](http://drops.javaweb.org/uploads/images/68e086bf87c78e609a9b05297f90a96c28a003da.jpg)

图 2-5. “第二代”病毒所使用的签名

经过我们的粗略分析,我们初步找到在用户计算机上流走的“灰色流量”源头就是”Bloom”病毒,该病毒集“劫持流量”、实时更新、与安全软件对抗于一身,把用户的计算机变成了帮助病毒制造者赚钱的“肉鸡”。

0x02 追根溯源
=========

* * *

通过病毒更新时使用的域名(`http://update.qido***ashi.com`),我们找到了与病毒相关的一个“U 盘启动制作工具”——“启动大师”的官网。如下图:

![](http://drops.javaweb.org/uploads/images/2a50f40200129202234d60304e5965bb641d51ef.jpg)

图 3-1. 与病毒相关的“U 盘启动制作工具”官网

通过下载和简单的文件信息查看,我们发现“启动大师”所用的签名虽然已经失效,但其与“第二代”Bloom 病毒所用的签名是相同的。所以我们初步推断,该病毒可能是被 “启动大师”所释放。

在用“启动大师”制作了 PE 启动盘后,我们进入了其预设好的 PE 系统。“启动大师”运行效果图如下:

![](http://drops.javaweb.org/uploads/images/c67434b0a0d96420d1d811fee9c51d43f85f9ca5.jpg)

图 3-2. “启动大师”运行效果图

在进入 PE 系统之后,我们发现“启动大师”的 Ghost 工具跃然于桌面,似乎有一种“恭候多时”的殷切。效果如下图:

![](http://drops.javaweb.org/uploads/images/64f2b7adf39d6161ed4cc398fb0a766d56327b99.jpg)

图 3-3. 使用“启动大师”制作的 PE 系统效果图

我们随即找到了该程序所在的位置,如下图:

![](http://drops.javaweb.org/uploads/images/f8e79c8af453920241fbe1f4d105e779fa61ad07.jpg)

图 3-4. 与病毒相关的 Ghost 还原程序总览

我们在该目录下,发现了很多号称是 Ghost 工具的程序和一些可疑的文件,我们对如上文件展开了详细分析。

其中有三个自称是 Ghost 工具的文件,其中“ghost32.exe”为正常的 Ghost 还原工具,其余的两个“qddsghost.exe”和“ghost.exe”可能都和病毒有关,我们下面将对它们进行详细介绍。

首先,我们对“qddsghost.exe”进行了分析,发现其不但会使用命令行调用真正的Ghost 还原程序“ghost32.exe”,还会在还原后系统中释放与修改首页和修改各种浏览器的收藏夹的相关文件。

“qddsghost.exe”所释放的文件都存在其资源中,资源 Type(EXEFILE)_Name(AD)_Lang(0x804)是一个批处理文件(下文称“ad.bat”)其起到了整体的调度分配作用。该批处理文件内容如下:

![](http://drops.javaweb.org/uploads/images/1b52f56789741707daeaff0790f2d8a132b960c8.jpg)

图 3-5. “ad.bat”内容总览

我们将该批处理的内容包括四个部分:

1) 浏览器“锁首”:其使用命令行修改了一些主流浏览器(360 安全浏览器、QQ浏览器、2345 浏览器等)的默认首页,并且进行了用户系统的注册表管理器,以防止用户修复首页。资源中包含的压缩包中包含着几种浏览器配置文件,替换配置文件后的浏览器默认首页和收藏夹就会被添加和修改。

2) 释放“首页劫持”快捷方式:`Type(EXEFILE)_Name(IK1)_Lang(0x804)`、`Type(EXEFILE)_Name(IK2)_Lang(0x804)`、`Type(EXEFILE)_Name(IK3)_Lang(0x804)`三个资源都是网址链接文件,病毒调用的批处理会将他们释放到用户文件夹下的Favorites 文件夹中。上述网址链接指向的推广网址如下:

```
http://www.ha**9.com/?1
http://www.ha**5.com/?1
http://www.taobao.com/go/chn/tbk_channel/channelcode.php?pid=mm_26101802_0_0&eventid=101329

```

3) 释放 Apache、搭建本地 PHP 跳转页面“黑吃黑”:其先下载`http://host.66***5.com/index3.txt`中存放的 hosts 文件,之后通过修改系统 hosts 文件将其过滤列表中的网址 IP 改为“127.0.0.1”(其搭建的跳转页),最后通过跳转页跳转至推广页面。

![](http://drops.javaweb.org/uploads/images/7ad31e4799cc119ae960f1be7f244ef57ddca979.jpg)

图 3-6. 病毒在本地搭建的 PHP 跳转页面

代码中过滤的大部分网址都为赌博、色情等非法网址,其通过本地过滤的方式借其他”灰色流量”为自己赚取流量,打压“黑色产业链”中的“同行”,借他们的手为自己赚钱。

4) 释放安全软件的白名单库:`Type(EXEFILE)_Name(BAIDUSD7)_Lang(0x804)`和`Type(EXEFILE)_Name(QQPCMGR)_Lang(0x804)`都是压缩包,前者压缩的文件是百度杀毒的白名单数据库,后者为腾讯电脑管家的白名单数据库及相关配置。

![](http://drops.javaweb.org/uploads/images/52bb2fcb319b0a104a89dc5a8b708117a6533060.jpg)

图 3-7. 百度杀毒白名单数据库设置

由于该病毒的“入场”时间早于安全软件,当安全软件安装时,会误认为这些白名单配置为之前安装后保留的配置,使得病毒可以通过修改安全软件白名单的方法直接绕过安全软件查杀。

5) 病毒使用的一些基础工具,包含命令行版的 7z 程序包等。

在“qddsghost.exe”启动之后还会调用“srcdll.dll”动态库中的导出函数“`shifanzyexedll`”。在该动态库中,我们也发现了很多资源,除了修改首页的批处理文件之外,其余全部都是可执行文件。

下图中显示了 srcdll.dll 文件释放出来的文件之间的相互关系,其首先会释放最上边的三个深色的可执行文件,通过对这三个文件的运行和分析,我们又得到了四组恶意程序。

![](http://drops.javaweb.org/uploads/images/2975d681d3c592ec8d6d02548030605775742127.jpg)

图 3-8. srcdll.dll 释放文件关系图(图中标号与下文对应)

第一组:“QQBrowser.exe”和“QQBrowser.reg”。可执行文件会将注册表文件导入到系统中,其将 QQ 浏览器的首页修改为病毒的跳转页。

第二组:“Home.exe”和“pic.exe”都会修改 IE 首页,“pf.exe”是释放出了很多网址链接文件和加了网址参数浏览器快捷方式用于其进行“流量变现”,在 WIN7 系统下其还会释放“淘宝商城.exe”,该程序会打开带有推广付费号的“爱淘宝”网址链接。“pf.exe”程序运行效果图如下:

![](http://drops.javaweb.org/uploads/images/94bf00074cac19e4a7a75bc13e635194b4868314.jpg)

图 3-9. “pf.exe”运行效果图

第三组:“sdff.exe”运行后会在桌面上建立了“story.exe”和“tutule.exe”的快捷方式,其两者会跳转到两个不同的推广页。除了释放上述文件和快捷方式外,其还会释放出“Bloom”病毒,每隔一段时间就会修改用户的浏览器快捷方式中加入推广网址。

第四组:“Project1.exe”会将“360safte.dll ”注册为 BHO(Browser Helper Object)插件,其会劫持浏览器访问的网址。该插件会根据自己在`http://bho.66***5.com/config.txt`上的劫持列表,在浏览器访问其中的网址时将其劫持为带有推广付费号的网址链接。如果系统中还有其他的程序在进行广告推广时,这些流量最终都会流进病毒作者的“腰包”,使得该病毒不但成为了其截取流量的工具,也成为了其用来“黑吃黑”的牟利工具。

![](http://drops.javaweb.org/uploads/images/e9bf1391b43ad47683f52e5b1a1dca58b053a243.jpg)

图 3-10. 病毒劫持网址列表

通过对以上四组文件的分析,我们发现了“启动大师”释放的所有病毒都是为了进行“流量变现”的,其所涉及的大型网站之多让人不禁嗔目结舌。以上四组病毒所涉及的网址站点如下:

*   百度(www.baidu.com)
*   淘宝聚划算(ju.taobao.com)
*   天猫商城(jx.tmall.com)
*   爱淘宝(ai.taobao.com)
*   京东商城(www.jd.com)
*   一号店(www.yhd.com)
*   搜狗网址导航(web.sougou.com)
*   2345 网址导航(www.2345.com)
*   Hao123 网址导航(www.hao123.com)
*   黄金屋(www.35kxs.com)
*   女人街(www.womenjie.com)

在使用“启动大师”之后,其所释放的众多程序通过相互配合源源不断地为其作者劫持“灰色流量”,通过高隐蔽性的伪装对“盗版用户”造成了高级的、可持续性威胁,使用户成为了病毒作者刷取“灰色流量”的“肉鸡”。用户在搜索、网购、浏览互联网媒体信息时,这些非法流量会将源源不断地计入这些病毒所用的计费账号。“启动大师”的“病毒制作团队”以 PE 工具箱为诱饵,借助用户安装盗版系统的“刚需”将病毒在系统还原的时间点释放到用户的计算机中,以多种形式进行“流量变现”,严重侵害了用户切身利益。对于该病毒的上述恶意行为,针对与该病毒相关的所有样本火绒已经率先进行查杀。

暗藏在“灰色流量”源头的幕后黑手仿佛已经浮出水面,但是究竟谁才是这种“疯狂套利”现象背后的推手?为何用户长时间处于病毒威胁之中却又无人过问?我想这是值得我们每个人深思的问题。

0x03 结论
=======

* * *

随着国内互联网环境的日益复杂,如今的用户所面对环境的复杂程度史无前例。内有“启动大师”这样的“流氓当道”,外有勒索病毒一类高威胁性病毒对我们的虎视眈眈,对于一个普通用户而言,能够保证自己远离这些威胁已经实属不易。但是随着当今互联网中“套利方式”的不断翻新,每天互联网中都会出现新的病毒、流氓软件威胁用户的信息安全和切身利益。

如今的互联网环境之下,用户电脑在无孔不入的病毒面前很容易失守,再加之国内的大型互联网公司本身对流量有着长期、迫切的需求,使得病毒和流氓软件的制造者疯狂的在攻击手段和套路上无所不用其极。

互联网环境日益复杂,而国内的互联网企业对于网络安全的重视程度有待提升,从而造成“黑色产业链”的参与者也加入了国内的互联网大潮,造成了当前互联网中恶意竞争事件屡见不鲜。一些互联网公司受到了这些“快速变现”的“邪门歪道”的影响,逐渐的也把重点放在了想方设法获取用户浏览,促进公司业绩上,对于见效慢、盈利周期长的主营业务则渐渐降低了成本,以此为循环不断地追求高绩效、高收益。而这些不也正是当前国内经济市场存在“产能过剩”的根源吗?长此以往定会影响国内互联网的健康发展。过剩的“灰色流量”不但降低了用户的使用体验,也大大地影响的急需广告推广的互联网企业的企业形象及推广质量。火绒在此呼吁广大互联网企业,在广告推广的同时加强审核力度,将“灰色流量”扼杀在源头。

0x04 附录
=======

* * *

文中涉及样本 SHA1

![](http://drops.javaweb.org/uploads/images/c892cdbde9e2d2662735e653bda9e276a3ea4e59.jpg)