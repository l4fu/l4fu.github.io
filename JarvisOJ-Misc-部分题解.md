# JarvisOJ Misc 部分题解


## 上帝之音

> 这是一段神奇的声音，可是上帝之音似乎和无字天书一样，是我们这些凡人无法理解的，你能以上帝的角度，理解这段 WAV 的含义么？  
> Hint1: 你们做音频题都不喜欢看时域图？  
> Hint2: 在数据传输过程中，我们往往会使用一种自带时钟的编码以减少误码率  
> [godwave.wav.26b6f50dfb87d00b338b58924acdbea1](https://dn.jarvisoj.com/challengefiles/godwave.wav.26b6f50dfb87d00b338b58924acdbea1)

Audacity 打开就是一段稀奇古怪的音频信号，仔细观察，发现不同段落其幅值有明显差异，应该是调幅了，MATLAB 导入 wav 文件看数据，发现大概是以 64 个点为周期，那么取幅值高的为 1，幅值低的为 0。

```
clc;
clear;
y = audioread('godwave.wav');
he = 0;
data = [];
for i = 1:length(y)
    he = he + abs(y(i,1));
    if mod(i,64) == 0
        if he > 10
            data = [data,1];
        else
            data = [data,0];
        end
        he = 0;
    end
end
fid = fopen('data.txt','w');
for i = 1:length(data)
    fprintf(fid,'%d',data(1,i));
end
fclose(fid);
```

解出的数据是曼彻斯特编码，解码后是一张图片。

```
# coding=utf-8

with open('data.txt', 'r') as f:
    data = f.readline()
    print len(data)
    count = 0
    res = 0
    ans = ''
    key = ""

    while data != '':
        pac = data[:2]
        if pac != '':
            if pac[0] == '0' and pac[1] == '1':
                res = (res<<1)|0
                count += 1
            if pac[0] == '1' and pac[1] == '0':
                res = (res<<1)|1
                count += 1
            if count == 8:
                ans += chr(res)
                count = 0
                res = 0
        else:
            break
        data = data[2:]

with open('out.png', 'wb') as f2:
    f2.write(ans)
```

扫描二维码即可。

## 炫酷的战队 Logo

> 欣赏过了实验室 logo，有人觉得我们战队 logo 直接盗图比较丑，于是我就重新设计了一个，大家再欣赏下？  
> [phrack.bmp.197c0ac62c8128bc4405a27eca3021b6](https://dn.jarvisoj.com/challengefiles/phrack.bmp.197c0ac62c8128bc4405a27eca3021b6)

这张 BMP 的文件头被抹掉了，修复一下发现只是一个 logo，文件末尾藏了一张 PNG，抠出来，头部也被破坏，010 看一下应该是宽高不对，随便给一个，看到中间有一坨灰色的东西，有点像是什么字被扭曲了，图片里还有 crc 校验值，可能是要用这个校验值来反推宽高吧，但是懒得弄，直接暴力生成一下。

```
for i in range(16,256):
    print hex(i)[2:]
    b=hex(i)[2:]
    a=('89504E470D0A1A0A0000000D49484452000001'+b+'000001000802000000F37A5E12000000017352474200AECE1CE90000000467414D410000B18F0BFC6105000000097048597300000EC300000EC301C76FA8640000072549444154785EEDDAB996DD380E0050E70E2B745491B3F9FFCF1B2D04454214A537AF8EDB5D736F262E00A807CAEEE507000000000000000000DFDCC7E77F569F1FE5F9EFF2F3D7EFADBCC54D85B1F2F7AF9F6564E0EF3E2CFC059A3B77185EAB729D0ECBC51AEE0E5B94E18AE64A96F9947194EBDEF5AE54458E76339DD53C75E129F35D90B2A13FF670F045FD516E4E128B6719F3618F0439F85794FF3FC5AF255665EB3EF16641702B7D410E6DF39DFB74B5F4EAE5EED51661B8A25E903A7B64BBCC3535DF35983D020E2B9C5FBDB2E57C8CC6BCE252519F6638F89AA8E4598C47ABD361DBB3F667FC82F217AFC51FFCB2ABB2A6CECE7F0C78D3E9261D6D5C7AAFE9EBB61B3F3E874D9E3B36760F1BB94C1E17E361AEE47657BBFF54E7B27B50407759EF945DD3227BA34FC278F03551FFB35262F52B19EB1BDAB479BEA0FCC50BF1E3A73CBFC7BCE6CD9A606E7093A23BB7A1DAD5779D18BBD2058EFDC37B5D2663EE3E57BD491FFBD275E1E30A8B1263585053C3C5FCA6965112A6733C90236CCE8365A468266A998B18EE17AF3E3F2E2A2DA5C6535F469242943DBF7F97028E439FCB3F6B3336F5366FEE71FC08D56E3EDB574D6B8277B57D5D447BAF43CF7A7515BBD2C20830DC5F26CBDC835CCDCDDB2C253EAF7033AD67D11EFE52BAD0A3B2F6894B39C2260DE6A831514FDC588F735ABF8CE63CFDF1E3695A6F0A7144887CF12A73AE919A31BE925544791AFF51EDB1EC6611BCE7D48DD1BCFB483CDDF7616EFA22E2B7EA92BEC71FE4EA8BEB86E615F665A41A0F83F82365595D54B7352E736C72844D3F385CB228E311BF9CAC2C8B73C66C0ED2CFC7D329472B85682394A912EDAAE056EC3EC78B81A7F1FBB936F2A61F9ED604EF4ADD57A566BDEFC3DCD6C5287EEAFC787C90ABBF499B6715E63206AB23D0229DE164504655C34C830C23F4834D3D8BBAB21F3EECE9E29CE99DD6DDFD7C3C0D0F1252883E4254B33EE55C23A38C6D8CE7F1FB5DC7BE2286CBBA6951F0A6D47D9BA6E7EA746DCB2BB9AD8BFE5634CA4433FE20577F9336CF2BDC0DD747F18B2761066534E6B3BBE19A3CD854B559276AF9D95E774CC72972C87E3E9E5E29F522C3F29C738D8C321E11D6A7A7F1633CA5CBDB57FBD8B42E78C7A8AF5BD1ADB75DD85F866AD4D6D5BEA74EDDE72A2BBAF9C715861C244ABC28726054C6613EBBBBBEEDB98AA3BAC532358F9E635C9D759F8FA769A929C4B9CAAEC287E76E16D5ED7D4DB7F14B5DEDCAC569FBBE6C5A14BC69D0D7BDA689BB355FF73F3C1D616F73E5AFC2E66ED732DF668F42CBDABA7B58E1582AE3E357933685BF12CBF27D1FD711452E73837A7FFEFA8C64CDC24DAA26658DD5D35ACB9E589333AC6A4DAB69B06365AE30069EC76F468FC579FBFE3CAD09DE156D376BB4DAE9BDB6D18F45FDE8A9AD3B65B29D9BE74A57BA9AEE8A123A35C4C5D671C1452A631462B67D374E7C1CED3C5FE6061BEBAED3EB4E8BE3BF90F76FE6483A5022C49AF10FDABCE369B0E16FB1BA3EC1E622FEF81DAE62FBBE715A13BC2BFAF3AED1CEFDDFF779EDE8343CBE154599CC73D7B9D295EE5CEFCA33EDF6F3AE625870712AA38F32DBDA49C9FB63A54F4417346D3CE662A2597DAC5DE2F7F3F1347C9F211D76906153D34C8335199BF3B5B15E8E5F27AA76C53E3BAD09FED5CA0DC837866F2BBE797FECBBB67FAC7D46F9C62EFF8AC1F7F4673FA3F52FBCFEA0E65B2BF7CA67F4FFC33FF119D55BF0CF69FEEDDDE1A54BF97E843FE66B4BBD8AF6C7FFA11E000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000080A11F3FFE0B3B73B0698B976EA80000000049454E44AE426082').decode("hex")
    f=open('1\\'+b+'.png',"wb")
    f.write(a)
    f.close()
```

反正 Windows 的图片查看器无视 crc 校验，挨个看过去，有一张图片里可以看到 flag。

## SCAN

WireShark 打开，包并不多，真想一个一个试也完全可以。。。因为一共只有 TCP UDP ICMP 三种包，前两个量都很大，信息应该藏在 Ping 包中，观察一下，扫描应该是针对 192.168.0.99 这一 IP 的，与之对应的第四个源 IP 就是 flag。

## Class 10

> 听说神盾局的网络被日穿之后，有人从里面挖出来一个神秘的文件，咋一看也没什么，可是这可是 class10 保密等级的哦，里面一定暗藏玄机，你能发现其中暗藏的玄机吗？  
> [class10.1c40ca6a83c607f424c23402abe53981](https://dn.jarvisoj.com/challengefiles/class10.1c40ca6a83c607f424c23402abe53981)

binwalk 一下看到有两个 zlib，`foremost` 无果，试试 `binwalk -e`，提取出几个文件，其中一个是一串 01，长度是个平方数，立马联想到二维码。。。（这是另外一个故事）

```
# coding=utf-8
from PIL import Image

import math
length = int(math.sqrt(841))
im = Image.new('RGB', (length * 5, length * 5))

data = '00000001001110110101010000000011111010101001110111101111100100010111001001111101010001001000101001010010111010'\
    '10001001000101111011000111110100010011111011100001000000101111100000000101010101010101000000011111111011111001101'\
    '11111111110010000110011010010000110100101001111010100001111000111110011000011100101011100101000111111110000000001'\
    '10110111011011100001000011110000111100001110010100000100100111111111001001010000100101100010101110011100110100000'\
    '00000000000000101010101010101000001010000100100101110010110110110010100111101100000100101101000011111111100010111'\
    '00110001110111001001011000000011111010000001100111111111100100000000111011110000000100110001100101010011101111101'\
    '00100100011001110100001000101011010010101000001000010001011101010101010010011100100010101101001100101101101101111'\
    '10100000101100011011010000000001111100001100011110011'

for x in range(length):
    for y in range(length):
        if data[x * length + y] == '0':
            for ii in range(x * 5, x * 5 + 5):
                for jj in range(y * 5, y * 5 + 5):
                    im.putpixel([ii, jj], (0,0,0))
        else:
            for ii in range(x * 5, x * 5 + 5):
                for jj in range(y * 5, y * 5 + 5):
                    im.putpixel([ii, jj], (255,255,255))

im.save('out.png')
```

二维码扫一下就行了。

## 取证 2

> 还记得取证那题吗？既然有了取证神器，这里有一个可疑文件以及该存储该文件电脑的一个内存快照，那么接下来我们实战一下吧。  
> 由于文件比较大，请大家至百度云盘下载：  
> 链接: [http://pan.baidu.com/s/1c2BIGLE](https://pan.baidu.com/s/1c2BIGLE) 密码: 9v2z

这题还费了点劲，主要功夫都花在装软件上。。。题目的意思是要我们用之前考过的内存取证神奇 Volatility，然而我并不会用，而是直接上了傻瓜软件 Elcomsoft Forensic Disk Decryptor，这东西在 XP 虚拟机下居然不能用，只好又装了个 Win7 的虚拟机，dump 出一个磁盘映像，用 DiskGenius 打开即可。

## 简单网管协议

> 分析压缩包中的数据包文件并获取 flag。flag 为 32 位小写 md5。  
> 题目来源：CFF2016  
> [simple\_protocol.rar.57175cf6f8e21242822fb828735b4155](https://dn.jarvisoj.com/challengefiles/simple_protocol.rar.57175cf6f8e21242822fb828735b4155)

WireShark 打开直接搜 flag。。

## 远程登录协议

> 分析压缩包中的数据包文件并获取 flag。flag 为 32 位小写 md5。  
> 题目来源：CFF2016  
> [telnet.rar.e7dedd279f225957aad6dc69e874eaae](https://dn.jarvisoj.com/challengefiles/telnet.rar.e7dedd279f225957aad6dc69e874eaae)

过滤一下 telnet 协议，再搜 flag。

## Webshell 分析

> 分析压缩包中的数据包文件并获取 flag。flag 为 32 位小写 md5。  
> 题目来源：CFF2016  
> [findwebshell.rar.96e24e913b817b7503f85fd36e0a4f17](https://dn.jarvisoj.com/challengefiles/findwebshell.rar.96e24e913b817b7503f85fd36e0a4f17)

Webshell 很可能用了 POST 请求，过滤一下，挨个看，发现一个可疑的 base64 加密，解密后是一个网站，有个二维码，扫一下。

## Struts 2 漏洞

> 分析压缩包中的数据包文件并获取 flag。flag 为 32 位大写 md5。  
> 题目来源：CFF2016  
> [struts2.rar.5b541c4760bb277b0853ec59e8726e86](https://dn.jarvisoj.com/challengefiles/struts2.rar.5b541c4760bb277b0853ec59e8726e86)

继续搜 flag。。。

## 上帝之音 FSK

> XMAN 训练营课后作业。上帝之音 FSK 版。  
> [godwavefsk.wav.9fc29fc3b7fe4c3cb4ddb35e1e79e320](https://dn.jarvisoj.com/challengefiles/godwavefsk.wav.9fc29fc3b7fe4c3cb4ddb35e1e79e320)

这题在 XMan 的时候做过了，把之前那题的调幅改成调频而已，分析一下各个段落的频率，判断过 0 或峰值都可以。

```
clc;
clear;
y = audioread('godwavefsk.wav');

count = 0;
num = [];
for i = 1:length(y)
    if i ~= length(y)
        if y(i,1)<0 && y(i+1)>=0
            count = count + 1;
        end
    end
    if mod(i,64) == 0
        num = [num,count];
        count = 0;
    end
end
data = [];
for i = 1:length(num)
    if num(1,i)>=10
        data = [data,1];
    else
        data = [data,0];
    end
end

fid = fopen('data.txt','w');
for i = 1:length(data)
    fprintf(fid,'%d',data(1,i));
end
fclose(fid);
```

依然是曼彻斯特解码。

```
# coding=utf-8

with open('data.txt', 'r') as f:
    data = f.readline()
    print len(data)
    count = 0
    res = 0
    ans = ''
    key = ""

    while data != '':
        pac = data[:2]
        if pac != '':
            if pac[0] == '0' and pac[1] == '1':
                res = (res<<1)|0
                count += 1
            if pac[0] == '1' and pac[1] == '0':
                res = (res<<1)|1
                count += 1
            if count == 8:
                ans += chr(res)
                count = 0
                res = 0
        else:
            break
        data = data[2:]

with open('out.png', 'wb') as f2:
    f2.write(ans)
```

## OFDM

> 为了简单起见，有如下假设：  
> 1.       假设信道噪声只是简单的正态分布噪声，无镜像反射等复杂情况。信噪比 SNR=15dB  
> 2.       LTE 信道存在非常强的加密，我不需要你知道明文内容，只要知道被调制的基带数据就可以了，解调出的基带直接提交。  
> 3.       为了进一步让这道题简单一点，给出如下信息：
> 
> ```
>         信道子载波数为20
>         每子载波含符号数为6
>         每信道采用16QAM调制，每符号含比特数为4
>         OFDM调制FFT长度为512
>         保护间隔与OFDM数据比例为1:4
>         每一个OFDM符号添加的循环前缀长度为FFT长度的1/4，即保护间隔长度为128
>         窗函数滚降系数为1/32
>         OFDM循环后缀长度为20
>      以上有些信息是冗余的，自行选择需要的信息，本题由于给出了FFT长度，所以和采样频率就无关了，不要问我采样频率多少。
> ```
> 
> 4.       根据第 3 个假设给出的信息，你其实已经可以计算出 FLAG 的长度了。
> 
> ```
>      [ofdm.xls.a62a5aa4bbb68d3894cbf0bfcb2f434b](https://dn.jarvisoj.com/challengefiles/ofdm.xls.a62a5aa4bbb68d3894cbf0bfcb2f434b)
> ```

题都看不懂，GG。


## Sunyelf-Cryptography-400-20161008

> [题目地址](https://github.com/sunnyelf/Self-initiated-CTF-challenges/tree/master/Sunyelf-Cryptography-400-20161008)

### 第一层：CRC32 碰撞

打开题目压缩包，有三个 6 字节的文件，猜测是要根据这三个文件的 CRC32 值碰撞获得内容。

利用 [脚本](https://github.com/theonlypwner/crc32/blob/master/crc32.py) 碰撞得到密码：`_CRC32_i5_n0t_s4f3`。

### 第二层：维吉尼亚密码

keys.txt 中包含了密钥，找到密钥解密 ciphertext.txt。

先去 [在线解密网站](https://www.guballa.de/vigenere-solver) 解一下，出来一个很像的，但还是差一点，在 keys 中找相近的 key。

```
Clear text using key "yewrutewcybnhhipxoyubjjpqiraaymyoneomtsv":
the getenere cilger is a method of thensating alphamagic text bu tsing a series of schbycent caesar nechers basac on the letters ou u mashord it is a sticle form ob oolyalphabetic hodonttution so plofword is vefenere cipher fucha
```

找到正确的密钥 `YEWCQGEWCYBNHDHPXOYUBJJPQIRAPSOUIYEOMTSV`。

```
-- MESSAGE w/Key #1 = 'yewcqgewcybnhdhpxoyubjjpqirapsouiyeomtsv' ----------------
the vigenere cipher is a method of encrypting alphabetic text by using a series of different caesar ciphers based on the letters of a keyword it is a simple form of polyalphabetic substitution so password is vigenere cipher funny
```

### 第三层：sha1 碰撞

```
import hashlib
import itertools
import string


def sha1(s):
    sha1_hash = hashlib.sha1()
    sha1_hash.update(s)
    return sha1_hash.hexdigest()


def check(s):
    if s[0:7] == "619c20c" and s[8] == "a" and s[16] == "9":
        print("Find!")
        matched = True
        return matched

letters = itertools.product(string.printable, repeat=4)
for i in letters:
    password = "".join((i[0], "7", i[1], "5-", i[2], "4", i[3], "3?"))
    # print(password)
    hash = sha1(password.encode("utf-8"))
    if check(hash):
        print(password)
        break
```

### 第四层：md5 相同文件不同

搜到一篇 [文章](http://www.programgo.com/article/89772288076/)，下载里面的两个程序，运行一下。

### 第五层：RSA

```
openssl rsa -pubin -in rsa_public_key.pem -text -modulus
```

看了下，e 很大，应该是 wienerattack，找到利用脚本。

```
'''
Created on Dec 14, 2011

@author: pablocelayes
'''

import ContinuedFractions, Arithmetic, RSAvulnerableKeyGenerator

def hack_RSA(e,n):
    '''
    Finds d knowing (e,n)
    applying the Wiener continued fraction attack
    '''
    frac = ContinuedFractions.rational_to_contfrac(e, n)
    convergents = ContinuedFractions.convergents_from_contfrac(frac)

    for (k,d) in convergents:

        #check if d is actually the key
        if k!=0 and (e*d-1)%k == 0:
            phi = (e*d-1)//k
            s = n - phi + 1
            # check if the equation x^2 - s*x + n = 0
            # has integer roots
            discr = s*s - 4*n
            if(discr>=0):
                t = Arithmetic.is_perfect_square(discr)
                if t!=-1 and (s+t)%2==0:
                    print("Hacked!")
                    return d

# TEST functions

def test_hack_RSA():
    n = 460657813884289609896372056585544172485318117026246263899744329237492701820627219556007788200590119136173895989001382151536006853823326382892363143604314518686388786002989248800814861248595075326277099645338694977097459168530898776007293695728101976069423971696524237755227187061418202849911479124793990722597
    e = 354611102441307572056572181827925899198345350228753730931089393275463916544456626894245415096107834465778409532373187125318554614722599301791528916212839368121066035541008808261534500586023652767712271625785204280964688004680328300124849680477105302519377370092578107827116821391826210972320377614967547827619
    d = hack_RSA(e, n)
    print "d=" + str(d)

if __name__ == "__main__":
    #test_is_perfect_square()
    #print("-------------------------")
    test_hack_RSA()
```

算出 d 后生成私钥解密。

[ctf](https://www.40huo.cn/tag/ctf/)