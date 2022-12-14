# linux基础-命令
![图片](_v_images/20210219085459435_17562)

  

## 第1章：Linux发行版及核心组件  

理解什么是Linux的发行版，然后选择适合自己的版本，掌握安装Linux的步骤，建立对Linux的确切认识。  

###  Linux介绍：  

 Linux操作系统的组成部分如下：  

- 用户工具：指提供用户使用的软件

- 服务器端软件：指用来提供某些功能或通过网络提供某种服务的软件。

- Shell:通过命令行与系统内核交互，需要运行Shell程序。

- 文件系统：文件和目录存储在一个特定的结构中，这种结构就叫文件系统

- 内核：系统的核心控制部分，负责与硬件的交互来实现系统的核心功能。

- 内核模块：为内核提供更多功能。

- GUI软件：为系统提供窗口式的交互界面。

- 库文件：某个软件用来完成特定任务而依赖的软件合集。

- 设备文件：在Linux中，一切皆是文件，包括硬件设备，系统通过设备文件与硬件通信。


安全提醒：

 Linux系统上一般指安装所需的软件，因为任何软件都会带来安全隐患，可能为成为攻击的点，所以要进来少安装不必要的软件，减少安全隐患。  

  

###   Linux发行版：  

常见的发行版：  

- 商业版：出于商业目的设计的发行版，会附有绑定的服务条款。附带的服务一般按年机费，为了让产品更加安全稳定，一般句也更缓慢的更新周期，典型发商业发行版有：Red Hat Enterprise Linux或许SUSE。

- 桌面或非专业版：主要是给不想用MAC OS和Windows的个人用户使用的，一般只有社区的技术支持，但更新速度非常快，典型的有：Fedora、Linux Mint和Ubuntu。

- 安全增强版：专门针对安全性设计的，本身句也额外的安全特性，或者提供可以用来增强安全性的攻击。典型的有：Kali Linux和Alpine Linux。

- 体验版：可直接在CD-ROM、DVD或U盘等移动介质启动，好处是可以不对服务器做任何修改进行测试。典型的有：Manjaro Linux 和Sntegros。


安全提醒：

商业版一般来说比家庭版更具有安全性。因为商业版一般在公司或政府部门中用来承载关键业务，所以商业版通常把安全性看作一个关键点。  
需要重点指出的是：以上只是发行版中的很少一部分，还有很多其他的发行版，有个非常好的网站：https://distrowatch.com/，在上面可以了解更多可以的发行版，也可搜索下载不同发行版所需的软件。

####  Shell  

Shell是一个使得用户可以提交给系统的接口。和windows的dos类似，Shell还提供了一个命令行接口（CLI),它的功能比图形界面更强大。

####  GUI软件  

GUI(图形界面）软件可以让你通过键盘鼠标与系统交互，对个人笔记本GUI是很好选择，不过一般GUI很可能是系统资源占用大户，所以不会在服务器上安装GUI软件。  

安全提醒：

可以认为每次向操作系统添加软件都会添加潜在安全风险，所以要对相应的软件进行正确的安全加固，尤其是提供给用户访问的软件尤为重要。  

  

###  安装Linux

选择一个适合自己的发行版，然后安装在虚机或物理机上使用，这里就不多赘述了。  

  

## 第2章：使用命令行

Linux有一个非常有魅力的特性，那就是有大量的命令行工具。多达几千个命令，为用户使用系统提供了非常大的灵活性。  

### 文件管理

Linux系统包含大量文件和目录，在上面的主要操作就是管理文件。  

####  Linux文件系统

![图片](_v_images/20210219085459004_32400)

####  执行命令

执行Shell命令的标准方式是在命令提示符下键入命令按下回车，例子如下：

![图片](_v_images/20210219085458676_31940)

命令也可以接受参数和选项：  

- 选项是可以改变命令结果预定义的值

- 一般选项都是一个单字符加连字符“ \- ”

- 参数是一个附加信息。

- 和选项不同，参数不是以单个（或多个）连字符开始


![图片](_v_images/20210219085458467_8131)

####  pwd命令  

 pwd(输出当前目录）命令显示了shell的当前目录

![图片](_v_images/20210219085458152_27528)

####  cd命令  

cd(切换目录）命令，cd命令是那种“没有消息就是好消息”的命令。命令成功没有回显（但是注意提示符改变）如何失败会 显示错误。  

![图片](_v_images/20210219085457830_21157)

  

cd的命令，还有一些特殊字符表示目录：  

- 两个点（..)的字符表示当前目录的上一级目录。

- 一个点（.)表示当前目录

- 波形字符（~）表示用户的家目录  


####  ls命令  

ls命令用来列出目录中的文件。默认列出当前目录中的文件  

![图片](_v_images/20210219085457413_20498)

和cd命令类似，参数可以使用绝对路径和相对路径列出其他目录文件。

ls命令常用选项：  

- -a : 列出所有文件，包括隐藏文件。

- -d：列出目录名，不列出目录的内容

- -F：在文件名后面追加一个字符代表文件的类型，例如 *（可执行文件），/（目录）以及@（软链接文件）

- -h：当和-l一起使用时，以便于阅读的格式显示文件大小

- -l：以长列表显示文件

- -r：逆序显示文件

- -s：按文件大小显示

- -t：按文件修改时间显示  


注意：在Linux中，命令、选项、参数以及所有其他的输入都区分大小写。  

  

ls -l输出的结果：  

![图片](_v_images/20210219085456993_29829)

  

####  文件名匹配：  

文件名匹配符（也叫通配符）是在命令行中，用代表一个或多个文件名字符的特殊字符。  

- * ：匹配文件名中零个或多个字符

- ？:  匹配文件名中的任意单个字符

- \[ \] : 匹配文件名中的单个字符，只要这个字符在\[ \]里  


####  file命令  

file输出文件内容的类型  

![图片](_v_images/20210219085454972_1903)

  

####  less命令  

用来显示内容非常多的文本文件，但是显示一页会暂停，按键盘上的某些键可以滚动浏览整个文件。  

- h : 显示帮助界面

- 空格 ： 当前页前进一页99

- b : 当前页后退一页

- 回车：当前页向下移动一行，下箭头也可以实现

- 上箭头：当前页上移一行

- /term ：在文档中搜索term的内容

- q : 退出文档浏览回到shell  


  

####  head 命令  

显示文本文件的头部内容，默认显示前十行。使用-n选项显示指定行数。

![图片](_v_images/20210219085454659_8463)

  

####  tail命令  

显示文本的尾部内容，默认显示后十行，使用-n选项显示指定行数

![图片](_v_images/20210219085454237_6394)

####  mkdir命令  

mkdir命令用于创建目录  

常用重要的命令选项如下：

- -p : 创建多级不存在的目录

- -v ：显示创建的每个目录的信息


  

####  cp命令  

此命令用于复制文件或目录  

注意：必须给出复制文件的目标目录

![图片](_v_images/20210219085453918_3704)

常用命令选项：  

- -i : 如复制会导致覆盖，则提示是否确认覆盖

- -n : 从不覆盖已存在的文件

- -r : 复制整个目录（r代表递归）

- -v ：详细模式  


####  mv命令  

移动或者重命名一个文件

常用命令选项：  

- -i : 如移动会导致覆盖，则提示是否确认覆盖

- -n:从不覆盖已存在的文件

- -v :详细模式  


####  rm命令  

用来移除（删除）文件或目录  

常用选项如下：  

- -i : 删除文件之前提示是否删除

- -r : 删除整个目录结构（r代表递归）

- -v : 详细模式  


####  rmdir命令  

用于移动（删除)空目录。如果目录不是空的，则此命令失败（用rm -r 来删除）  

####  touch命令  

此命令有两个功能，创建一个空文件及更新一个已存在文件的访问和修改时间戳。  

常用命令选项：  

- -a : 只改变文件的访问的时间戳，不改变文件的修改的时间戳

- -d date ：设置文件的时间戳为特定时间

- -m : 只改变文件的修改的时间戳，不改变文件的访问时间戳

- -r file : 使用file文件的时间戳作为参考值去设置指定文件的时间戳  


  

###  Shell特性

####  shell变量  

shell变量用于在shell内保存信息，便于操作使用。

echo 命令用来显示信息，一般显示变量的值。

echo命令常用字符串：  

- \\n :换行符

- \\a :终端提示音

- \\t ：制表符

- \\\ ：一个反斜杠字符  


set 命令在不带参数执行时显示所有shell变量和值。  

常用命令选项：  

- -b：后台任务被终止时向shell报告。

- -n：读取脚本命令但不执行，检查语法错误时很有用

- -u: 使用未定义的变量时提示错误信息

- -C：使用重定向时不允许覆盖已存在文件  


unset 命令是从shell环境中移除一个变量  

PS1变量定义终端主提示符，通常使用特定字符串  

PATH变量：预定义命令路径，即可在命令提示符中直接键入。  

环境变量：创建变量时，它仅可在shell中使用的本地变量，其他命令不可调用，使用export命令转换成环境变量，即可被调用。  

env命令显示当前shell中的环境变量，执行时本地变量不会显示。  

  

####  shell的初始化文件

![图片](_v_images/20210219085453600_524)

####  别名  

别名是用一个单个“命令”来执行一组命令的shell特性。  

![图片](_v_images/20210219085453382_20959)

显示所有别名直接执行alias命令即可  

撤销别名使用unalias命令

####  历史命令  

内存里会保存之前执行的命令列表。可用通过history命令查看，例如列出最后5个命令：  

![图片](_v_images/20210219085453054_22609)

常用的命令选项：

- -c ：清空当前历史命令列表

- -r ：读取用于保存历史命令文件的内容

- -w：将当前的历史命令列表写入到历史命令文件中  


如过你想执行历史命令列表中的一个命令，输入！后面紧跟命令。例如想要执行第3个命令（之前的ls命令），输入如下：

![图片](_v_images/20210219085452634_24346)

执行之前命令的技巧：

- !！：执行历史命令列表中的上一个命令

- !-n：执行历史 命令列表中的倒数第n个命令（例如：!-2）

- !string：执行历史命令列表中上一个以此字符串开头的命令

- !?string：执行历史命令列表中上一个在任意位置含此字符的命令

- ^str1^str2：执行上一个命令，但是用str2替换掉str1  


####  输入输出重定向  

每个命令都偶两个输出流（输出和错误输出），并且可用接收一个数据流（输入）。  

重定向方法：  

| <br> | <br> |
| :-: | :-: |
| 方法   | 描述   |
| cmd < file   | 指定文件内容替代输入   |
| cmd > file   | 输出到指定文件   |
| cmd 2>file| 错误输出到指定文件   |
| cmd &> file| 输出和错误输出同时输到指定文件   |
| cmd 1 | cmd2   | 将cmd1的输出作为cmd2d 输入   |

管道符：  

用管道符（之所以这么叫是因为 | 字符被称之为“管道”）将一个命令的输出发送到另外一个命令使命令行功能更强大。  

![图片](_v_images/20210219085452215_25712)

注意：命令的执行顺序不同产生的结果也是不同的。

![图片](_v_images/20210219085451884_21642)

子命令：  

将命令放到$( )字符中，获取该命令的输出并将其作用到另一个命令的参数。  

![图片](_v_images/20210219085451655_19179)

date和pwd输出作为参数传递给echo命令。

  

###  高级命令

####  find命令  

用不同的规则在文件系统中搜索文件及目录  

命令格式如下：  

find  \[options\] starting_point  criteria  action  

- staring_point代表从那个目录开始搜索

- criteria代表搜索什么

- action 代表对结果如何操作  


####  正则表达式

术语regex代表正则表达式（RE),是指用来匹配其他字符的一个或多个字符。  

基础正则表达式：  

| <br> | <br> |
| :-: | :-: |
| RE规则   | 描述   |
| ^   | 匹配一行的开始   |
| $   | 匹配一行的结束   |
| *   | 匹配0或多个字符   |
| . | 匹配单个字符   |
| \[\] | 以中括号为范围的一串字符，一串字符（\[abc\])或字符范围（\[a-c\]）都可以   |
| \[^\]   | 以中括号为范围的匹配不在此范围的 |
| \   | 对表达式的中的特殊字符进行转义   |

####  grep命令  

此命令是在文件中逐行查找包含指定模式的行。默认找到后显示整行。

![图片](_v_images/20210219085451233_685)

注意：上例搜索文件时使用的是基础正则表达式。

####  sed命令  

sed命令用于非交互模式下的修改文件。  

![图片](_v_images/20210219085451013_18273)

sed命令的操作：

| <br> | <br> |
| :-: | :-: |
| 操作   | 描述   |
| s/   | 用新的值替换所有匹配到的字符或表达式   |
| d   | 删除 |
| a\   | 在匹配到的行后插入数据   |
| i\ | 在匹配到的行前插入数据   |

####  压缩命令  

tar命令，一般用来打包多个文件为单个文件。  

重要选项：  

- -c :创建一个.tar文件

- -t :列出一个.tar文件的内容

- -x：提取一个.tar的文件内容

- -f：指定.tar文件的名称

- -v：输出详细信息

- -A：追加新文件到.tar文件中

- -d：比较.tar文件和目录中的文件的不同

- -u：更新，只向存在的.tar文件中追加新文件

- -j：使用bzip2工具压缩或解压一个.tar文件

- -J：使用xz工具压缩或解压一个.tar文件

- -z: 使用gzip工具压缩或解压缩一个.tar文件


  

gzip命令来压缩文件  

重要选项：  

| <br> | <br> |
| :-: | :-: |
| 选项   | 描述   |
| -c   | 输出内容到STDOUT且不可替代源文件   |
| -d   | 解压缩文件（也可使用gunzip命令）   |
| -r   | 递归：压缩目录及其子目录的所有文件 |
| -v   | 详细信息   |

gunzip命令  

此命令用来解压缩gzip压缩的文件  

bzip2命令压缩文件

注意：bzip2命令会用压缩后的文件替换原始文件。  

重要选项：  

- -c : 不替换原始文件

- -d：解压缩文件

- -v：详细信息  


  

xz命令压缩文件  

重要选项：  

| <br> | <br> |
| :-: | :-: |
| 选项   | 描述   |
| -c   | 不替代原文件，输出到新文件   |
| -d   | 解压缩文件（也可用unxz命令） |
| -l   | 列出已存在的压缩文件信息   |
| -v   | 详细信息   |

  

## 第3章 获取帮助  

###  man page  

可以使用man page获取命令和配置文件的更多信息。  

例如：相要学习更多ls命令的知识，可执行 man ls  

![图片](_v_images/20210219085450701_28799)

浏览命令：

| <br> | <br> |
| :-: | :-: |
| 移动命令   | 描述   |
| h   | 用来显示帮助界面   |
| 空格   | 前进一页   |
| b   | 后退一页 |
| 回车   | 向下移动一行，下箭头也可实现   |
| 上箭头   | 向上移动一行   |
| /term   | 在文档中搜索term的内容   |
| q   | 退出   |

####  man page组件  

![图片](_v_images/20210219085450271_30362)

![图片](_v_images/20210219085450053_12528)

- NAME:命令的名称

- SYNOPSIS:命令的简介

- DESCRIPTION：命令的细节描述

- AUTHOR：命令作者

- REPORTING BUGS：命令的问题信息发送处

- SEE  ALSO ：关于此帮助也没的其他命令或文档

- EXAMPLES:命令操作案例  


  

####  man page的分类  

| <br> | <br> | <br> | <br> |
| :-: | :-: | :-: | :-: |
| 类别   | 描述 | 类别 | 描述 |
| 1   | 可执行的命令及shell命令   | 6 | 游戏 |
| 2 | 系统调用 | 7 | 杂项 |
| 3 | 系统库调用 | 8 | 系统管理员的基本命令 |
| 4 | 特殊文件 | 9 | 内核相关内容 |
| 5 | 文件格式 |||

你可能好奇如何知道特定的man page分类名。可以使用 man -f命令来获得如下信息。  

![](_v_images/20210219085449826_15872)

注意：man  -f命令与whatis命令是相同的

![](_v_images/20210219085449714_761)

####  man page 的存储位置

某些情况下 man page不在标准的存储位置，这时需要给man page 指定一个替代的为了可以使用-M选项。

###  命令的help选项  

某些命令支持提供一些基础的帮助选项  

###  help命令

help命令只对shell内置的命令提供帮助信息，因为这些命令没有单独的man page。  

###  info命令  

与man page不同，info page更新一个阅读一个拥有众多超链接的网站

![图片](_v_images/20210219085449556_7030)

info命令的移动命令：  

| <br> | <br> |
| :-: | :-: |
| 命令   | 描述 |
| n | 移动到下一节点 |
| p | 移动到前一节点 |
| u | 移动到父节点 |
| l | 移动到最近的节点（上一次所在节点） |
| b | 移动到当前节点的开始位置 |
| t | 移动到所有节点顶部 |
| q | 退出info命令 |

###  /usr/share/doc目录  

更多的文档可以在此目录中找到。能找到什么文档取决系统安装什么软件。

###  因特网资源  

- http://www.tldp.org/

- www.linuxtoday.com

- https://www.linuxfornums.org  


  

##  第4章编辑文件

###  vi编辑器

vi编辑器是现在linux和unix系统的标准文本编辑器  

#### vim是什么

vim作为vi编辑器的复制品，有同样的基本功能，但是vim有一些额外的功能。有些发行版只有vi编辑器，许多发行版是两者都有的，也有些vi命令其实是vim编辑器的链接。

这里不多赘述：详情参考：[Linux vi/vim操作入门到精通（图文版）](http://mp.weixin.qq.com/s?__biz=MzIxMTEyOTM2Ng==&mid=2247486112&idx=2&sn=82ca941371d8cff8dabd7097e4f0db79&chksm=975b4fd9a02cc6cf3c0f3378e20f2736eb70c37b509f455ae4b45e4127972f555314f82b2302&scene=21#wechat_redirect)  

###  其他编辑器  

####  Emacs

类似vi编辑器，如果你在图形化终端只要运行emacs命令即可。

####  gedit和kwrite

是完全图形化的编辑器，类似windows的记事本。

####  nano和joe  

只可在命令行环境下使用的编辑器，所以不需要图形环境

####  lime 和bluefish

通过一下工具和特性文本文件的编辑提升到一个新的层次，是为开发人员创建代码而设计的，包括语法高亮、代码自动补全以及代码自动缩进。

  

###  第5章故障处理  

首先坏消息是：出了问题，如命令会失败、程序会崩溃、配置会出错。
而好消息是:这些问题都是由技术可以修复的。故障处理不是仅仅去凭空猜想。应该在发生故障时采取特定的步骤来定位问题，并确定最佳解决方案。

###  故障排除的科学  

首先是排除问题的一般步骤：  

- 收集与问题相关的所有信息。

- 确定哪些是最可能到账故障的原因。

- 采取行动前，把计划用来解决问题的步骤记录下来。

- 仅仅执行被记录下来的操作解决问题。

- 确认问题是否被妥善解决。

- 如果没有解决，使用步骤3，回退系统状态到你开始解决问题之前的状态。

- 如果解决了，确认你操作完是否还有其他问题。

- 实用一种便于查询的技术，把文档保存下来，包括未能解决问题的操作。

- 考虑下你还能做些什么来预防此类问题再次发生。  


### 通知用户  

确保用户及时了解网络或系统中的更改非常重要。  

#### 登陆前和登陆后的消息  

/etc/issue文件  

登陆前提升系统的名称以及内核版本，都是来自此文件。  

文件中的可用特殊值：

- \\d 本地端时间的日期  

- \\l 显示第几个终端机的接口  

- \\n 显示主机的网络名称  

- \\o 显示 domain name  

- \\r 操作系统的版本 (类似 uname-r)  

- \\t 显示本地端时间的时间  

- \\s 操作系统的名称  

- \\v 操作系统的版本 

-  \\r详细的内核版本  

- \\m给出当前操作系统的位数


注意：这些值实际商被一个叫agetty的进程使用。

  

/etc/motd文件  

登陆进系统后，提示的信息在此文件中编辑。

内容设置给一些建议：

- 一个友好的欢迎信息

- 一个“千万不要”的警告信息

- 服务器将要做的变动

- 服务器的用途  


  

  

####  广播消息  

wall命令  

给所有用户终端商发广播消息。  

注意：wall命令发的消息不能超过20行  

shutdown命令  

你想给用户传递信息你要关闭系统，可以使用shutdown命令