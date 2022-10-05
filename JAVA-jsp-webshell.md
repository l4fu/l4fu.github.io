# java-jsp_webshell
目前很多大型厂商都选择使用Java进行Web项目的开发，近年来随着各种JAVA指定环境RCE漏洞的出现，Java Web的安全逐渐被人们所重视，与漏洞相关的还有用于后期维持权限的Webshell。与PHP不同的是，JSP的语言特性较为严格，属于强类型语言，并且在JDK9以前并没有所谓的eval函数。一般而言JSP的变形免杀较为困难，但是依旧存在很多的”黑魔法”。

不知攻，焉知防。阿里云安骑士Webshell检测系统在迭代升级过程中，除了内部的不断绕过尝试以外，也长期邀请大量白帽子进行持续的绕过测试。经过不断总结沉淀在JSP Webshell查杀引擎方面我们形成了基于字节码跟反汇编代码的检测方式，可以有效对抗云上高强度对抗性样本。

本文分为函数调用篇/战略战术篇/内存马篇/降维打击篇四个部分，将从攻击者的角度与大家一起分享JSP Webshell的攻击姿势。

## 关于JSP

JSP全称”Java Server Page”，其本质是一种Java Servlet。

JSP在第一次被访问的时候会先被翻译成Java文件，这个步骤由Tomcat等web容器完成；接着Java文件会被编译成JVM可以识别的class文件，这个步骤由JDK完成。

## 函数调用篇

[![](_v_images/20200820092041341_24239.png)](https://p2.ssl.qhimg.com/t012554bfb5fee20d29.png)

### 直接调用

常见的直接调用是通过 java.lang.Runtime#exec和java.lang.ProcessBuilder#start

#### java.lang.Runtime

[![](_v_images/20200820092041130_23576.png)](https://p1.ssl.qhimg.com/t014f67a1efaaae2159.png)

#### java.lang.ProcessBuilder

[![](_v_images/20200820092040918_19675.png)](https://p4.ssl.qhimg.com/t013ee3f9a1f4150d5c.png)

### 反射调用

反射可以说是Java中最强大的技术，很多优秀的框架都是通过反射完成的。一般的类都是在编译期就确定下来并装载到JVM中，但是通过反射我们就可以实现类的动态加载。如果查阅源码可以发现，图中提到的很多命令执行方式的底层都是反射。

因为反射可以把我们所要调用的类跟函数放到一个字符串的位置，这样我们就可以利用各种字符串变形甚至自定义的加解密函数来实现对恶意类的隐藏。

[![](_v_images/20200820092040706_24403.png)](https://p5.ssl.qhimg.com/t012dc5b91092bc7063.png)

除此以外，反射可以直接调用各种私有类方法，文章接下来的部分会让大家进一步体会到反射的强大。

### 加载字节码

说到加载字节码就必须提到java.lang.ClassLoader这个抽象类，其作用主要是将 class 文件加载到 jvm 虚拟机中去，里面有几个重要的方法。

- loadClass()，加载一个类，该方法会先查看目标类是否已经被加载，查看父级加载器并递归调用loadClass()，如果都没找到则调用findClass()。
- findClass()，根据类的名称或位置加载.class字节码文件，获取字节码数组，然后调用defineClass()。
- defineClass()，将字节码加载到jvm中去，转化为Class对象

> 更详细的说明可以参考这篇文章：[https://zhuanlan.zhihu.com/p/103151189](https://zhuanlan.zhihu.com/p/103151189)

#### 调用defineClass

提到defineClass就想到了冰蝎，冰蝎可以说是第一个实现JSP一句话的Webshell管理工具。其中defineClass这个函数是冰蝎实现的核心。

因为java在1.8以前并没有像php的eval函数，所以要实现动态执行payload就要另外想办法。因为java世界中所有的执行都是依赖于字节码，不论该字节码文件来自何方，由哪种编译器编译，甚至是手写字节码文件，只要符合java虚拟机的规范，那么它就能够执行该字节码文件。所以如果可以让服务端做到动态地将字节码解析成Class，就可以实现“JSP一句话”的效果。

正常情况下，Java并没有提供直接解析class字节数组的接口。不过classloader内部实现了一个protected的defineClass方法，可以将byte\[\]直接转换为Class。但是因为该方法是protected的，我们没办法在外部直接调用。这里就有两种处理办法：

第一种是继承，直接自定义一个类继承classloader，然后在子类中调用父类的defineClass方法。这种方式比较简单，所以原版冰蝎中采用的这种办法。

第二种是反射，通过反射来修改保护属性，从而调用defineClass。

以下为蚁剑基于冰蝎的原理实现的JSP一句话样本。利用ClassLoader类中的defineClass，我们就可以把一个自定义的类传入并加载。

[![](_v_images/20200820092040289_8136.png)](https://p1.ssl.qhimg.com/t01bcdee472f64469e8.png)

#### BCEL字节码

这个就是一个比较神奇的类了，可以直接通过classname来进行字节码的加载。

[![](_v_images/20200820092040076_32534.png)](https://p0.ssl.qhimg.com/t0129cad4d00d042d6a.png)

查看loadClass方法的源码，发现会判断传入的bcelcode是否有”$$BCEL$$”这个字符串，就会将后面的内容转换成标准字节码，然后使用defineClass进行加载。

  protected Class loadClass(String class_name, boolean resolve)
    throws ClassNotFoundException
  {
...
      if(cl == null) {
        JavaClass clazz = null;
        /\* Third try: Special request?
         */
        if(class_name.indexOf("$$BCEL$$") >= 0)
          clazz = createClass(class_name);
        else { // Fourth try: Load classes via repository
          if ((clazz = repository.loadClass(class_name)) != null) {
            clazz = modifyClass(clazz);
          }
          else
            throw new ClassNotFoundException(class_name);
        }
        if(clazz != null) {
          byte\[\] bytes  = clazz.getBytes();
          cl = defineClass(class_name, bytes, 0, bytes.length);
        } else // Fourth try: Use default class loader
          cl = Class.forName(class_name);
      }
      if(resolve)
        resolveClass(cl);
    }

#### URLClassLoader远程加载

URLClassLoader是ClassLoader的子类，它用于从指定的目录或者URL路径加载类和资源。当URL里的参数是由”http://”开头时，会加载URL路径下的类。

[![](_v_images/20200820092039456_31924.png)](https://p0.ssl.qhimg.com/t01944577d6a0025cb2.png)

#### URLClassLoader本地加载

当URL里的参数是由”file://”开头时，会加载本地路径下的类。

由于加载的字节码是固定的并且不可直接修改，没办法直接实现对命令的动态解析。要么配合冰蝎一样的客户端，每次都调用ASM等字节码框架动态生成字节码传过去，要么就想其他办法把我们要执行的指令传递进去。

这个例子利用了一个很巧妙的方法：把收到的指令拼凑成源代码后直接在服务端进行编译，然后写入到本地文件中，再利用URLClassLoader对写入的文件进行加载。

[![](_v_images/20200820092039243_13389.png)](https://p2.ssl.qhimg.com/t01c17c8fe3ddbbbc8e.png)

### 表达式类调用

#### ScriptEngineManager

通过ScriptEngineManager这个类可以实现Java跟JS的相互调用，虽然Java自己没有eval函数，但是ScriptEngineManager有eval函数，并且可以直接调用Java对象，也就相当于间接实现了Java的eval功能。但是写出来的代码必须是JS风格的，不够正宗，所以将这部分归类为“表达式类调用”部分。

[![](_v_images/20200820092038922_22020.png)](https://p2.ssl.qhimg.com/t016c5d0c7d5fb8c339.png)

#### EL表达式

> 表达式语言（Expression Language），或称EL表达式，简称EL，是Java中的一种特殊的通用编程语言，借鉴于JavaScript和XPath。主要作用是在Java Web应用程序嵌入到网页（如JSP）中，用以访问页面的上下文以及不同作用域中的对象 ，取得对象属性的值，或执行简单的运算或判断操作。EL在得到某个数据时，会自动进行数据类型的转换。
> 
> [https://blog.csdn.net/FZW_Faith/article/details/54235104](https://blog.csdn.net/FZW_Faith/article/details/54235104)

除了ScriptEngineManager以外，ELProcessor也有自己的eval函数，并且可以调用Java对象执行命令。

[![](_v_images/20200820092038708_8875.png)](https://p5.ssl.qhimg.com/t019373232e9693e324.png)

#### Expression

java.beans.Expression同样可以实现命令执行，第一个参数是目标对象，第二个参数是所要调用的目标对象的方法，第三个参数是参数数组。这个类的优势是可以把要执行的方法放到一个字符串的位置，不过限制就是第一个参数必须是Object。不过我们可以配合反射将Runtime类的关键字给隐藏掉。

[![](_v_images/20200820092038389_2217.png)](https://p3.ssl.qhimg.com/t01b2f9fed274ef83dc.png)

[![](_v_images/20200820092038075_5714.png)](https://p1.ssl.qhimg.com/t01df6811c09173ca16.png)

[![](_v_images/20200820092037863_16476.png)](https://p3.ssl.qhimg.com/t018596e497bba68ddc.png)

除了上面提到的以外还有OGNL(Struct)，SpEL(Spring)等表达式，但不是jdk自带的，在这里不予分析。

### 反序列化

序列化的过程是保存对象的过程，与之相反的，反序列化就是把对象还原的过程。在这里提到的反序列化并不仅仅指直接ObjectInputStream读入二进制流，利用XML/XSLT同样可以使保存的对象还原，达到反序列化的目的。

#### 重写ObjectInputStream的resolveClass

[![](_v_images/20200820092037653_31213.png)](https://p3.ssl.qhimg.com/t01119ac62de86fc7d7.png)

#### XMLDecoder

XMLDecoder可以将XMLEncoder创建的xml文档内容反序列化为一个Java对象，研究过Weblogic系列漏洞的同学对这个类一定不陌生。通过传入恶意的XML文档即可实现任意命令的执行。

[![](_v_images/20200820092037337_7305.png)](https://p0.ssl.qhimg.com/t0111f207048b340365.png)

#### XSLT

XSL 指扩展样式表语言（EXtensible Stylesheet Language）, 它是一个 XML 文档的样式表语言。通过构建恶意的模板让Webshell来解析，同样可以达到命令执行的目的。

[![](_v_images/20200820092037125_18711.png)](https://p3.ssl.qhimg.com/t01c7efdd1874abd20e.png)

### JNDI注入

> JNDI (Java Naming and Directory Interface) 是一组应用程序接口，它为开发人员查找和访问各种资源提供了统一的通用接口，可以用来定位用户、网络、机器、对象和服务等各种资源。比如可以利用JNDI在局域网上定位一台打印机，也可以用JNDI来定位数据库服务或一个远程Java对象。JNDI底层支持RMI远程对象，RMI注册的服务可以通过JNDI接口来访问和调用。

提到jndi注入就想到了fastjson，通过lookup一个恶意的远程Java对象即可达到任意命令执行。相关的文章已有很多，这里不再赘述。

[![](_v_images/20200820092036806_7715.png)](https://p5.ssl.qhimg.com/t010b6cfc3d8acd5865.png)

### JNI调用

JNI全称 Java Native Interface，通过JNI接口可以调用C/C++方法，同样可以实现命令执行的目的。

> 详细介绍：[https://javasec.org/javase/JNI/](https://javasec.org/javase/JNI/)

[![](_v_images/20200820092036593_29709.png)](https://p2.ssl.qhimg.com/t0165eb0e852b01b23d.png)

### JShell

JShell 是 Java 9 新增的一个交互式的编程环境工具。与 Python 的解释器类似，可以直接输入表达式并查看其执行结果。

[![](_v_images/20200820092036380_32448.png)](https://p1.ssl.qhimg.com/t013cb3a8e011f70028.png)

但是由于JDK8跟JDK9之间更改幅度较大，目前来说并没有普遍使用，所以暂时实战效果并不明显。

## 战略战术篇

由于Java面向对象的特性，几乎每个类都不是独立的，背后都是有一系列的继承关系。查杀引擎可能会识别常见的恶意类，但是我们就可以通过查找恶意类的底层实现或者高层包装类进行绕过，从而实现Webshell的免杀。

### 向下走–寻找底层实现类

这里以常见的Runtime类跟Expression类为例

#### ProcessImpl

查看Runtime类中exec方法的源码，可以发现exec实际上调用了ProcessBuilder的start方法

[![](_v_images/20200820092036170_23790.png)](https://p0.ssl.qhimg.com/t01865a01f3781f6201.png)

进一步查看ProcessBuilder可以发现是触发了java.lang.ProcessImpl的start方法

[![](_v_images/20200820092035859_17314.png)](https://p3.ssl.qhimg.com/t01427e55b429d24e63.png)

跟进ProcessImpl的start发现最后调用了其构造方法。

[![](_v_images/20200820092035547_23592.png)](https://p4.ssl.qhimg.com/t01ff7dacd14c12af39.png)

看一下ProcessImpl的构造方法是private类型的，并且没有任何共有构造器，所以直接实例化ProcessImpl就会报错。

[![](_v_images/20200820092035236_15263.png)](https://p4.ssl.qhimg.com/t01e6c8e21655f63d37.png)

在Java中，如果想要阻止一个类直接被实例化一般有两种方法，一种是直接把类名用private修饰，另一种是只设置私有的构造器。虽然我们不能直接new一个ProcessImpl，但是可以利用反射去调用非public类的方法。

[![](_v_images/20200820092034924_24759.png)](https://p1.ssl.qhimg.com/t017ad3922b21cecebb.png)

#### Statement

上文中提到了Expression的getValue方法可以实现表达式的执行，看一下他的源码的内容

发现Expression类继承了Statement，并且再构造函数中调用的也是父类的构造函数

[![](_v_images/20200820092034612_11522.png)](https://p0.ssl.qhimg.com/t013403af6c536a9443.png)

[![](_v_images/20200820092034402_6751.png)](https://p5.ssl.qhimg.com/t0154e4d9cb3ca87f0d.png)

查看getValue方法，发现调用了父类的invoke函数

[![](_v_images/20200820092034092_22752.png)](https://p5.ssl.qhimg.com/t014b36cb370a236a2b.png)

查看invoke函数，跳转到了java.beans.Statement#invoke

[![](_v_images/20200820092033882_30484.png)](https://p3.ssl.qhimg.com/t01e573fd57fcaf28b9.png)

跟进java.beans.Statement#invokeInternal发现底层的实现其实就是反射

[![](_v_images/20200820092033570_13167.png)](https://p4.ssl.qhimg.com/t012732d9c77862eb02.png)

综上所述，Expression的getValue实际上是调用了Statement类的invoke()函数，再通过一系列的反射实现表达式的计算。但是invoke函数不是public类型的，不能直接调用。但是我们可以发现同类中的java.beans.Statement#execute方法调用了invoke，且同时满足是public类型，可以直接调用。Statement类也是public的，可以直接new，所以我们就可以构造出一个新的利用方式。

[![](_v_images/20200820092033254_27172.png)](https://p3.ssl.qhimg.com/t0141a2e854729926dc.png)

[![](_v_images/20200820092033044_21864.png)](https://p0.ssl.qhimg.com/t011944ee88d5a054f9.png)

#### ELManager

查看ELProcessor的eval的底层实现，找到javax.el.ELProcessor#getValue

[![](_v_images/20200820092032834_17120.png)](https://p0.ssl.qhimg.com/t011e022f2b232c3be7.png)

其实是调用了this.factory的createValueExpression方法，跟进this.factory发现是ELProcessor类的构造方法中通过ELManager.getExpressionFactory()获取的。

[![](_v_images/20200820092032624_18553.png)](https://p1.ssl.qhimg.com/t016ad54f7bd5147559.png)

所以就可以构造如下形式进行绕过。

[![](_v_images/20200820092032413_6615.png)](https://p2.ssl.qhimg.com/t0196a0406a55b628b1.png)

### 向上走–寻找调用跟包装类

既然可以用底层类来绕过，那么我们当然可以寻找哪些类对我们的恶意类进行了调用跟包装。

#### sun.net.[www.MimeLauncher](https://www.anquanke.com/post/id/www.MimeLauncher)

从源码中可以看到sun.net.[www.MimeLauncher#run](https://www.anquanke.com/post/id/www.MimeLauncher#run)方法中最后调用了Runtime类的exec方法

[![](_v_images/20200820092032198_31054.png)](https://p2.ssl.qhimg.com/t01008d98948bd22c0a.png)

但是这个类是package-private修饰的，所以不能直接调用。不过没关系，我们还有反射。

[![](_v_images/20200820092031987_24289.png)](https://p1.ssl.qhimg.com/t0131a728df70eb673c.png)

构造所需参数，然后通过反射调用run方法

[![](_v_images/20200820092031676_23485.png)](https://p1.ssl.qhimg.com/t016157c83e3f5a05d9.png)

[![](_v_images/20200820092031453_10129.png)](https://p1.ssl.qhimg.com/t01bf0ac373b75705e4.png)

在源码中grep一下关键字可以看到同样的类还有几个，这里不再赘述。

[![](_v_images/20200820092031236_14875.png)](https://p2.ssl.qhimg.com/t01cd12beff4d9417b1.png)
## 内存马篇

内存马主要利用了Tomcat的部分组件会在内存中长期驻留的特性，只要将我们的恶意组件注入其中，就可以一直生效，直到容器重启。

本部分主要讲一讲三种Tomcat内存Webshell。

### 基础知识

#### Container – 容器组件

> 引用自：
> 
> [https://www.freebuf.com/articles/system/151433.html](https://www.freebuf.com/articles/system/151433.html)

Tomcat 中有 4 类容器组件，从上至下依次是：

1. Engine，实现类为 org.apache.catalina.core.StandardEngine
2. Host，实现类为 org.apache.catalina.core.StandardHost
3. Context，实现类为 org.apache.catalina.core.StandardContext
4. Wrapper，实现类为 org.apache.catalina.core.StandardWrapper

“从上至下” 的意思是，它们之间是存在父子关系的。

- Engine：最顶层容器组件，其下可以包含多个 Host。
- Host：一个 Host 代表一个虚拟主机，其下可以包含多个 Context。
- Context：一个 Context 代表一个 Web 应用，其下可以包含多个 Wrapper。
- Wrapper：一个 Wrapper 代表一个 Servlet。

#### Filter Servlet Listener

- Servlet：servlet是一种运行服务器端的java应用程序，具有独立于平台和协议的特性，并且可以动态的生成web页面，它工作在客户端请求与服务器响应的中间层。Servlet 的主要功能在于交互式地浏览和修改数据，生成动态 Web 内容。
- Filter：filter是一个可以复用的代码片段，可以用来转换HTTP请求、响应和头信息。Filter无法产生一个请求或者响应，它只能针对某一资源的请求或者响应进行修改。
- Listener：通过listener可以监听web服务器中某一个执行动作，并根据其要求作出相应的响应。

#### 三者的生命周期

> 参考自
> 
> [https://mp.weixin.qq.com/s/whOYVsI-AkvUJTeeDWL5dA](https://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652474966&idx=1&sn=1c75686865f7348a6b528b42789aeec8&scene=21#wechat_redirect)

Servlet ：Servlet 的生命周期开始于Web容器的启动时，它就会被载入到Web容器内存中，直到Web容器停止运行或者重新装入servlet时候结束。这里也就是说明，一旦Servlet被装入到Web容器之后，一般是会长久驻留在Web容器之中。

- 装入：启动服务器时加载Servlet的实例
- 初始化：web服务器启动时或web服务器接收到请求时，或者两者之间的某个时刻启动。初始化工作有init()方法负责执行完成
- 调用：从第一次到以后的多次访问，都是只调用doGet()或doPost()方法
- 销毁：停止服务器时调用destroy()方法，销毁实例

Filter：自定义Filter的实现，需要实现javax.servlet.Filter下的init()、doFilter()、destroy()三个方法。

- 启动服务器时加载过滤器的实例，并调用init()方法来初始化实例；
- 每一次请求时都只调用方法doFilter()进行处理；
- 停止服务器时调用destroy()方法，销毁实例。

Listener：以ServletRequestListener为例，ServletRequestListener主要用于监听ServletRequest对象的创建和销毁,一个ServletRequest可以注册多个ServletRequestListener接口。

- 每次请求创建时调用requestInitialized()。
- 每次请求销毁时调用requestDestroyed()。

最后要注意的是，web.xml对于这三种组件的加载顺序是：listener -> filter -> servlet，也就是说listener的优先级为三者中最高的。

#### ServletContext跟StandardContext的关系

Tomcat中的对应的ServletContext实现是ApplicationContext。在Web应用中获取的ServletContext实际上是ApplicationContextFacade对象，对ApplicationContext进行了封装，而ApplicationContext实例中又包含了StandardContext实例，以此来获取操作Tomcat容器内部的一些信息，例如Servlet的注册等。

通过下面的图可以很清晰的看到两者之间的关系

[![](_v_images/20200820092121069_28241.png)](https://p5.ssl.qhimg.com/t0143fd7056b3eb1b2e.png)

#### 如何获取StandardContext

- 由ServletContext转StandardContext

如果可以直接获取到request对象的话可以用这种方法

[![](_v_images/20200820092120645_25234.png)](https://p3.ssl.qhimg.com/t011df9153c20898e2e.png)

- 从线程中获取StandardContext

如果没有request对象的话可以从当前线程中获取

> [https://zhuanlan.zhihu.com/p/114625962](https://zhuanlan.zhihu.com/p/114625962)

- 从MBean中获取

> [https://scriptboy.cn/p/tomcat-filter-inject/](https://scriptboy.cn/p/tomcat-filter-inject/)
> 
> [![](_v_images/20200820092120330_17156.png)](https://p5.ssl.qhimg.com/t01e41ca8a2a5a989b8.png)

### Filter型

#### 注册流程

首先我们看下正常的一个filter的注册流程是什么。先写一个filter，实现Filter接口。

```
package com.yzddmr6;

import javax.servlet.*;
import java.io.IOException;

public class filterDemo implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("Filter初始化创建....");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        System.out.println("进行过滤操作......");
        // 放行
        chain.doFilter(request, response);
    }
    @Override
    public void destroy() {

    }

}
```

在web.xml中添加filter的配置

[![](_v_images/20200820092119705_11182.png)](https://p2.ssl.qhimg.com/t010c18c366a1b131f6.png)

然后调试看一下堆栈信息，找到filterChain生效的过程

[![](_v_images/20200820092119489_24118.png)](https://p3.ssl.qhimg.com/t01c0fb99391f6514ec.png)

然后看看这个filterChain是怎么来的

[![](_v_images/20200820092119068_15819.png)](https://p1.ssl.qhimg.com/t01c44bd275690dd198.png)

查看org.apache.catalina.core.ApplicationFilterFactory#createFilterChain源代码

```

...
            filterChain.setServlet(servlet);
            filterChain.setServletSupportsAsync(wrapper.isAsyncSupported());
            StandardContext context = (StandardContext)wrapper.getParent();
            FilterMap[] filterMaps = context.findFilterMaps();
            if (filterMaps != null && filterMaps.length != 0) {
                DispatcherType dispatcher = (DispatcherType)request.getAttribute("org.apache.catalina.core.DISPATCHER_TYPE");
                String requestPath = null;
                Object attribute = request.getAttribute("org.apache.catalina.core.DISPATCHER_REQUEST_PATH");
                if (attribute != null) {
                    requestPath = attribute.toString();
                }

                String servletName = wrapper.getName();

                int i;
                ApplicationFilterConfig filterConfig;
                for(i = 0; i < filterMaps.length; ++i) {
                    if (matchDispatcher(filterMaps[i], dispatcher) && matchFiltersURL(filterMaps[i], requestPath)) {
                        filterConfig = (ApplicationFilterConfig)context.findFilterConfig(filterMaps[i].getFilterName());
                        if (filterConfig != null) {
                            filterChain.addFilter(filterConfig);
                        }
                    }
                }

                for(i = 0; i < filterMaps.length; ++i) {
                    if (matchDispatcher(filterMaps[i], dispatcher) && matchFiltersServlet(filterMaps[i], servletName)) {
                        filterConfig = (ApplicationFilterConfig)context.findFilterConfig(filterMaps[i].getFilterName());
                        if (filterConfig != null) {
                            filterChain.addFilter(filterConfig);
                        }
                    }
                }

                return filterChain;
            } else {
                return filterChain;
            }
        }
...
```

到这里就要掰扯一下这三个的关系：filterConfig、filterMaps跟filterDefs

#### filterConfig、filterMaps、filterDefs

直接查看此时StandardContext的内容，我们会有一个更直观的了解

[![](_v_images/20200820092118754_2708.png)](https://p0.ssl.qhimg.com/t013c18791db3d01984.png)

注入内存马实际上是模拟了在web.xml中写配置的过程，两者是一一对应的。其中filterDefs存放了filter的定义，比如名称跟对应的类，对应web.xml中如下的内容

```
<filter>
    <filter-name>filterDemo</filter-name>
    <filter-class>com.yzddmr6.filterDemo</filter-class>
</filter>
```

filterConfigs除了存放了filterDef还保存了当时的Context，从下面两幅图可以看到两个context是同一个东西

[![](_v_images/20200820092118434_28371.png)](https://p1.ssl.qhimg.com/t01e189f0325a09dd77.png)

[![](_v_images/20200820092118121_10608.png)](https://p1.ssl.qhimg.com/t01baf1c59ee24e5c7e.png)

FilterMaps则对应了web.xml中配置的<filter-mapping>，里面代表了各个filter之间的调用顺序。[![](_v_images/20200820092117809_12656 "")](https://mmbiz.qpic.cn/mmbiz_png/tCS9QJPdcGdicOsegjRicDmQ2DWiaj76IlH6gAZAPZzaDV16Z6oZChibyiabAZ30CtFlQ9kAVK6k0KEn14v1rSZnFEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

[![](_v_images/20200820092117393_15562.png)](https://p1.ssl.qhimg.com/t016db33edecb3d80e9.png)

即对应web.xml中的如下内容

```
<filter-mapping>
    <filter-name>filterDemo</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

都添加完之后， 调用doFilter ，进入过滤阶段。

#### 实现步骤

综上所述，如果要实现filter型内存马要经过如下步骤：

- 创建恶意filter
- 用filterDef对filter进行封装
- 将filterDef添加到filterDefs跟filterConfigs中
- 创建一个新的filterMap将URL跟filter进行绑定，并添加到filterMaps中

要注意的是，因为filter生效会有一个先后顺序，所以一般来讲我们还需要把我们的filter给移动到FilterChain的第一位去。

每次请求createFilterChain都会依据此动态生成一个过滤链，而StandardContext又会一直保留到Tomcat生命周期结束，所以我们的内存马就可以一直驻留下去，直到Tomcat重启。

### Servlet型

#### 注册流程

这次我们换种方式：不进行一步步的调试，直接查看添加一个servlet后StandardContext的变化

```
<servlet>
        <servlet-name>servletDemo</servlet-name>
        <servlet-class>com.yzddmr6.servletDemo</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>servletDemo</servlet-name>
        <url-pattern>/demo</url-pattern>
    </servlet-mapping>
```

可以看到我们的servlet被添加到了children中，对应的是使用StandardWrapper这个类进行封装

[![](_v_images/20200820092117077_7986.png)](https://p1.ssl.qhimg.com/t018dfbb474123c51f0.png)

一个child对应一个封装了Servlet的StandardWrapper对象，其中有servlet的名字跟对应的类。StandardWrapper对应配置文件中的如下节点：

```
<servlet>
        <servlet-name>servletDemo</servlet-name>
        <servlet-class>com.yzddmr6.servletDemo</servlet-class>
    </servlet>
```

类似FilterMaps，servlet也有对应的servletMappings，记录了urlParttern跟所对应的servlet的关系

[![](_v_images/20200820092116662_14122.png)](https://p4.ssl.qhimg.com/t0184f57bef469856ab.png)

servletMappings对应配置文件中的如下节点

```
<servlet-mapping>
        <servlet-name>servletDemo</servlet-name>
        <url-pattern>/demo</url-pattern>
    </servlet-mapping>
```

#### 实现步骤

所以综上所述，Servlet型内存Webshell的主要步骤如下：

- 创建恶意Servlet
- 用Wrapper对其进行封装
- 添加封装后的恶意Wrapper到StandardContext的children当中
- 添加ServletMapping将访问的URL和Servlet进行绑定

### Listener型

目前公开提到的只有Filter Servlet两种内存Webshell，但是实际上通过Listener也可以实现内存马。并且Listener型webshell在三者中的优先级最高，所以危害其实是更大的。

> 关于监听器的详细介绍可以参考这篇文章
> 
> [https://tianweili.github.io/2015/01/27/Java%E4%B8%AD%E7%9A%84Listener-%E7%9B%91%E5%90%AC%E5%99%A8/](https://tianweili.github.io/2015/01/27/Java%E4%B8%AD%E7%9A%84Listener-%E7%9B%91%E5%90%AC%E5%99%A8/)

#### Listener的分类

Listener主要分为以下三个大类：

- ServletContext监听
- Session监听
- Request监听

其中前两种都不适合作为内存Webshell，因为涉及到服务器的启动跟停止，或者是Session的建立跟销毁，目光就聚集到第三种对于请求的监听上面，其中最适合作为Webshell的要数ServletRequestListener，因为我们可以拿到每次请求的的事件：ServletRequestEvent，通过其中的getServletRequest()函数就可以拿到本次请求的request对象，从而加入我们的恶意逻辑 。

#### 实现步骤

在ServletContext中可以看到addListener方法，发现此方法在ApplicationContext实现

javax.servlet.ServletContext#addListener(java.lang.String)

[![](_v_images/20200820092116451_22285.png)](https://p1.ssl.qhimg.com/t017a10bbef022fb7c9.png)

跟进org.apache.catalina.core.ApplicationContext#addListener(java.lang.String)，发现调用了同类中的重载方法

[![](_v_images/20200820092116138_25148.png)](https://p5.ssl.qhimg.com/t01a740d3ed6074f7b2.png)

跟进org.apache.catalina.core.ApplicationContext#addListener(T)，发现遇到了跟添加filter很相似的情况，在开始会先判断Tomcat当前的生命周期是否正确，否则就抛出异常。实际上最核心的代码是调用了 this.context.addApplicationEventListener(t)，所以我们只需要反射调用addApplicationEventListener既可达到我们的目的。

```
public <T extends EventListener> void addListener(T t) {
        if (!this.context.getState().equals(LifecycleState.STARTING_PREP)) {
            throw new IllegalStateException(sm.getString("applicationContext.addListener.ise", new Object[]{this.getContextPath()}));
        } else {
            boolean match = false;
            if (t instanceof ServletContextAttributeListener || t instanceof ServletRequestListener || t instanceof ServletRequestAttributeListener || t instanceof HttpSessionIdListener || t instanceof HttpSessionAttributeListener) {
                this.context.addApplicationEventListener(t);
                match = true;
            }

            if (t instanceof HttpSessionListener || t instanceof ServletContextListener && this.newServletContextListenerAllowed) {
                this.context.addApplicationLifecycleListener(t);
                match = true;
            }

            if (!match) {
                if (t instanceof ServletContextListener) {
                    throw new IllegalArgumentException(sm.getString("applicationContext.addListener.iae.sclNotAllowed", new Object[]{t.getClass().getName()}));
                } else {
                    throw new IllegalArgumentException(sm.getString("applicationContext.addListener.iae.wrongType", new Object[]{t.getClass().getName()}));
                }
            }
        }
    }
```

综上所述，Listener类型Webshell的实现步骤如下：

- 创建恶意Listener
- 将其添加到ApplicationEventListener中去

Listener的添加步骤要比前两种简单得多，优先级也是三者中最高的。

#### 实现效果

首先注入一个恶意的listener事件监听器

[![](_v_images/20200820092115820_31911.png)](https://p1.ssl.qhimg.com/t0123fdb8064b211b0e.png)

访问内存Webshell，一片空白说明注入成功

[![](_v_images/20200820092115609_16523.png)](https://p1.ssl.qhimg.com/t013b7839227dac12ce.png)

在任意路径下加上?mr6=xxx即可执行命令

[![](_v_images/20200820092115398_14094.png)](https://p0.ssl.qhimg.com/t01fd191e331a5b89da.png)

## 降维打击篇

本部分主要分享一些利用JSP特性来对抗语法树类引擎的技巧。

### “非主流”JSP语法

上面提到JSP在第一次运行的时候会先被Web容器，如Tomcat翻译成Java文件，然后才会被Jdk编译成为Class加载到jvm虚拟机中运行。JDK在编译的时候只看java文件的格式是否正确。而Tomcat在翻译JSP的不会检查其是否合乎语法。

所以我们就可以利用这一点，故意构造出不符合语法规范的JSP样本，来对抗检测引擎的AST分析。

[![](_v_images/20200820092115080_29647.png)](https://p0.ssl.qhimg.com/t012bdf92330883086c.png)

[![](_v_images/20200820092114765_18897.png)](https://p3.ssl.qhimg.com/t01f4d7ee6ab7290e56.png)

可以看到编译后的文件刚好把上下文的try catch闭合，形成了合法的Java源文件，所以能够通过JDK的编译正常运行。

### “特殊”内置对象

继续来看翻译后的Java文件，可以看到翻译后的Servlet继承了org.apache.jasper.runtime.HttpJspBase类

[![](_v_images/20200820092114448_14505.png)](https://p5.ssl.qhimg.com/t01f9acefd94c1ffc7f.png)

在\_jspService中有我们写的业务逻辑，在此之前可以看到一系列包括request，response，pageContext等内置对象的赋值操作。其中发现pageContext会赋值给\_jspx\_page\_context，所以就可以直接使用\_jspx\_page_context来代替pageContext，帮助我们获取参数。

[![](_v_images/20200820092114236_22935.png)](https://p2.ssl.qhimg.com/t016c20294b70d9ab91.png)

[![](_v_images/20200820092114023_16838.png)](https://p3.ssl.qhimg.com/t015c7fc3f49e29bc9c.png)

引擎如果没有识别出\_jspx\_page_context就可能当作未定义变量来处理，从而导致污点丢失。

### 利用Unicode编码

JSP可以识别Unicode编码后的代码，这个特性已经被大家所知。如果引擎没有对样本进行Unicode解码处理，就可以直接造成降维打击。

[![](_v_images/20200820092113813_11672.png)](https://p0.ssl.qhimg.com/t0181cdb750a124767b.png)

### 利用HTML实体编码

除了JSP以外，还有一种可以动态解析的脚本类型叫JSPX，可以理解成XML格式的JSP文件。在XML里可以通过实体编码来对特殊字符转义，JSPX同样继承了该特性。我们就可以利用这个特点，来对敏感函数甚至全文进行编码。

[![](_v_images/20200820092113599_30653.png)](https://p2.ssl.qhimg.com/t0111ef535b9e54e86d.png)

[![](_v_images/20200820092113487_22962.png)](https://p5.ssl.qhimg.com/t013ead7d41713059da.png)

### 利用CDATA拆分关键字

XML还有个特性为CDATA区段。

[![](_v_images/20200820092113275_12318.png)](https://p4.ssl.qhimg.com/t017dce4f1c87b5311f.png)

同样可以利用这一点，将关键字进行拆分打乱，以干扰引擎的分析。

[![](_v_images/20200820092113027_16623.png)](https://p5.ssl.qhimg.com/t01d7decbc65868ec45.png)

[![](_v_images/20200820092112710_7575.png)](https://p2.ssl.qhimg.com/t01c92155620574e931.png)

由于安骑士采用的是基于反汇编跟字节码的检测技术，所以并不会被上文中表面形式的混淆所干扰。

## 最后

Java博大精深，深入挖掘还可以发现更多有趣的特性。本文仅为抛砖引玉，如果有不严谨的地方欢迎指正。

关于我们

阿里云安全-能力建设团队以安全技术为本，结合云计算时代的数据与算力优势，建设全球领先的企业安全产品，为阿里集团以及公有云百万用户的基础安全保驾护航。

团队研究方向涵盖WEB安全、二进制安全、企业入侵检测与响应、安全数据分析、威胁情报等。
