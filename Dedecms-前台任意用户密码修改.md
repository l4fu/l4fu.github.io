# Dedecms 前台任意用户密码修改

## 影响版本

dedecmsV5.7 SP2

## 漏洞成因

在用户密码重置功能处，php 存在弱类型比较，导致如果用户没有设置密保问题的情况下可以绕过验证密保问题，直接修改密码(管理员账户默认不设置密保问题)。值得注意的是修改的密码是 member 表中的密码，即使修改了管理员密码也是 member 表中的管理员密码，仍是无法进入管理。

## 复现

在找回密码处，点击通过安全问题取回

![](Dedecms%20%E5%89%8D%E5%8F%B0%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E5%AF%86%E7%A0%81%E4%BF%AE%E6%94%B9/a2ea6d0d1c946ac0f125cc858abd952a.png) 

填写信息并抓包，修改 id 和 userid 为想要重置密码的对象，再加上以上分析内容，发包即可得到修改密码 url ![](Dedecms%20%E5%89%8D%E5%8F%B0%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E5%AF%86%E7%A0%81%E4%BF%AE%E6%94%B9/d05ffaa4c133f9a4d2af347cd61b15b1.png) 进入该url，修改密码。 ![](Dedecms%20%E5%89%8D%E5%8F%B0%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E5%AF%86%E7%A0%81%E4%BF%AE%E6%94%B9/2b3d56c8cf5fdeb4bbaa837ae457fa08.png)

## 修复意见

改为强类型比较