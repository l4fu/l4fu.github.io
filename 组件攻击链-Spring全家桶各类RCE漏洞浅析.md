# Spring全家桶简介 
  

Spring发展到现在，全家桶所包含的内容非常庞大，这里主要介绍其中关键的5个部分，分别是spring framework、 springboot、 spring cloud、spring security、spring mvc。其中的spring framework就是大家常常提到的spring， 这是所有spring内容最基本的底层架构，其包含spring mvc、springboot、spring core、IOC和AOP等等。Spring mvc就是spring中的一个MVC框架，主要用来开发web应用和网络接口，但是其使用之前需要配置大量的xml文件，比较繁琐，所以出现springboot，其内置tomcat并且内置默认的XML配置信息，从而方便了用户的使用。下图就直观表现了他们之间的关系。

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333676.jpeg)

而spring security主要是用来做鉴权，保证安全性的。Spring Cloud基于Spring Boot，简化了分布式系统的开发，集成了服务发现、配置管理、消息总线、负载均衡、断路器、数据监控等各种服务治理能力。

整个spring家族有四个重要的基本概念，分别是IOC、Context、Bean和AOP。其中IOC指控制反转，在spring中的体现就是将对象属性的创建权限回收，然后统一配置，实现解耦合，便于代码的维护。在实际使用过程中可以通过autowired注解，不是直接指定某个类，将对象的真实类型放置在XML文件中的bean中声明，具体例子如下：

<bean name="WelcomeService" class="XXX.XXX.XXX.service.impl.WelcomeServiceImpl"/>

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/108.png "108.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/108.png)

Spring将所有创建或者管理的对象称为bean，并放在context上下文中统一管理。至于AOP就是对各个MVC架构的衔接层做统一处理，增强了代码的鲁棒性。下面这张图就形象描述了上述基本概念。

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333677.jpeg)

  

  
  
  

## 各子组件介绍

  

Spring发展至今，整个体系不断壮大，子分类非常庞大，这里只对本次涉及的一些组件做简单的介绍。

首先是Spring Websocket，Spring内置简单消息代理。这个代理处理来自客户端的订阅请求，将它们存储在内存中，并将消息广播到具有匹配目标的连接客户端。Spring Data是一个用于简化数据库访问，并支持云服务的开源框架，其主要目标是使数据库的访问变得方便快捷。Spring Data Commons是Spring Data下所有子项目共享的基础框架，Spring Data家族中的所有实现都是基于Spring Data Commons。简单点说，Spring Data REST把我们需要编写的大量REST模版接口做了自动化实现，并符合HAL的规范。Spring Web Flow是Spring MVC的扩展，它支持开发基于流程的应用程序，可以将流程的定义和实现流程行为的类和视图分离开来。

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336771.jpeg)

  

  
  
  

## 使用量及使用分布

  

根据全网数据统计，使用Spring的网站多达80万余，其中大部分集中在美国，中国的使用量排在第二位。其中香港、北京、上海、广东四省市使用量最高。通过网络空间搜索引擎的数据统计和柱状图表，如下图所示。

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333678.jpeg)

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336781.jpeg)

  

  
  
  

## 漏洞背景介绍（SpEL使用）

  

### 0x10 SpEL是什么  


SpEL是基于spring的一个表达式语言，类似于struts的OGNL，能够在运行时动态执行一些运算甚至一些指令，类似于Java的反射功能。就使用方法上来看，一共分为三类，分别是直接在注解中使用，在XML文件中使用和直接在代码块中使用。

  

### 0x20 SpEL能做什么

● 基本表达式

包括逻辑运算，三目运算和正则表达式等等。

● 类操作表达式

对象方法调用，对象属性引用，自定义函数和类实例化等等。

● 集合操作表达式

字典的访问，投影和修改等等。

● 其他表达式

模板表达式

  

### 0x30 SpEL demo

### 0x31 基于注解的SpEL

可以结合sping的@Value注解来使用，可以直接初始化Bean的属性值

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/109.png "109.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/109.png)

  

在这种情况下可以直接将test的值初始化为## AAA## 。

此外，还有很多其他注解的使用方式，可以结合上面提到的表达式的四种使用模式。

  

### 0x32 基于XML的SpEL

可以直接在XML文件中使用SpEL表达式如下：

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns=" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       
       xsi:schemaLocation="http://www.springframework.org/schema/beans       
                        http://www.springframework.org/schema/beans/spring-beans.xsd">    
    <bean id="world" class="java.lang.String">      
      <constructor-arg value="#{' World!'}"/>    
    </bean>    
    <bean id="hello" class="java.lang.String">     
       <constructor-arg value="#{'Hello'}#{world}"/>    
    </bean>
</beans>

  

public class SpEL {
    public static void main(String\[\] args){  
          ApplicationContext ctx = new ClassPathXmlApplicationContext("test.xml");        
          String hello = ctx.getBean("hello", String.class);        
          System.out.println(hello);   
    }
}

  

上面的代码将会输出## Hello World!## ，可以看到递归往下找到world的值，最终成功返回。

  

### 0x33 字符串操作

```
import org.springframework.expression.Expression;
```

注：类似的字符串操作比如toUpperCase()，substr()等等

  

### 0x34 类相关操作

使用T(class)来表示类的实例，除了java.lang的包，剩下的包需要指明。此外还可以访问类的静态方法和静态字段，甚至实例化类。

public class SpEL {
    public static void main(String\[\] args){     
       ExpressionParser parser = new SpelExpressionParser();        
       Expression exp = parser.parseExpression("T(Runtime).getRuntime().exec('calc.exe')");        
       Object message = exp.getValue();        
       System.out.println(message);    
   }
}

  

如上述操作，最终就可以执行命令，弹出计算器。这也是后面SpEL RCE漏洞的利用形式。

  

### 0x35 集合相关操作

public class SpEL {
    public static void main(String\[\] args){     
       ExpressionParser parser = new SpelExpressionParser();        
       Expression exp = parser.parseExpression("{'sangfor', 'busyer', 'test'}");        
       List<String> message = (List<String>) exp.getValue();        
       System.out.println(message.get(1));  //busyer    
    }
}

  

通过上面的操作，可以将字符串转化成数组，最终可以输出busyer。

  

### 0x36 SpEL原理

#### SpEL原理

首先来了解几个概念：

● 表达式

可以认为就是传入的字符串内容

● 解析器

将字符串解析为表达式内容

● 上下文

表达式对象执行的环境

● 根对象和活动上下文对象

根对象是默认的活动上下文对象，活动上下文对象表示了当前表达式操作的对象

具体的流程如下，其实就是编译原理里面的词法分析和句法分析：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336782.jpeg)

（1）首先给定表达式1+2

（2）然后给定SpelExpressionParser解析器，该解析器就实现了上图中的分析

（3）定义上下文对象，这个是可选的，默认是StandardEvaluationContext

（4）使用表达式对象求值，例如getValue

具体代码如下：

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("{'sangfor', 'busyer', 'test'}");
//StandardEvaluationContext context = new StandardEvaluationContext();String message = (String)exp.getValue(context, String.class);

  

#### root和this

SpEL中#root总是指的刚开始的表达式对象，而#this总是指的当前的表达式对象，用他们可以直接操作当前上下文。

#### SimpleEvaluationContext和StandardEvaluationContext

SimpleEvaluationContext: 不包含类相关的危险操作，比较安全

StandardEvaluationContext: 包含所有功能，存在风险

  

  
  
  

## 高危漏洞介绍

  

通过对Spring漏洞的收集和整理，过滤出其中影响较大的远程代码执行高危漏洞，可以得出如下列表：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333679.png)

从上表可以看出，这些漏洞分布在Spring不同的子分类之间，且大多都是较低的版本，用户只要及时升级高版本并及时关注新的漏洞信息即可轻松规避这些漏洞。尽管近期没有出现相关漏洞，但是这些高风险漏洞依然不可忽视。这里面出现的漏洞大多不需要复杂的配置就可以直接攻击成功，从而执行任意代码，危害较大。所以， 开发者在使用Spring进行开发的过程中，一定要关注其历史风险点，尽量规避高危漏洞，减少修改不必要的配置信息。 

  

  
  
  

## 漏洞利用链

  

上述漏洞基本不依赖其他Spring漏洞即可直接获取权限，下图对其利用方式做了简要概述：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333681.png)

  

  
  
  

## 高可利用漏洞分析

  

### 1 CVE-2018-1270

### 1.1 威胁等级

严重

  

### 1.2 影响范围

Spring Framework 5.0 - 5.0.5

Spring Framework 4.3 - 4.3.15

  

### 1.3 利用难度

简单

  

### 1.4 漏洞描述

在上面描述的存在漏洞的Spring Framework版本中，允许应用程序通过spring-messaging模块内存中STOMP代理创建WebSocket。攻击者可以向代理发送消息，从而导致远程执行代码攻击。

  

### 1.5 漏洞分析

点击connect，首先将触发DefaultSubscriptionRegistry.java中的addSubscriptionInternal方法，

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/110.png "110.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/110.png)

第80行将首部的selector字段的值取出，就是我们之前传入的恶意表达式，接着到83行，这一步就很熟悉了，使用解析器去解析表达式，显然这个时候再有一个getValue方法触发并且没有使用simpleEvaluationContext就能够直接执行我们传入的表达式了。

监听网络流量，发现后面send信息的时候，将会将消息分发给不同的订阅者，并且转发的消息还会包含之前connect的上下文，即这里的expression将会包含在内。

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/111.png "111.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/111.png)

  

于是，尝试随便在文本框中输入一些内容，然后点击Send，最终可以触发SimpleBrokerMessageHandler.java中的sendMessageToSubscribers方法如下：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/112.png "112.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/112.png)

继续进入findSubscriptions方法，并且不断往下走，最终可以发现在DefaultSubscriptionRegistry.java中filterSubscriptions方法中对上下文中的expresion做了提取，并使用StandardEvaluationContext指定了上下文，也就是说这里面可以直接执行代码，没有任何限制。并最终在第164行使用getValue方法触发漏洞，弹出计算器。

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/113.png "113.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/113.png)

### 1.6 补丁分析##   

补丁中直接将上面的StandardEvaluationContext替换成SimpleEvaluationContext，使用该方法能够避免了恶意类的加载。

  

### 2##  ## CVE-2018-1273

### 2.1 威胁等级

严重

  

### 2.2 影响范围##   

Spring Data Commons 1.13 - 1.13.10 (Ingalls SR10)

Spring Data REST 2.6 - 2.6.10 (Ingalls SR10)

Spring Data Commons 2.0 to 2.0.5 (Kay SR5)

Spring Data REST 3.0 - 3.0.5 (Kay SR5)

  

### 2.3 利用难度

简单

  

### 2.4 漏洞描述

Spring Data Commons组件中存在远程代码执行漏洞，攻击者可构造包含有恶意代码的SPEL表达式实现远程代码攻击，直接获取服务器控制权限。

  

### 2.5 漏洞分析

从上述/users入口，最终会调用到MapPropertyAccessor静态类中对用户名进行处理。而在该类中包含了进行SpEL注入需要满足的条件如下：

● 首先创建解析器：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/114.png "114.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/114.png)

● 接着使用Standard上下文

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/115.png "115.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/115.png)

● 然后包含待解析表达式

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/116.png "116.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/116.png)

● 最后使用setValue触发

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/117.png "117.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/117.png)

  

### 2.6 补丁分析

补丁依旧直接将上面的StandardEvaluationContext替换成SimpleEvaluationContext，使用该方法能够避免了恶意类的加载。

  

### 3##  ## CNVD-2016-04742

### 3.1 威胁等级

严重

  

### 3.2 影响范围

Springboot 1.1.0-1.1.12

Springboot 1.2.0-1.2.7

Springboot 1.3.0

  

### 3.3 利用难度

简单  

  

### 3.4 漏洞描述

低版本的springboot在处理内部500错误时，使用了spel表达式，并且递归向下解析嵌套的，其中message参数是从外部传过来的，用户就可以构造一个spel表达式，达到远程代码执行的效果。

  

### 3.5 漏洞分析

访问上面的URL，可以进入到我们的控制器，并紧接着抛出异常如下：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/118.png "118.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/118.png)

进入异常的代码，经过冗长的代码调试，最终可以来到关键点的render方法：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/119.png "119.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/119.png)

接着进入render方法查看，这里面的replacePlaceholders方法将会进行形如${}的spel表达式替换：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/120.png "120.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/120.png)

进入该方法查看，最后进入parseStringValue方法,该方法会循环将带有${}的错误页面的HTML字符串中的一个个${}的内容进行替换，并且这里面的${message}是我们传入的值。

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/121.png "121.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/121.png)

于是可以就此构造我们的payload，借助他的循环，继续解析spel，最终造成任意代码执行。其中，解析spel的代码如下：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/122.png "122.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/122.png)

  

### 3.6 补丁分析

通过添加一个NonRecursivePropertyPlaceholderHelper类，对于二次解析的值进行限制：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333688.jpeg)

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336881.jpeg)

  

### 4 CVE-2017-8046

### 4.1 威胁等级

严重

  

### 4.2 影响范围

Spring Data REST prior to 3.0.1 and Spring Boot versions prior to 1.5.9

Spring Data REST prior to 2.6.9 Spring Boot versions prior to 1.5.9

  

### 4.3 利用难度

简单

  

### 4.4 漏洞描述

用户在使用PATCH方法局部更新某个值的时候，其中的path参数会被传入SpEL表达式，进而导致代码执行。

  

### 4.5 漏洞分析

执行上述payload，定位到程序的入口如下：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336882.jpeg)

（注：这个类在springmvc里面，名字为JsonPatchHandler）

重点看这个三目运算，其中的判断是看HTTP方法是否为PATCH和content-type是否为我们上面提到的那个，然后会进入this.applyPatch方法，接着根据我们指定的replace字段进入对应的处理器：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333689.jpeg)

然后实例化patchOperation，并初始化spel解析器：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336891.jpeg)

最后再调用setValue触发：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336892.jpeg)

  

### 4.6 补丁分析

这里用2.6.9中的修复方案举例子，在perform中不是直接setvalue，而是先做一个参数合法性校验（此处添加了SpelPath类），将path中的参数用'.'分割，然后依次判断是否是类的属性，只要有一个不是就直接报错，从而解决了上述问题，部分补丁图片如下：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333690.jpeg)

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336901.jpeg)

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336902.jpeg)

  

### 5 CVE-2017-4971

### 5.1 威胁等级

中危

  

### 5.2 影响范围

Spring Web Flow 2.4.0 ~ 2.4.4

Spring Web Flow 2.4.4 ~ 2.4.8

  

### 5.3 利用难度

较高

  

### 5.4 漏洞描述

当用户使用Spring Web Flow受影响的版本时，如果配置了view-state，但是没有配置相应的binder,并且没有更改useSpringBeanBinding默认的false值，当攻击者构造特殊的http请求时，就可以导致SpEL表达式注入，从而造成远程代码执行漏洞。

  

### 5.5 漏洞分析

首先通过执行confirm请求，断点到如下位置：

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/123.png "123.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/123.png)

这里可以发现可以通过判断binderConfiguration是否为空来选择进入哪个处理方法，这里的binderConfiguration值指的是在配置文件中配置的binder内容。深入查看这两个处理方法。其实都用了SpEL表达式，不过addModelBindings方法传入的参数的是上面提到的binder，是写死在xml文件中的，无法去更改，所以这里面就考虑当没配置binder的情况下走进addDefaultMapping方法的情况。

[![.png](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/124.png "124.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/12/124.png)

addDefaultMappings方法如上，其作用是遍历所有的参数，包括GET参数和POST中的参数，然后一个个判断其是否以"\_"开头，如果符合就进入addEmptyValueMapping方法进行处理，否则就进入addDefaultMapping方法进行处理。本次漏洞的触发点是上面这一个，所以我们深入查看一下addEmptyValueMapping方法。

  

可以看到该方法用SpEL表达式解析了传入的变量名，并在后面使用了get操作，从而可以导致漏洞的产生。

  

### 5.6 补丁分析

查看官方补丁源码如下：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333692.jpeg)

将表达式类型换成了BeanWrapperExpressionParser，因为该类型内部实现不能够处理类所以避免了该问题的发生。

然而上述还提到如果参数类型不是以"\_"开头的将会进入addDefaultMapping方法，下面我们进入该方法进行查看：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333693.jpeg)

可以看到这里也对传入的参数进行了解析但是没有看到明显的get方法来触发，继续往下寻找get方法。首先这里面将解析器放入了mapper中，下面就重点追踪这个mapper的使用即可。

首先发现一步步回到之前的bind方法，可以发现最后一行对该mapper进行了操作，跟进该map方法:

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336931.jpeg)

在这里就进行了get操作，从而再次触发了漏洞。

对此，也可能跟这个没关系，官方最终将全局的解析器换成SimpleEvaluationContext来彻底解决此问题。

  

### 6 CNVD-2019-11630

### 6.1 威胁等级

严重

  

### 6.2 影响范围

Spring Boot 1-1.4

Spring Boot 2.x

  

### 6.3 利用难度

简单

  

### 6.4 漏洞描述

用户在通过env路径修改spring.cloud.bootstrap.location的位置，将该地址设置为一个恶意地址时，并在后面使用refresh接口进行触发就可以导致靶机加载恶意地址中的文件，远程执行任意代码。

  

### 6.5 漏洞分析

搭建环境并按上述方式进行攻击，并搜索到spring-cloud-context-1.2.0.RELEASE.jar中的environment和refresh，然后下断点跟进，可以发现首先的env改变会将下面体现：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336932.jpeg)

其实就是将环境中该变量的属性值进行更新。

之后看一下关键点refresh接口，首先一旦refresh接口被触发，就会将有变化的信息以及一些基本信息挑选出来，如下图可以看到之前变化的值已经被挑选出来：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333694.jpeg)

接着进入到addConfigFilesToEnvironment方法进行处理，先获取到所有的环境值，然后设置一个监听器，依次处理变化的信息：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336941.jpeg)

这里我们直接跳转到处理这个恶意地址的关键部分，首先进入ConfigFileApplicationListener的load方法：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336942.jpeg)

这里面先判断url是否存在文件路径，如果存在才进入处理该地址，否则将name的参数设置成searchName进行处理，这里的值为“bootstrap”，后面会强行加上后缀。然后一直深入到PropertySourcesLoader类中的load方法：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-1607333695.jpeg)

首先会发送一个head请求判断文件是否存在，以及是否是一个文件，然后会根据文件后缀来判断是否能解析，这里面就是yml文件，所以判断可以用YamlPropertySourceLoader类来处理。然后进入该类的load方法中：

![](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/1970/01/beepress--148624-16073336951.jpeg)

在这里将会加载远程yml文件，并处理里面的内容，而导致远程代码执行的发生。

  

### 6.6 补丁分析

在springboot 1.5及以后，官方对这些接口添加了授权验证，不能够再肆意的调用他们了。