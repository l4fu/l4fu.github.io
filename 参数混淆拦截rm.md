# 参数混淆拦截rm

使用echo等工具新建一个文件名是-rm的文件，这样删除的时候bash会认为-rmt是rm的参数。

![](images/security_wiki/15905499877887.png)


这样就可以阻止管理员去删除， 一般只有rm -- '-rm'才会删除掉。

![](images/security_wiki/15905499974780.png)


但是如果管理员使用正常删除方式会报错。

![](images/security_wiki/15905500085882.png)


可以用以下方法。

```bash
echo 123123123 > --

```

![](images/security_wiki/15905500163766.png)


这样可以看到，命令执行了，文件没删除，命令也没有报错。类似的可以利用bash的\r缺陷，和管道符、重定向符之类的。

