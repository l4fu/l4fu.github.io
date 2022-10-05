# Web权限维持
通过对webshell的动静态免杀绕过防护软件，进行权限维持。通过修改webshell时间戳，放到不被管理员关注的一些深层目录中，去除敏感shell函数特征，通过对shell流量双向加密去避开常规waf检测

### 4.1Webshell隐藏

使用windows自带命令行工具attrib用来显示或更改文件属性。

```
attrib +r +s +h
```

![.png](_v_images/20200817103128112_22498.png)

### 4.2配置文件型后门

在.htaccess中添加php解析的新后缀并上传，之后上传该后缀的木马即可。

```
AddType application/x-httpd-php .txt
```

![.png](_v_images/20200817103127792_13435.png)

![.png](_v_images/20200817103127586_20492.png)

### 4.3中间件后门

将编译好的so文件添加到php.ini的extension中。当模块被初始化时，会去加载执行我们的代码。当发送特定参数的字符串过去时，即可触发后门。