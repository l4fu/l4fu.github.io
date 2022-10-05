# goby资产收集
[![](_v_images/386683309211616.png)](https://p0.ssl.qhimg.com/t0128e803dfaf2921ca.png)

**前言：** 之前在进行 Web 打点的资产收集时，总是手动的收集父子公司、手动或半自动化收集根域、子域、C 段，当目标资产较多或多个目标时往往非常繁琐。

值得高兴的是，Goby 内测版发布了一个新功能，叫 ”IP 库“，可以自动化进行资产收集的各个环节，当然不仅限于上述几个方面，还包括 HTTS 证书、图标、关键字等资产收集方式，进一步详细查看官方发布的演示视频，发现其功能涵盖了目前资产收集常见的各个方面，集成了此功能后，前渗透 Web 攻击中的资产收集、指纹识别、漏洞检测、口令检测 Goby 都已涵盖，是一个较全面的工具。

本文的定位是 Goby 新功能 ”IP 库“ 的基本原理讲解及图文教程，会通过一个案例 kingsoft.com 讲述如何使用 ”IP 库” 进行基本的资产收集。

## 0×01 基本原理

大佬可以略过，之前不了解资产收集的可以看一下，不然容易搞不懂 Goby 的每一步在干什么。

常规的资产收集包括：公司备案名查询、父子公司收集、根域名收集、子域名收集、 C 段收集。

1.  公司备案名查询：通过 ICP 备案查询根域名对应的公司备案名
2.  父子公司收集：通过天眼查中的股权关系查询备案公司的父子公司
3.  根域名收集：对收集到的公司名，查询每一个公司的全部根域名
4.  子域名收集：对收集到的根域名，收集每一个根域名的全部子域名
5.  C 段收集：对收集到的子域名，收集每一个子域名的 C 段

## 0×02 图文教程

需要 >=1.9.304 的版本，本文使用 v1.9.307，如下图

[![](_v_images/382633309221086.png)](https://p0.ssl.qhimg.com/t0134f2c81d412dcd43.png)

点击右侧的 IP Lib，如下图

[![](_v_images/375603309227952.png)](https://p0.ssl.qhimg.com/t01c28485cfa31766af.png)

点击后需要登录才可访问此功能，没有账号的需要注册一下，登录后如下图

[![](_v_images/367553309234406.png)](https://p2.ssl.qhimg.com/t01e744532ac231578d.png)

左上角的个图标依次是

[![](_v_images/359513309239270.png)](https://p5.ssl.qhimg.com/t012d77aec8314bd1b2.png)

点击 start，会弹出如下提示框，Type 选择 “domain”， Name 填写 “kingsoft.com”

[![](_v_images/347463309216830.png)](https://p0.ssl.qhimg.com/t01bcc11dfcf5a4615e.png)

点击 Add 后，会生成一个名为 domain 的范围域，包含 2 个节点，将鼠标移到节点上会发现，节点名分别为 domainRoot 和 kingsoft.com，同时操作会显示在右侧的 log 栏中，如下图

[![](_v_images/341433309235589.png)](https://p1.ssl.qhimg.com/t01395ceb6b4991b9c6.png)

右键点击节点 kingsoft.com->Import icp，会生成 kingsoft.com 对应公司的备案名和备案号，如下图

[![](_v_images/334373309238087.png)](https://p2.ssl.qhimg.com/t01016fa4ed5473c8b6.png)

在范围域 company 中，右键点击节点北京金山数字娱乐科技有限公司->Import company->Goby，会弹出如下窗口

[![](_v_images/328333309240483.png)](https://p3.ssl.qhimg.com/t01d202cb2d00189976.png)

展开公司并勾选全部父子公司，点击 Add，会导入全部的父子公司，如下图

[![](_v_images/324283309222603.png)](https://p4.ssl.qhimg.com/t014a29d334c1f82abe.png)

在范围域 company 中，右键点击名为 companyRoot 的节点->Import all icp->All，导入全部父子公司的 ICP 备案号，如下图

[![](_v_images/318243309226849.png)](https://p1.ssl.qhimg.com/t012dab931e04856926.png)

再根据备案号查询全部根域，右键点击范围域 icp 中名为 icpRoot 的节点->Import all domain->All，此时可以看到，在范围域 domain 中多出了好多根域名，此域名即为金山父子公司的根域名，如下图

[![](_v_images/311213309230294.png)](https://p3.ssl.qhimg.com/t014cf1ac088f88c0d2.png)

最后根据全部根域查询子域，右键点击范围域 domain 中名为 domainRoot 的节点->Import all subdomain/ip->All，右下角可以看到，当前共收集了 3 个根域、212 个子域、2 个icp、236 个ip、6 个公司、当前界面显示 90 个节点、全部节点数 948 个，如下图

[![](_v_images/303163309237625.png)](https://p0.ssl.qhimg.com/t0194920a8f71a3ac49.png)

最后点击左上角的导出功能，可将数据导入到 Goby 中进行扫描，也可导出到 Excel 中，如下图

[![](_v_images/289093309217459.png)](https://p4.ssl.qhimg.com/t01e25131bceb723774.png)

[![](_v_images/256963309229592.png)](https://p0.ssl.qhimg.com/t0193fe9647a8eb0fa5.png)

## 0×03 小结

上述的一系列操作，梳理如下：

通过根域名获取公司备案名、备案号 -> 通过公司备案名获取父子公司 -> 对收集到的全部公司名收集备案号 -> 对收集到的全部备案号收集根域名 -> 对收集到的全部根域名收集子域名及 C 段

如果每一步都去手动或用工具收集，效率会非常慢，利用 Goby 中的 “IP 库” 进行资产收集就方便了很多，你值得拥有。

IP 库支持版本：Goby 版本 ≥ 1.9.304。目前内测版用户可以不限制节点使用，还在一个月有效期内，冲鸭！！！

文章来自Goby社区成员：ybdt，转载请注明出处。  
下载Goby内测版，请关注公众号：Gobysec  
下载Goby正式版，请访问官网：[http://gobies.org](http://gobies.org/)