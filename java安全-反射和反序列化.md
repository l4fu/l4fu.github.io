# java安全-反射和反序列化

Java安全漫谈笔记
https://www.anquanke.com/post/id/241089#h2-6
## 反射 1

java反射的函数

```
public void execute(String className, String methodName) throws Exception {
     Class clazz = Class.forName(className);//获取一个类
     clazz.getMethod(methodName).invoke(clazz.newInstance());//
}

```

- 获取类的⽅法： forName
- 实例化类对象的⽅法： newInstance
- 获取函数的⽅法： getMethod
- 执⾏函数的⽅法： invoke

forName有两个函数重载：

- Class forName(String name)
- Class forName(String name, **boolean** initialize, ClassLoader loader)

```
Class.forName(className)
// 等于
Class.forName(className, true, currentLoader)

```

`这里第二函数中的第二个参数表示是否初始化，而这里的是否初始化是告诉Java虚拟机是否执⾏”类初始化“。`其中， static {} 就是在”类初始化”的时候调⽤的，⽽ {} 中的代码会放在构造函数的 super() 后⾯，但在当前构造函数内容的前⾯。所以说我们如果控制一个恶意类，使用forName(“恶意类”)去获得就可能执行恶意代码。

red.java

```
package ref;
public class ref {
    public static void main(String[] args) throws Exception {
        String name = "ref.exp";
        Class.forName(name);//获得class
    }
}

```

exp.java

```
package ref;

public class exp {
    static {
        try {
//            Runtime.getRuntime().exec("calc");
        Runtime rt =Runtime.getRuntime();
        String[] commands = {"calc"};
        Process pc = rt.exec(commands);
        pc.waitFor();//对象的进程结束才返回调用
        }catch (Exception e){
         //do nothing
        }
    }
}

```

[![](_v_images/20210527141022486_17318.png)](https://p0.ssl.qhimg.com/t01aee34cba5a0195c5.png)

## 反射 2

class.newInstance() 的作用就是**调用这个类的无参构造函数，这个比较好理解**(如果类构造函数是私有的就利用不成功)

[![](_v_images/20210527141022246_18276.png)](https://p4.ssl.qhimg.com/t01fd47246a00e96dd6.png)

原因是 Runtime 类的构造方法是私有的。而这种模式是**单例模式**。

[![](_v_images/20210527141022010_25339.png)](https://p2.ssl.qhimg.com/t0161f4838cca0c01b9.png)

Runtime类就是单例模式，我们只能通过 Runtime.getRuntime() 来获取到 Runtime 对

象。所以payload为

```
Class clazz = Class.forName("java.lang.Runtime");
clazz.getMethod("exec", String.class).invoke(clazz.getMethod("getRuntime").invoke(clazz), "calc.exe");

```

这里用到了 getMethod 和 invoke 方法。

**getMethod** 的作用是通过反射获取一个类的某个特定的公有方法。而后面的**String.class**表示该方法接受的参数，为什么要写？是因为java中存在重载(既相同的方法可能会接受不同的参数)

[![](_v_images/20210527141021768_11946.png)](https://p4.ssl.qhimg.com/t01119643eff42e3181.png)

**invoke** 的作用是执行方法，它的第一个参数是：

如果这个方法是一个普通方法，那么第一个参数是类对象

如果这个方法是一个静态方法，那么第一个参数是类

所以正常执行方法是 \[1\].method(\[2\], \[3\], \[4\]…) ，其实在反射里就是method.invoke(\[1\], \[2\], \[3\], \[4\]…) 。

**这里的方法就是exec()是一个普通方法**

[![](_v_images/20210527141021533_26460.png)](https://p3.ssl.qhimg.com/t0102eb85c4d58fd117.png)

所以invoke的第一个参数就是类对象

[![](_v_images/20210527141021304_7037.png)](https://p0.ssl.qhimg.com/t013f00ad535b18981c.png)

Payload分解一下就是:

```
Class clazz = Class.forName("java.lang.Runtime");//获得类
Method Method = clazz.getMethod("exec", String.class);//获得exec方法
Method getRuntime = clazz.getMethod("getRuntime");//获得getRuntime方法
Object runtime = getRuntime.invoke(clazz);//实例化getRuntime方法
Method.invoke(runtime,"calc");//反射执行

```

## 反射 3

如果一个利用类，是有参构造方法，并且没有单例模式里的静态方法。那怎么办？？

如：**ProcessBuilder**

[![](_v_images/20210527141021070_31875.png)](https://p1.ssl.qhimg.com/t0117641597f020290f.png)

这里我们就需要使用另一个反射方法 **getConstructor**

getConstructor接收的参数是**构造函数**列表类型，因为构造函数也支持重载，所以必须用参数列表类型才能唯一确定一个构造函数。(也就是需要指明参数类型)，获取到构造函数后，我们使用newInstance来执行。(实例化类对象的⽅法)

这里我们使用反射来获取其构造函数，然后调用start() 来执行命令：

```
Class clazz = Class.forName("java.lang.ProcessBuilder");
((ProcessBuilder) clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe"))).start();

```

[![](_v_images/20210527141020918_15048.png)](https://p4.ssl.qhimg.com/t01c67db3c443629f0b.png)

其实我们可以清楚的看到利用构造函数然后实例化了这个ProcessBuilder类

[![](_v_images/20210527141020681_6785.png)](https://p1.ssl.qhimg.com/t01c3802ae7fcb5d99b.png)

这里虽然可以执行成功，不过利用漏洞写exp的时候没有进行强行转换**ProcessBuilder**

所以我们应该使用反射来执行命令。

```
Class clazz = Class.forName("java.lang.ProcessBuilder");
clazz.getMethod("start").invoke(clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe")));
//方法.invoke(类的实例化)

```

通过 getMethod(“start”) 获取到start方法，然后 invoke 执行， invoke 的第一个参数就是

ProcessBuilder Object了。

在上面我们使用的构造函数参数类型是**List**，但是ProcessBuilder类的构造方法支持接收俩个参数类型。见最上面的图片。**另一个是String…**。而这个类型其实是数组。

> 这又涉及到Java里的可变长参数（varargs）了。正如其他语言一样，Java也支持可变长参数，就是当你定义函数的时候不确定参数数量的时候，可以使用 … 这样的语法来表示“这个函数的参数个数是可变的”
> 
> 对于可变长参数，Java其实在编译的时候会编译成一个数组，也就是说，如下这两种写法在底层是等价的（也就不能重载）：
> 
> public void hello(String\[\] names) {}
> 
> public void hello(String…names) {}

所以对于反射来说，如果要获取的目标函数里包含可变长参数，其实我们认为它是数组就行了。还有一个小问题：在调用 newInstance 的时候，因为这个函数本身接收的是一个可变参数，我们传给ProcessBuilder的也是一个可变长参数，二者叠加为一个二维数组，所以整个Payload如下

```
Class clazz = Class.forName("java.lang.ProcessBuilder");
((ProcessBuilder)clazz.getConstructor(String[].class).newInstance(new
String[][]{{"calc.exe"}})).start();

```

不过我们这里还是需要将其转换成反射类型的payload。

```
Class clazz = Class.forName("java.lang.ProcessBuilder");//反射
clazz.getMethod("start").invoke(clazz.getConstructor(String[].class).newInstance(new String[][]{{"calc"}}));

```

[![](_v_images/20210527141020543_16374.png)](https://p1.ssl.qhimg.com/t01fda26e1b72ea4c02.png)

还记得前面我们利用的Runtime类？我们是使用的**单例模式**(用 Runtime.getRuntime()来

获取对象)来生成的payload，而避免了私有的构造函数利用。

就涉及到 getDeclared 系列的反射了，与普通的 getMethod 、 getConstructor 区别是：

- getMethod 系列方法获取的是当前类中所有公共方法，包括从父类继承的方法
- getDeclaredMethod 系列方法获取的是当前类中“声明”的方法，是实在写在这个类里的，包括私有的方法，但从父类里继承来的就不包含了

getDeclaredMethod 的具体用法和 getMethod 类似， getDeclaredConstructor 的具体用法和getConstructor 类似。

这样我们就可以利用**Runtime**这个类的构造函数(虽然是私有的)来实例化对象，进而执行命令：

```
Class clazz = Class.forName("java.lang.Runtime");
Constructor m = clazz.getDeclaredConstructor();//获得构造函数
m.setAccessible(true);//设置构造函数为可访问
clazz.getMethod("exec", String.class).invoke(m.newInstance(), "calc.exe");

```

可见，这里使用了一个方法 setAccessible ，这个是必须的。我们在获取到一个私有方法后，必须用setAccessible 修改它的作用域，否则仍然不能调用。

## RMI 1

RMI全称是Remote Method Invocation，远程⽅法调⽤。从这个名字就可以看出，他的⽬标和RPC其实是类似的，是让某个Java虚拟机上的对象调⽤另⼀个Java虚拟机中对象上的⽅法，只不过RMI是Java独有的⼀种机制。`RMI`用于构建分布式应用程序，`RMI`实现了`Java`程序之间跨`JVM`的远程通信。

下面我们实验一下

**RMIServer.java**

```
package rmi;

import java.rmi.Naming;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.server.UnicastRemoteObject;

public class RMIServer {
    public interface IRemoteHelloWorld extends Remote{
        // 继承了 java.rmi.Remote 的接⼝
        public String hello() throws RemoteException;
        //中定义我们要远程调⽤的函数，⽐如这⾥的 hello()
//        public String exec() throws  RemoteException;
    }
    public class RemoteHelloWorld extends UnicastRemoteObject implements IRemoteHelloWorld{
        protected RemoteHelloWorld() throws RemoteException{
            super();
        }//⼀个实现了此接⼝的类
        public String hello() throws RemoteException{
            System.out.println("call from 10.23.71.34.....");
            return "hello world";
        }
//        public String exec() throws RemoteException{
//            System.out.println("call from 10.23.71.34.....");
//            try {
//                Runtime.getRuntime().exec("calc");
//            } catch (IOException e) {
//                e.printStackTrace();
//            }
//            return "nice";
//        }
    }
    private void start() throws Exception{
        RemoteHelloWorld h = new RemoteHelloWorld();
        LocateRegistry.createRegistry(1099);
        Naming.rebind("rmi://127.0.0.1:1099/Hello",h);
    }

    public static void main(String[] args) throws Exception {
        //⼀个主类⽤来创建Registry，并将上⾯的类实例化后绑定到⼀个地址。这就是我们所谓的Server了
        new RMIServer().start();
    }
}

```

**RMIClient.java**

```
package rmi;

import java.rmi.Naming;

public class RMIClient {
    public static void main(String[] args) throws Exception{
        //使⽤ Naming.lookup 在Registry中寻找到名字是Hello的对象
        RMIServer.IRemoteHelloWorld hello = (RMIServer.IRemoteHelloWorld)
                Naming.lookup("rmi://10.23.71.34:1099/Hello");
        String[] s = Naming.list("rmi://10.23.71.34:1099");//
        System.out.println(s);
        String ret = hello.hello();
        System.out.println(ret);

    }
}

```

虽说执⾏远程⽅法的时候代码是在远程服务器上执⾏的，但实际上我们还是需要知道有哪些⽅法，这时候接⼝的重要性就体现了，这也是为什么我们前⾯要继承 Remote 并将我们需要调⽤的⽅法写在接⼝IRemoteHelloWorld ⾥，因为客户端也需要⽤到这个接⼝。

我们可以抓包看看RMI的通信过程。

[![](_v_images/20210527141020405_22101.png)](https://p2.ssl.qhimg.com/t015ee0cba9d084bc87.png)

这就是完整的通信过程，我们可以发现，整个过程进⾏了两次TCP握⼿，也就是我们实际建⽴了两次TCP连接。

第⼀次建⽴TCP连接是连接远端 10.23.71.34 的1099端⼝，这也是我们在代码⾥看到的端⼝，⼆者进⾏沟通后，我向远端发送了⼀个“Call”消息，远端回复了⼀个“ReturnData”消息，然后我新建了⼀个TCP连接，连到远端的53758端⼝。（应该来说每个人连到远端端口不一样）

[![](_v_images/20210527141020160_32678.png)](https://p5.ssl.qhimg.com/t016210c896f395c13f.png)

下面的新建的TCP连接的数据包

[![](_v_images/20210527141019911_15823.png)](https://p3.ssl.qhimg.com/t01007eaf49d7a0e401.png)  
所以捋⼀捋这整个过程，⾸先客户端连接Registry，并在其中寻找Name是Hello的对象，这个对应数据流中的Call消息；然后Registry返回⼀个序列化的数据，这个就是找到的Name=Hello的对象，这个对应数据流中的ReturnData消息；客户端反序列化该对象，发现该对象是⼀个远程对象，地址在10.23.71.34:53758 ，于是再与这个地址建⽴TCP连接；在这个新的连接中，才执⾏真正远程⽅法调⽤，也就是 hello() 。

我们借⽤下图来说明这些元素间的关系（来原p师傅）

[![](_v_images/20210527141019558_2988.png)](https://p2.ssl.qhimg.com/t01c94ab682c9febf36.png)

简单的说：RMI Registry就像⼀个⽹关，他⾃⼰是不会执⾏远程⽅法的，但RMI Server可以在上⾯注册⼀个Name到对象的绑定关系；RMI Client通过Name向RMI Registry查询，得到这个绑定关系，然后再连接RMI Server；最后，远程⽅法实际上在RMI Server上调⽤。

其实下面的一段代码中包含了RMI Registry和Server俩部分。第一行创建并运行RMI Registry，第二行将RemoteHelloWorld对象绑定到Hello这个名字上。

```
LocateRegistry.createRegistry(1099);
Naming.bind("rmi://127.0.0.1:1099/Hello", new RemoteHelloWorld());

```

## RMI 2

我们可以在客户端利用list和lookup方法远程调用。

```
String[] s = Naming.list("rmi://10.23.71.34:1099");//list方法可以列出目标上所有绑定的对象：
System.out.println(s);
Remote a =Naming.lookup("rmi://10.23.71.34:1099");
System.out.println(a);

```

lookup作用就是获得某个远程对象,那么，只要目标服务器上存在一些危险方法，我们通过RMI就可以对其进行调用，[工具](https://github.com/NickstaDB/BaRMIe)，其中一个功能就是进行危险方法的探测。

[![](_v_images/20210527141019313_31398.png)](https://p5.ssl.qhimg.com/t01b598021839e4edaa.png)

## 反序列化

> p师傅的思想：Java设计 readObject 的思路和PHP的`__`wakeup 不同点在于： readObject 倾向于解决反序列化时如何还原一个完整对象这个问题，而PHP的 `__`wakeup 更倾向于解决反序列化后如何初始化这个对象的问题。

Java在序列化时一个对象，将会调用这个对象中的 writeObject 方法，参数类型是

ObjectOutputStream ，开发者可以将任何内容写入这个stream中；反序列化时，会调用

readObject ，开发者也可以从中读取出前面写入的内容，并进行处理。

请注意，一个类的对象要想序列化成功，必须满足两个条件：

该类必须实现 java.io.Serializable 接口。

该类的所有属性必须是可序列化的。如果有一个属性不是可序列化的，则该属性必须注明是短暂的。如果你想知道一个 Java 标准类是否是可序列化的，请查看该类的文档。检验一个类的实例是否能序列化十分简单， 只需要查看该类有没有实现 java.io.Serializable接口。

自己写了一个demo了更好的理解

```
package xlh;

import java.io.*;

public class Person implements java.io.Serializable{
    public String name;
    public int age;

    Person(String name,int age){
        this.name = name;
        this.age = age;
    }
    public static void serialize(Object obj) throws IOException {
        FileOutputStream fileOut = new FileOutputStream("1.ser");
        ObjectOutputStream out = new ObjectOutputStream(fileOut);
        out.writeObject(obj);
        out.writeObject("this is object");
        out.close();fileOut.close();
    }
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Person person = new Person("LX",21);
        serialize(person);
        FileInputStream fileIn = new FileInputStream("1.ser");
        ObjectInputStream in = new ObjectInputStream(fileIn);
        Person e = (Person)in.readObject();//反序列化
        System.out.println("姓名:"+e.name+"年龄:"+e.age);
        in.close();fileIn.close();
    }
}

```

然后我们使用工具SerializationDumper.jar对生成的序列化文件1.ser进行分析

```
java -jar SerializationDumper.jar -r 1.ser > 1.txt

```

```
STREAM_MAGIC - 0xac ed //声明使用了序列化协议
STREAM_VERSION - 0x00 05 //序列号协议的版本
Contents
  TC_OBJECT - 0x73  //声明这是一个新的对象
    TC_CLASSDESC - 0x72 //声明这里开始一个新的class
      className
        Length - 10 - 0x00 0a  //class名字的长度
        Value - xlh.Person - 0x786c682e506572736f6e //class类名
      serialVersionUID - 0xfd ad 62 68 96 15 ea 2c //序列化ID。如果没有指定，则会由算法随机生成一个8byte的ID
      newHandle 0x00 7e 00 00
      classDescFlags - 0x02 - SC_SERIALIZABLE //标记号，该值声明该对象支持的序列化
      fieldCount - 2 - 0x00 02 //该类所包含的域个数
      Fields
        0:
          Int - I - 0x49  //域类型，49表示"I",也就是Int
          fieldName
            Length - 3 - 0x00 03   //域名字的长度
            Value - age - 0x616765 // 域名字的描述
        1:
          Object - L - 0x4c //域类型
          fieldName
            Length - 4 - 0x00 04   //域名字的长度
            Value - name - 0x6e616d65  // 域名字的描述
          className1
            TC_STRING - 0x74  //代表一个new String 用String来引用对象
              newHandle 0x00 7e 00 01
              Length - 18 - 0x00 12  //该String的长度
              Value - Ljava/lang/String; - 0x4c6a6176612f6c616e672f537472696e673b  //JVM的标准对象签名表示法
      classAnnotations
        TC_ENDBLOCKDATA - 0x78  //对象数据块结束的标志
      superClassDesc
        TC_NULL - 0x70  //说明没有其他超类的标志
    newHandle 0x00 7e 00 02
    classdata
      xlh.Person
        values
          age
            (int)21 - 0x00 00 00 15  //age域的值
          name
            (object)
              TC_STRING - 0x74  //代表一个new String 用String来引用对象
                newHandle 0x00 7e 00 03
                Length - 2 - 0x00 02  //name域的长度
                Value - LX - 0x4c58  //name域的值
TC_STRING - 0x74  //代表一个new String 用String来引用对象
    newHandle 0x00 7e 00 04
    Length - 14 - 0x00 0e  //域的长度
    Value - this is object - 0x74686973206973206f626a656374  //域的值

```

然后我们知道通过readObject ()反序列化类的时候，如果没有控制好(白名单或黑名单)就可能出现问题造成rce。

这里就不在分析URLDNS啦，因为比较简单。就放一张照片吧(做的不好~~2333)

[![](_v_images/20210527141019073_11339.png)](https://p2.ssl.qhimg.com/t01433ef2ca82c8b4d4.png)

然后就是我们经常说的CC1。流程图如下（网上已经有很多很多好的文章了这里就不在分析）

[![](_v_images/20210527141018725_14550.png)](https://p4.ssl.qhimg.com/t01b1bdc513494f7b1e.png)

不过ysoserial中的利用链却不是TransformedMap，而是**LazyMap**

LazyMap和TransformedMap类似，都来自于Common-Collections库，并继承AbstractMapDecorator。

LazyMap的漏洞触发点和TransformedMap唯一的差别是，TransformedMap是在写入元素的时候执行transform，而LazyMap是在其get方法中执行的 factory.transform 。其实这也好理解，LazyMap的作用是“懒加载”，在get找不到值的时候，它会调用 factory.transform 方法去获取一个值：

```
public Object get(Object key) {
    // create value for key if key is not currently in the map
    if (map.containsKey(key) == false) {
        Object value = factory.transform(key);
        map.put(key, value);
        return value;
    }
    return map.get(key);
}

```

不过这样就不能直接通过sun.reflect.annotation.AnnotationInvocationHandler中的readObject方法直接调用到Map的get方法，因为readObject方法中没有直接调用到Map的get方法。

所以ysoserial找到了另一条路，AnnotationInvocationHandler类的invoke方法有调用到get：

[![](_v_images/20210527141017940_21167.png)](https://p4.ssl.qhimg.com/t01a7e757b8dd1de799.png)

而这过程用到了java中的对象代理。

java中也有类似于php中的魔术方法`__call`，就是java.reflect.Proxy

```
Map proxyMap = (Map) Proxy.newProxyInstance(Map.class.getClassLoader(), new
Class[] {Map.class}, handler);

```

Proxy.newProxyInstance 的第一个参数是ClassLoader，我们用默认的即可；第二个参数是我们需要代理的对象集合；第三个参数是一个实现了InvocationHandler接口的对象，里面包含了具体代理的逻辑。

(这里代理相当于，我们需要找某个人，但是不能直接去找他而是需要通过找和这个人有关系的人才能实现找到他)

更加简单的说如果一个对象用Proxy进行代理只要调用任意方法，就会进入到 该对象的invoke方法中（不知道这样理解对不对，如果有错希望师傅们指出）

## 总结

上面基本上是对p师傅项目的内容进行学习与总结，不过自己也想出了自己认为好的学习方法，也就是在学习java反序列化的时候将链子通过照片的形式展示出来，如上面的urldns和cc1。因为比较烦锁，上面也就只是分享了两个。接下来自己会制作全部的链子（看情况），如果有师傅也感兴趣我们可以一起交流交流通过学习。