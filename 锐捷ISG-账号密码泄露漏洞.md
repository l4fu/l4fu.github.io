# 锐捷ISG 账号密码泄露漏洞

## 漏洞描述

锐捷ISG 存在账号密码泄露漏洞，通过查看前端，可以获取密码的md5值, 解密后获取后台权限

## 漏洞影响

> [!NOTE]
>
> 锐捷ISG

## FOFA

> [!NOTE]
>
> title="RG-ISG"

## 漏洞复现

登录页面如下

![-20210531190445488](http://wikioss.peiqi.tech/vuln/-20210531190445488.png?x-oss-process=/auto-orient,1/quality,q_90/watermark,_c2h1aXlpbi9zdWkucG5nP3gtb3NzLXByb2Nlc3M9aW1hZ2UvcmVzaXplLFBfMTQvYnJpZ2h0LC0zOS9jb250cmFzdCwtNjQ,g_se,t_17,x_1,y_10)

F12 查看到账号密码

![-20210531190500232](http://wikioss.peiqi.tech/vuln/-20210531190500232.png?x-oss-process=/auto-orient,1/quality,q_90/watermark,_c2h1aXlpbi9zdWkucG5nP3gtb3NzLXByb2Nlc3M9aW1hZ2UvcmVzaXplLFBfMTQvYnJpZ2h0LC0zOS9jb250cmFzdCwtNjQ,g_se,t_17,x_1,y_10)

解密md5 后登陆系统

![-20210531190555201](http://wikioss.peiqi.tech/vuln/-20210531190555201.png?x-oss-process=/auto-orient,1/quality,q_90/watermark,_c2h1aXlpbi9zdWkucG5nP3gtb3NzLXByb2Nlc3M9aW1hZ2UvcmVzaXplLFBfMTQvYnJpZ2h0LC0zOS9jb250cmFzdCwtNjQ,g_se,t_17,x_1,y_10)