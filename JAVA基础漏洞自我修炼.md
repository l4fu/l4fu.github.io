# JAVA基础漏洞自我修炼
## 练气期-反射篇

  

众所周知，气是修炼的基础即反射是java的其中的一个高级特性。正是因为反射的特性引出了后续的动态代理，AOP，RMI，EJB等功能及技术，在后续再来说下代理，RMI等及其漏洞原理吧，在之前先来看看反射所有的原理及漏洞，那么，在修炼初期应该注意什么问题呢？

  

俗话说万事开头难--什么是反射

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

在日常开发中，经常会遇到访问装载在JVM中类的信息，包括构造方法，成员变量，方法，或者访问一个私有变量，方法。

  

###  修炼进行时--反射方法

反射方法很多只列举部分重要的来说。

#### 获取class的字节码对象

前面说到反射是对运行中的类进行查询和调用，所以首先我们需要获取运行类的对象，即字节码对象（可以看看JVM加载原理）。方式有三种来看看。

方式一：

Class.forName("类的字符串名称");

方式二：

简单类名加.class来获取其对应的Class对象；

方式三：

Object类中的getClass()方法的。

三种区别主要是调用者不同，以及静态和动态区别（java是依需求加载，对于暂时不用的可以不加载）。

#### 获取构造函数

getConstructors()//获取所有公开的构造函数

getConstructor(参数类型)//获取单个公开的构造函数

getDeclaredConstructors()//获取所有构造函数

getDeclaredConstructor(参数类型)//获取一个所有的构造函数

#### 获取名字

可以反射类名。

getName()//获取全名  例如：com.test.Demo

getSimpleName()//获取类名 例如：Demo

#### 获取方法

getMethods()//获取所有公开的方法

#### 获取字段

getFields()//获取所有的公开字段

getField(String name)//参数可以指定一个public字段

getDeclaredFields()//获取所有的字段

getDeclaredField(String name)//获取指定所有类型的字段

#### 设置访问属性

默认为false，设置为true之后可以访问私有字段。

Field.setAccessible(true)//可访问

Field.setAccessible(false)//不可访问

以及Method类的invoke方法

invoke(Object obj, Object... args) //传递object对象及参数调用该对象对应的方法

  

### 打怪修炼—实战示例

来看一个简单的反射案例，可以执行运行计算器命令。  

![](_v_images/20200820082448320_31390.png)

通过Class.forName获取字节码对象，调用getMethod获取到Runtime的getRuntime方法，用invoke执行方法，最后同样的执行exec方法执行calc命令。

说到这，大家都熟悉，那么具体的反射漏洞有哪些，我们来看看。

  

### 反射攻击

#### 通过反射来突破单例模式

我们知道单例模式的特点就是单例类只能有一个实例，但是不好的代码就可以突破单例限制，比如：

![](_v_images/20200820082448013_31768.png)

![](_v_images/20200820082447704_3604.png)

运行结果：

![](_v_images/20200820082447297_16258.png)

私有的构造方法，类变量，可以看出代码实现了单例的要求，new的时候没有创建对象，就新建，有的话就返回这个对象，但是通过反射（反序列化也可以突破，这里只说反射）可以直接调用private方法创建实例。

![](_v_images/20200820082446889_5191.png)

运行结果：

![](_v_images/20200820082446380_18251.png)

所以我们要在构造方法的时候就要判断是不是已经创建过对象，如果有就主动抛出异常。  

![](_v_images/20200820082446071_14044.png)

#### 突破瓶颈---通过反射来突破泛型限制

我们知道泛型的特点就是明确规范参数使用的类型，但是不好的代码就可以突破单例限制。

![](_v_images/20200820082445563_8073.png)

就会抛出异常。

![](_v_images/20200820082445055_31393.png)

同样的我们可以通过反射：

![](_v_images/20200820082444347_18780.png)

结果如下。

![](_v_images/20200820082443838_1673.png)

这种我们就需要添加黑名单来禁止反射，当然也可以绕过。

#### 利用反射链的序列化漏洞

以前我们经常能看见这种构造的序列化漏洞的文章。

先来看看部分实现代码：

```
Transformer[] transformers = new Transformer[] {
```

下面是InvokerTransformer类的transform方法的源码。

![](_v_images/20200820082443329_30400.png)

战后总结分析：

  

```
Object[] argss=new Object[]{"getRuntime",null};
```

相当于执行了：

  

Method mm=Runtime.class.getMethod("getRuntime", null);

\-\-\-  

  

Runtime rr=(Runtime) mm.getClass().getMethod("invoke", new Class\[\] {Object.class,Object\[\].class}).invoke(mm,new Object\[\] {null,null} );

相当于执行了：

mm.invoke();

\-\-\-
rr.getClass().getMethod("exec", new Class\[\] {String.class}).invoke(rr, "calc");

相当于执行了rr.exec("calc"); //rr已经是Runtime对象了，而不是Runtime类。

ConstantTransformer在初始化的时候放入里面的一个final变量中，transform(任意Object)都会返回那个变量。

利用jd-gui来看一下ChainedTransformer的源码。

[![.png](_v_images/20200820082442802_11969.png "294.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/294.png)

[![.png](_v_images/20200820082442492_4574.png "295.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/295.png)

那么就可以利用这一点，进行反射，反射代码如下：

  

```
Transformer[] transformer=new Transformer[]{
```

事实上，前面说了ConstantTransformer的特点，所有最后执行的Object.class可以为任何Object，比如null，new Object()。

这里进行调用了transform方法，如何才能不通过调用transform方法执行反射链呢？

我们就要找到实现本身实现tranform的方法。

经过查找发现：

AbstractInputCheckedMapDecorator类下：

[![.png](_v_images/20200820082442079_13910.png "296.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/296.png)

TransformedMap类下：

[![.png](_v_images/20200820082441570_8769.png "297.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/297.png)

[![.png](_v_images/20200820082441261_15367.png "298.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/298.png)

所以我们要控制valueTransformer的值为ChainTransformer对象，找到这个值的赋值点。  

[![.png](_v_images/20200820082440953_30559.png "299-1024x115.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/299.png)

[![.png](_v_images/20200820082440543_29489.png "300-1024x172.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/300.png)

所以我们要实现这个链环，就要满足基本条件，先

  

```
Map mp=new HashMap();
```

  

## 金丹期--反序列化篇

Java 的序列化是把 Java 的对象转换为jvm可以识别的字节序列的过程，方便于存在文件，jvm内存，网络的传输等。

常见的ObjectOutputStream类的 writeObject() 方法可以实现序列化的功能。而反序列化是指把字节序列重新恢复成 Java 对象，反序列化用ObjectInputStream 类的 readObject() 方法。

  

知己知彼之什么是序列化，反序列化

  

[![.png](_v_images/20200820082440131_14150.png "301.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/301.png)

[![.png](_v_images/20200820082439822_12288.png "302.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/302.png)

结果如下:

[![.png](_v_images/20200820082439310_22910.png "303.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/303.png)

这就是序列化和反序列化的过程。

  

### 金丹实战--反序列化漏洞示例

  

[![.png](_v_images/20200820082438986_32321.png "304.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/304.png)

实战结果

[![.png](_v_images/20200820082438374_345.png "305.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/305.png)

很显然在实现自己的readObject方法，反序列化后readObject正好被利用，触发恶意代码。反序列化利用的方式很多。

  
## 元婴期实战演练
### JNDI注入

已有多位前辈修炼至此境界，吾将在此吸取前人经验，不便在此过多停留。

JNDI漏洞原理：在lookup参数可控的情况下，我们传入Reference类型及其子类的对象，当远程调用类的时候默认首先会在rmi的服务器中的classpath中去查找，如果不存在对应的class，就会去提供的url地址去加载类。如果都加载不到的话就会失败。


JNDI这里我们先搭建一个Registry

Server:

[![.png](_v_images/20200820082437966_1164.png "306-1024x182.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/306.png)

Reference中写好自己的要执行payload的class对象名称，以及对应开启的web服务，然后绑定在registry中。

ExecTest:

[![.png](_v_images/20200820082437354_26690.png "307.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/307.png)

这里写入自己payload，我用的静态块，方便执行。

[![.png](_v_images/20200820082436824_27048.png "308.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/308.png)

用javac -source 1.5 -target 1.5 ExecTest.java编译成1.5jdk版本支持的ExecTest.class字节码文件，有一些警告信息提示1.5版本在未来版本被移除，忽略掉。

为了保证是真的成功，要把对应下的bin/ExecTest.class文件给删除掉，前面说了，JNDI会先加载本地的class文件，所以需要先删除对应的class文件，确保是真的远程加载。

我这里把编译的文件放入D盘，开启Web服务。

Client：

[![.png](_v_images/20200820082436214_26576.png "309.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/309.png)

启动好Server，运行Client，可以看见如下：

[![.png](_v_images/20200820082435906_24643.png "310.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/310.png)

远程加载ExecTest.class文；

[![.png](_v_images/20200820082435596_1193.png "311-1024x513.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/311.png)

成功执行命令。

但是看见要求必须是1.6以下的版本，后面的版本都对其进行限制，有限制就有绕过，对应的，默认不允许从远程的Codebase加载Reference工厂类，就可以添加如下代码，将

com.sun.jndi.rmi.object.trustURLCodebase;com.sun.jndi.cosnaming.object.trustURLCodebase两个属性值设置为true。

[![.png](_v_images/20200820082434771_30089.png "312.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/08/312.png)

还有LDAP + JNDI请求LDAP地址来突破限制，利用LDAP反序列化执行本地Gadget来绕过等。

金丹期修炼时--序列化这里，java以rmi（java以rpc为基础的java技术）为根基来衍生更多，比如熟悉的EJB，为了使用其他语言，使用Web服务；实现与平台无关，又使用了SOAP协议。而Weblogic在RMI上的实现使用了T3协议等等。所以了解RMI，了解java基础漏洞的自我修炼，只有知己知彼，才能百战百胜。

修炼永无止尽，万物皆是如此，需屏气凝神方能比其更为强大，以至于交手时不落于下风。

本文作者：[酒仙桥六号部队](https://www.secpulse.com/archives/newpage/author?author_id=33870)

本文为安全脉搏专栏作者发布，转载请注明：[https://www.secpulse.com/archives/138080.html](https://www.secpulse.com/archives/138080.html)

Tags: [AOP](https://www.secpulse.com/archives/tag/aop)、[EJB](https://www.secpulse.com/archives/tag/ejb)、[InvokerTransformer类](https://www.secpulse.com/archives/tag/invokertransformer%e7%b1%bb)、[java](https://www.secpulse.com/archives/tag/java)、[JAVA反射机制](https://www.secpulse.com/archives/tag/java%e5%8f%8d%e5%b0%84%e6%9c%ba%e5%88%b6)、[JNDI注入](https://www.secpulse.com/archives/tag/jndi%e6%b3%a8%e5%85%a5)、[ReadObject](https://www.secpulse.com/archives/tag/readobject)、[Registry](https://www.secpulse.com/archives/tag/registry)、[RMI](https://www.secpulse.com/archives/tag/rmi)、[transform方法](https://www.secpulse.com/archives/tag/transform%e6%96%b9%e6%b3%95)、[writeObject()](https://www.secpulse.com/archives/tag/writeobject)、[反射](https://www.secpulse.com/archives/tag/%e5%8f%8d%e5%b0%84)、[反射攻击](https://www.secpulse.com/archives/tag/%e5%8f%8d%e5%b0%84%e6%94%bb%e5%87%bb)、[反序列化](https://www.secpulse.com/archives/tag/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96)、[序列化漏洞](https://www.secpulse.com/archives/tag/%e5%ba%8f%e5%88%97%e5%8c%96%e6%bc%8f%e6%b4%9e)、[泛型](https://www.secpulse.com/archives/tag/%e6%b3%9b%e5%9e%8b)
