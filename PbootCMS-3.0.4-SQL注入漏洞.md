# PbootCMS 3.0.4 SQL注入漏洞

## 漏洞描述

PbootCMS是全新内核且永久开源免费的PHP企业网站开发建设管理系统，是一套高效、简洁、 强悍的可免费商用的PHP CMS源码，但存在SQL注入漏洞，攻击者可构造恶意语句进行获取敏感数据。

## 漏洞影响

> PbootCMS3.0.4

## FOFA

> app="PBOOTCMS"

## 源码分析

漏洞代码位置：

```
core\basic\Model.php
```

![1](_v_images/PbootCMS/1.png)

当传递的参数$where是一个数组时就遍历数组，当$where是一个索引数组时则：$where_string.=$value。

接下来找到“$where”函数中要传递的代码为索引数组时的代码：

```
apps\home\controller\ParserController.php
```

在“parserSearchLabel()”方法中，传入的数据被分配到变量“$receive”进行遍历，“$key”被带入“request()”进行过滤。

![2](_v_images/PbootCMS/2.png)

![3](_v_images/PbootCMS/3.png)

![4](_v_images/PbootCMS/4.png)

![5](_v_images/PbootCMS/5.png)

![6](_v_images/PbootCMS/6.png)

![7](_v_images/PbootCMS/7.png)

![8](_v_images/PbootCMS/8.png)

![9](_v_images/PbootCMS/9.png)

![10](_v_images/PbootCMS/10.png)

通过上述方法传入索引数组的值只能包含中文、字母、数字、水平线、点、逗号和空格！它由“htmlspecialchars()”和“addslashes()”编码。
最后，它被传递到“$where3”。

![11](_v_images/PbootCMS/11.png)

![12](_v_images/PbootCMS/12.png)

![13](_v_images/PbootCMS/13.png)

“getlists()”中的“$where3”是可控的，它将以“and”的形式进入语句，所以最终造成了SQL注入。

## 本地复现

默认数据库是sqlite。为了测试方便，我们需要用mysql数据库替换默认数据库。
mysql数据库目录：

```
pbootcms\static\backup\sql\0cb2353f8ea80b398754308f15d1121e_20200705235534_pbootcms.sql
```

![14](_v_images/PbootCMS/14.png)

接下来，我们以POST的形式发送索引数组，还记得源码里数组中的值要以“and”的形式进入“where”条件：

![15](_v_images/PbootCMS/15.png)

当条件为真时：

![16](_v_images/PbootCMS/16.png)

![17](_v_images/PbootCMS/17.png)

当条件为假时：

![18](_v_images/PbootCMS/18.png)



![19](_v_images/PbootCMS/19.png)有效载荷：
由于数据经过过滤，因此只能使用“正则表达式”进行常规匹配。
例如：“用户名=管理员”可以表示为“用户名regepx 0x5E612E2A”，其中“5E612E2A”是“^ a”的十六进制代码。

![20](_v_images/PbootCMS/20.png)

![21](_v_images/PbootCMS/21.png)

就可以获得管理员的账号密码了。