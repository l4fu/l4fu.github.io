# mimikatz 提取用户凭证

Mimikatz提取用户凭证功能，其主要集中在sekurlsa模块，该模块又包含很多子模块，如msv,wdigest,kerberos等，使用这些子模块可以提取相应的用户凭证，如sekurlsa::wdigest提取用户密码明文，sekurlsa::kerberos提取域账户凭证，sekurlsa::msv提取ntlm hash凭证，在kuhl\_m\_c_sekurlsa这个数组里面有各个子模块功能的注释，如下：

![15833049322348](_v_images/20200520155410968_3464.png =1000x)

Mimikatz使用了一个的框架，来处理对内存的操作。以wdigest子模块为例，如果在目标机器上运行mimikatz，会跨进程读取读取lsass.exe进程的wdigest.dll模块的数据，如果是用dump文件方式提取密码则是直接打开文件。

 [![.png](_v_images/20200520154956671_1265.png!small =1000x)](https://.3001.net/_v_s/20200304/1583304941438.png)

## 获取用户凭证信息

通过遍历加载的DLL模块名称，来初始化Mimikatz关注的一些DLL的统计信息结构体。

[![.png](_v_images/20200520154953460_31539.png!small =1000x)](https://.3001.net/_v_s/20200304/1583304962404.png)

重点是第二步和第三步。

第二步调用了kuhl\_m\_sekurlsa\_utils\_search继而调用kuhl\_m\_sekurlsa\_utils\_search_generic如下，

[![.png](_v_images/20200520154949651_14318.png!small =1000x)](https://.3001.net/_v_s/20200304/15833049759619.png)

搜索的是LsaSrv.dll的特征码，结合偏移量找到所有的登录会话信息。

[![.png](_v_images/20200520154949144_15935.png!small =1000x)](https://.3001.net/_v_s/20200304/15833049854120.png)

最后打印出来的会话信息

 [![.png](_v_images/20200520154948435_14432.png!small =1000x)](https://.3001.net/_v_s/20200304/15833049944353.png) 
## 获取加密用户密码的密钥

第三步调用了lsassLocalHelper->AcquireKeys(&cLsass, &lsassPackages\[0\]->Module.Informations);，实际对于NT6系统实际调用的是kuhl\_m\_sekurlsa\_nt6\_acquireKeys

[![.png](_v_images/20200520154947027_30157.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050097309.png)

用PTRN\_WALL\_LsaInitializeProtectedMemory_KEY作为特征码进行搜索

[![.png](_v_images/20200520154946518_20191.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050347292.png)

获取初始化向量和密钥本身

[![.png](_v_images/20200520154945910_1349.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050495087.png)

## 解密账户密码

基于2.1得到的登录会话信息（包含加密后的用户密码）和2.2得到的加密密钥和初始化向量，mimikatz的wdigest子模块便可以提取用户密码明文。

[![.png](_v_images/20200520154943402_6099.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050667155.png)

kuhl\_m\_sekurlsa\_genericCredsOutput实际调用kuhl\_m\_sekurlsa\_nt6_LsaEncryptMemory进行密码的密文解密。

[![.png](_v_images/20200520154940688_14163.png!small =1000x)](https://.3001.net/_v_s/20200304/15833050827750.png)

最后根据密文长度是否是8的倍数，来调用Aes解密和Des解密（BCryptDecrypt）。
