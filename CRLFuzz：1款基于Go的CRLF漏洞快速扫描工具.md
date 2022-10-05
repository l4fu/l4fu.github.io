# CRLFuzz：一款基于Go的CRLF漏洞快速扫描工具
## CRLFuzz

CRLFuzz是一款功能强大的CRLF漏洞扫描工具，该工具基于Go语言开发，可以帮助广大研究人员以非常快的速度完成CRLF漏洞的扫描工作。

## 工具安装

### 源码安装

该工具的安装非常简单，广大研究人员可以直接从该项目的Release页面下载预编译的源码，下载完成后解压即可运行，或者直接使用下列命令安装：

```
curl -sSfL https://git.io/crlfuzz | sh -s -- -b /usr/local/bin
```

### 源安装

如果你在你的本地设备上安装好了Go v1.13+环境，可以通过下列命令进行工具的安装和配置：

```
GO111MODULE=on go get -v github.com/dwisiswant0/crlfuzz/cmd/crlfuzz
```

如需更新工具，你可以使用-u参数来运行go get命令。

### GitHub安装

除此之外，广大研究及人员还可以使用下列命令从GitHub来安装并配置该工具：

```
git clone https://github.com/dwisiswant0/crlfuzz
```

## 工具使用

### 基础使用

该工具的使用也非常简单，我们可以直接运行下列工具来扫描CRLF漏洞：

```
crlfuzz -u "http://target"
```

### 参数选项

```
crlfuzz -h
```

上述命令将会显示工具的帮助命令，下面是该工具支持的所有参数选项：

![图片](_v_images/64283317237632.webp)

### 目标选择

我们可以通过下列三种方式来定义待扫描的目标：

单一URL：

```
crlfuzz -u "http://target"
```

URL列表：

```
crlfuzz -l /path/to/urls.txt
```

Stdin输入（支持其他工具获取输入）：

```
subfinder -d target -silent | httpx -silent | crlfuzz
```

### 请求方法

默认配置下，CRLFuzz会使用GET方法来发送请求。如果你想要修改请求发送方法，可以使用-X参数来修改：

```
crlfuzz -u "http://target" -X "GET"
```

### 结果输出

我们还可以使用-o选项来将扫描结果保存至文件中：

```
crlfuzz -l /path/to/urls.txt -o /path/to/results.txt
```

### 数据

如果你想要使用POST、DELETE、PATCH或其他方法来发送一个数据请求，你可以使用-d参数：

```
crlfuzz -u "http://target" -X "POST" -d "data=body"
```

### 添加Header

我们可以使用下列方法来使用自定义Header来添加Cookie或其他东西：

```
crlfuzz -u "http://target" -H "Cookie: ..." -H "User-Agent: ..."
```

### 使用代理

在使用代理时，可以使用一个protocol://前缀来定义代理字符串，并指定代理协议：

```
crlfuzz -u "http://target" -x http://127.0.0.1:8080
```

### 并发

在使用该工具时，我们可以指定模糊测试的并发数。CRLFuzz的默认并发数为25，可以使用-c参数来修改：

```
crlfuzz -l /path/to/urls.txt -c 50
```

### 静默模式

我们可以使用-s参数来激活工具的静默模式，此时我们将只能看到存在漏洞的目标：

```
crlfuzz -l /path/to/urls.txt -s | tee vuln-urls.txt
```

### Verbose模式

跟静默模式不同，该模式将显示详细的扫描信息，该模式使用-v参数激活：

```
crlfuzz -l /path/to/urls.txt -v
```

### 版本信息

我们可以使用-V参数来查看CRLFuzz的版本信息：

```
crlfuzz -V
```

### 代码库

我们还可以将CRLFuzz以代码库的形式使用，或整合进其他项目之中：

```
package main
```

## 工具运行截图

![图片](_v_images/63213317213213.webp)

## 许可证协议

CRLFuzz项目的开发与发布遵循MIT开源许可证协议。

## 项目地址

CRLFuzz：https://github.com/dwisiswant0/crlfuzz