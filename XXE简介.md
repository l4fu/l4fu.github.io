## XXE 简介

XXE(XML外部实体注入、XML External Entity），在应用程序解析XML输入时，当允许引用外部实体时，可以构造恶意内容导致读取任意文件或SSRF、端口探测、DoS拒绝服务攻击、执行系统命令、攻击内部网站等。Java中的XXE支持sun.net.www.protocol里面的所有[协议](http://www.liuhaihua.cn/archives/tag/protocol)：[http](http://www.liuhaihua.cn/archives/tag/http)，[https](http://www.liuhaihua.cn/archives/tag/https)，file，[ftp](http://www.liuhaihua.cn/archives/tag/ftp)，[mail](http://www.liuhaihua.cn/archives/tag/mail)to，jar，[netdoc](http://qclover.cn/2019/01/13/java自定义通信协议及利用.html) 。一般利用file协议读取文件、利用http协议探测内网，没有回显时可组合利用file协议和ftp协议来读取文件。

