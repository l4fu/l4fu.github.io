## CLR

CLR的简述（来自百度百科）

```
CLR(公共语言运行库,Common Language Runtime)和Java虚拟机一样也是一个运行时环境，是一个可由多种编程语言使用的运行环境。CLR的核心功能包括：内存管理、程序集加载、安全性、异常处理和线程同步，可由面向CLR的所有语言使用。并保证应用和底层操作系统之间必要的分离。CLR是.NET Framework的主要执行引擎。

```

需要注意的是`CLR`能够劫持系统中全部`.net`程序，而且系统默认会调用`.net`程序，从而导致我们的后门自动触发，这是我们后门持久化的一个好的思路，下面来实现一下

修改一下注册表，注册表路径：`HKEY_CURRENT_USER\Software\Classes\CLSID\`，新建子项`{11111111-1111-1111-1111-111111111111}`（名字随便，只要不与注册表中存在的名称冲突就行），然后再新建子项`InProcServer32`，新建一个键`ThreadingModel`，键值为：`Apartment`，默认的键值为我们`dll`的路径

![](images/security_wiki/15906331895792.png)


然后在cmd下设置一下：

**PS：要注册为全局变量，不然只能在当前cmd窗口劫持.net程序**

```bash
SETX COR_ENABLE_PROFILING= 1 /M
SETX COR_PROFILER= {11111111-1111-1111-1111-111111111111} /M

```

然后执行一波，效果如下，可以看到已经成功劫持了

![c201801a63754b0c9629e2628056ca77](images/security_wiki/c201801a63754b0c9629e2628056ca77.gif)


