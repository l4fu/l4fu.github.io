# webshell扫描

## 方法1：在生产系统上扫描
目前已经给2000+的系统上传了 CloudWalker_webshell-detector-1.0.0-linux-amd64 这个webshell扫描工具，部分主机上传了 hm-XX的webshell扫描工具。
工具路径 /home/aqassoc/CloudWalker_webshell-detector-1.0.0-linux-amd64 ,复制工具或者自行上传，修改权限 755 即可是使用。
对于 unix 主机，  shellpub.com的扫描工具已经下线无法使用，但可以使用linux版本进行尝试。
建议 unix 主机管理员将待扫描的文件下载到本地主机，使用本地杀软或检测工具进行分析扫描。其他操作起来有困难的场景，也可以使用该方法。

- 连接运行的主机
- 上传或复制扫描工具
- 增加执行权限
```
chmod +x  CloudWalker_webshell 
#或者
chmod +x hm
```
- 扫描指定路径
第一种：生成的结果文件 result.html 
`./CloudWalker_webshell-detector-1.0.0-linux-amd64 -html -output result.html    待扫描的绝对路径`   
第二种：生成的结果文件 result.csv
`./hm scan 待扫描的绝对路径`
- 针对扫描结果进行处置：木马后门攻击场景处置手册

## 方法2：在离线环境（本地）扫描   不推荐

- 连接远程主机
- 下载待扫描的目录文件
- 使用本地的扫描工具进行查杀
- 针对扫描结果进行处置：木马后门攻击场景处置手册
- 清理下载的文件

