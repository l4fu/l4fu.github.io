# RSA史上最强剖析,附常用工具使用方法及CTF中RSA典型例题
**RSA加解密类题型是ctf题中常见题型，考点比较广泛，涉及各种攻击手法，以前在这栽了不少跟头，这里好好总结一下。包括RSA加密原理，RSA常用工具使用方法及下载地址，RSA典型例题。**  

## RSA加密基本原理

### 加密过程

选择两个大素数p和q，计算出模数N = p * q

计算φ = (p−1) * (q−1) 即N的欧拉函数，然后选择一个e (1<e<φ)，且e和φ互质

取e的模反数为d，计算方法: e * d ≡ 1 (mod φ)

对明文A进行加密：`B≡A^e (mod n) 或 B = pow(A,e,n)`，得到的B即为密文

对密文B进行解密，`A≡B^d( mod n) 或 A = pow(B,d,n)`，得到的A即为明文

> p 和 q ：大整数N的两个因子（factor）
> 
> N：大整数N，我们称之为模数（modulus）
> 
> e 和 d：互为模反数的两个指数（exponent）
> 
> c 和 m：分别是密文和明文，这里一般指的是一个十进制的数

一般有如下称呼：

> （N，e）：公钥
> 
> （N，d）：私钥

### 加密分析

RSA算法是一种非对称密码算法，所谓非对称，就是指该算法需要一对密钥，使用其中一个加密，则需要用另一个才能解密。

RSA的算法涉及三个参数，n、e、d。

其中，n是两个大质数p、q的积，n以二进制表示时所占用的位数，就是所谓的密钥长度。

e和d是一对相关的值，e可以任意取，但要求e与(p-1)(q-1)互质；再选择d，要求`(ed) ≡ 1(mod(p-1)×(q-1))`。

令φ = (p-1)(q-1) 上式即 `d*e = 1 mod φ` 即`：(d*e - 1)% φ = 0`

（n，e),(n，d)就是密钥对。其中(n，e)为公钥，(n，d)为私钥。RSA加解密的算法完全相同，设A为明文，B为密文，则：A≡B^d( mod n)；B≡A^e (mod n)；（公钥加密体制中，一般用公钥加密，私钥解密）

e和d可以互换使用，即：

A≡B^e (mod n)；B≡A^d( mod n);

### 附加解释

同余符号 “ ≡ ”

数论中的重要概念。给定一个正整数m，如果两个整数a和b满足（a-b）能够被m整除，即（a-b）/m得到一个整数，那么就称整数a与b对模m同余，记作a≡b(mod m)。对模m同余是整数的一个等价关系

比如26≡14(mod 12)

互质

公约数只有1的两个数，称为互质

python pow() 函数

```
x^y = pow(x,y)

x^y  mod  z = pow(x,y,z)

```

## RSA工具

### RSA-Tool 2使用方法

**软件参数**

> P= 第一个大素数Q= 第二个大素数 (P和Q的长度不能相差太大！) 
> 
> E= 公钥 (一个随机数，必须满足：GCD(E,(P-1)*(Q-1))==1)(译者注：即E和(p-1)(Q-1)互素) 
> 
> N= 公用模数，由P和Q生成：N=P*Q
> 
> D= 私钥：D=E^(-1) mod ((P-1)*(Q-1))

参数N和E是公开的但是D是私有的并且绝不能公开！P和Q在生成密钥后便不再需要了，但是必须销毁。

为了从公钥(N,E)得到D,需要试图分解N为它的两个素数因子。对于一个很大的模数N（512位或更大）要想分解出它的P和Q是件非常困难的事。

**生成一组RSA密钥对**

> 按下'Start'按钮，通过移动你的鼠标指针来收集一些随机数据.这必须一次完成，因为收集的数据会被保存在你的RSA-Tool文件夹里面的一个文件中。 选择要创建的密钥的长度(等于N的长度)。最大为4096位. 选择你的公钥(E)并把它输入到相应的编辑框作为十进制数。常用的E有（考虑到计算速度的原因):3,17,257和65537(十进制). 按下'Generate'，等到密钥生成完成。

注意，生成很大的数需要一些时间，取决于你的CPU的计算能力。

特别说明：你可以常按'Generate'.做为密钥生成过程的一个组成部分的内置随机数生成系统会在运行的时候重新进行初始化。这是故意这么做的，这样可使那些滥用这个工具做其它事情变得更困难...　　 注意两次或两次以上生成相同的密钥对是不可能的。

**分解一个数**

选择正确的进制 在Modules(N)编辑框中输入或复制这个数。这会激活'Factor'按钮。 然后按Factor按钮.注意分解大于240位的数会花很大的内存和时间！

甚至很小的数也会需要几个小时。

如果用Multiple Polynomial Quadratic Sieve(MPQS)算法来分解整数,需要大量的内存.原因是这个算法的设计,而不是编码风格。

例如：分解一个给定大小的N的内存使用情况(使用MPQS算法)...

256位~89MB, 280 位~140MB, 296位~185MB等等.

**由素数因子P和Q计算私钥D**

选择参数P和Q的正确进制，在相应的文本区域中输入或粘贴P和Q 按下'Calc.D'，得到整数的精确位长度

为你要进行检查的数选择正确的进制 在Modules(N)文本框中输入或粘贴整数 按小的'Exact size'按钮.会显示出这个数使用的精确的位数

### msieve使用方法

msieve是一个大数分解工具 [下载地址](http://gilchrist.ca/jeff/factoring/index.html)

msieve.exe --help 查看帮助 l filename 保存日志到filename文件中，默认为msieve.log i filename 从filename文件中读取数字，默认worktodo.ini one_number: 待分解数字，0开头代表8进制，0x开头代表16进制，否则为10进制。如不填，则从worktodo.ini中读取数字。 -v 意思打印具体分解的情况 -q 仅仅打印能找到的因子

各自运行结果如下：

```
E:\ctf工具\ctf工具\加解密工具\RSA专用工具\msieve152_svn939_win64_i7>msieve.exe 0xA41006DEFD378B7395B4E2EB1EC9BF56A61CD9C3B5A0A73528521EEB2FB817A7 -v

Msieve v. 1.52 (SVN 939)
Tue Feb 27 14:16:37 2018
random seeds: 4700eb70 a74b9797
factoring 74207624142945242263057035287110983967646020057307828709587969646701361764263 (77 digits)
searching for 15-digit factors
commencing quadratic sieve (77-digit input)
using multiplier of 7
using VC8 32kb sieve core
sieve interval: 12 blocks of size 32768
processing polynomials in batches of 17
using a sieve bound of 921409 (36471 primes)
using large prime bound of 92140900 (26 bits)
using trial factoring cutoff of 26 bits
polynomial 'A' values have 10 factors
restarting with 19759 full and 186804 partial relations

36609 relations (19759 full + 16850 combined from 186804 partial), need 36567
sieving complete, commencing postprocessing
begin with 206563 relations
reduce to 51387 relations in 2 passes
attempting to read 51387 relations
recovered 51387 relations
recovered 38239 polynomials
attempting to build 36609 cycles
found 36609 cycles in 1 passes
distribution of cycle lengths:
   length 1 : 19759
   length 2 : 16850
largest cycle: 2 relations
matrix is 36471 x 36609 (5.3 MB) with weight 1094350 (29.89/col)
sparse part has weight 1094350 (29.89/col)
filtering completed in 4 passes
matrix is 24957 x 25021 (4.0 MB) with weight 837221 (33.46/col)
sparse part has weight 837221 (33.46/col)
saving the first 48 matrix rows for later
matrix includes 64 packed rows
matrix is 24909 x 25021 (2.6 MB) with weight 614783 (24.57/col)
sparse part has weight 438151 (17.51/col)
commencing Lanczos iteration
memory use: 2.6 MB
lanczos halted after 395 iterations (dim = 24905)
recovered 16 nontrivial dependencies
prp39 factor: 258631601377848992211685134376492365269
prp39 factor: 286924040788547268861394901519826758027
elapsed time 00:00:05

E:\ctf工具\ctf工具\加解密工具\RSA专用工具\msieve152_svn939_win64_i7>msieve.exe 0xA41006DEFD378B7395B4E2EB1EC9BF56A61CD9C3B5A0A73528521EEB2FB817A7 -q

0xA41006DEFD378B7395B4E2EB1EC9BF56A61CD9C3B5A0A73528521EEB2FB817A7
prp39: 258631601377848992211685134376492365269
prp39: 286924040788547268861394901519826758027

```

注：prp39 即是分解出来的p,q

注意前面加上相应进制。八进制0 十六进制0x 十进制什么也不加。

**openssl**

生成私钥，并导出公钥生成2048 bit的PEM格式的RSA Key：Key.pem

```
徐超@yoona MINGW64 ~/Desktop/test
$ openssl genrsa -out key.pem -f4 2048
Generating RSA private key, 2048 bit long modulus
................+++
..............................+++
e is 65537 (0x10001)

```

从私钥导出公钥：Key_public.pem

```
徐超@yoona MINGW64 ~/Desktop/test
$ openssl rsa -in key.pem -pubout -out key_public.pem
writing RSA key

```

**准备测试数据**

为了简便起见，这里将字符串”hello rsa”存放到文件msg.txt作为测试数据：

```
徐超@yoona MINGW64 ~/Desktop/test
$ echo "hello rsa" > msg.txt

```

**公钥加密**

使用公钥key_public.pem对测试数据msg.txt进行加密生成msg.txt.enc，并查看加密后的数据：

```
徐超@yoona MINGW64 ~/Desktop/test
$ openssl rsautl -in msg.txt -out msg.txt.enc -inkey key_public.pem -pubin -encrypt -pkcs

徐超@yoona MINGW64 ~/Desktop/test
$ cat msg.txt.enc
Ff▒▒li▒Yn▒HA▒cǴ▒Ԋ7▒V4▒
                      KS콳h)▒▒w)▒▒{kv▒+ ▒h▒▒qi?Ǭ▒▒f▒▒T.▒▒▒▒ߍ▒.▒▒▒&▒;Z▒▒▒▒▒▒$y▒▒)n_▒8▒0▒")[▒w▒▒]~)r▒▒'▒▒u▒▒6g▒*F▒F▒'ٲ▒▒]   +G▒m▒j%k*▒▒s&
                             ▒▒JQ~n▒▒▒.هsi▒▒}~▒▒f ▒Eӝ▒}▒7C▒▒▒Z▒Z|>▒]A▒▒▒ǐ▒
▒▒▒▒e▒Uc▒▒B▒▒▒(dh▒üj▒՗▒vĊ

```

这里使用：

> -in 选项指定原始数据文件msg.bin -out 选项指定加密后的输出文件msg.bin.enc -inkey 选项指定用于加密的公钥Key_pub.pem，由于输入是公钥，所以需要使用选项-pubin来指出 -encrypt 选项表明这里是进行加密操作 -pkcs 选项指定加密处理过程中数据的填充方式，对于填充，可选项有：-pkcs, -oaep, -ssl, -raw，默认是-pkcs，即按照PKCS#1 v1.5规范进行填充

**私钥解密**

使用私钥key.pem对加密后的数据msg.txt.enc进行解密，并将结果存放到msg.txt.dec文件中：

```
徐超@yoona MINGW64 ~/Desktop/test
$ openssl rsautl -in msg.txt.enc -out msg.txt.dec -inkey key.pem -decrypt -pkcs

徐超@yoona MINGW64 ~/Desktop/test
$ cat msg.txt.dec
hello rsa

```

这里使用：

> -in 选项指定待解密的数据文件msg.bin.enc -out 选项指定解密后的输出文件msg.bin.dec -inkey 选项指定用于解密的私钥Key.pem，由于输入是私钥，所以不再需要使用选项-pubin -decrypt 选项表明这里是进行解密操作 -pkcs 选项指定解密处理过程中数据的填充方式，对于填充，可选项有：-pkcs, -oaep, -ssl, -raw，默认是-pkcs，即按照PKCS#1 v1.5规范进行填充

### yafu

主要用来分解N，命令为factor(N)

先运行文件夹下的yafu-x64.exe进入命令行

执行factor(N)

执行过程：（也可以像下面这样后面直接加上factor(N)）

```
E:\ctf工具\ctf工具\加解密工具\RSA专用工具\yafu>yafu-x64.exe factor(920139713)

fac: factoring 920139713
fac: using pretesting plan: normal
fac: no tune info: using qs/gnfs crossover of 95 digits
div: primes less than 10000
fmt: 1000000 iterations
Total factoring time = 0.0080 seconds

***factors found***

P5 = 49891
P5 = 18443

ans = 1

```

mismatched parens报错

这是因为N的位数过长，命令行不支持。

把N值保存到文件中，如rsa.txt ，然后执行

```
yafu-x64.exe "factor(@)" -batchfile rsa.txt

```

执行后rsa.txt就会被自动删除。

执行过程如下：

```
E:\ctf工具\ctf工具\加解密工具\RSA专用工具\yafu>yafu-x64.exe "factor(@)" -batchfile rsa.txt

=== Starting work on batchfile expression ===
factor(966808932627497190635859236054960349099463975227350564265384373280336699853387254070662881265937565163000758606154308757944030571837175048514574473061401566330836334647176655282619268592560172726526643074499534129878217409046045533656897050117438496357231575999185527675071002803951800635220029015932007465117818739948903750200830856115668691007706836952244842719419452946259275251773298338162389930518838272704908887016474007051397194588396039111216708866214614779627566959335170676055025850932631053641576566165694121420546081043285806783239296799795655191121966377590175780618944910532816988143056757054052679968538901460893571204904394975714081055455240523895653305315517745729334114549756695334171142876080477105070409544777981602152762154610738540163796164295222810243309051503090866674634440359226192530724635477051576515179864461174911975667162597286769079380660782647952944808596310476973939156187472076952935728249061137481887589103973591082872988641958270285169650803792395556363304056290077801453980822097583574309682935697260204862756923865556397686696854239564541407185709940107806536773160263764483443859425726953142964148216209968437587044617613518058779287167853349364533716458676066734216877566181514607693882375533)
=============================================
fac: factoring 966808932627497190635859236054960349099463975227350564265384373280336699853387254070662881265937565163000758606154308757944030571837175048514574473061401566330836334647176655282619268592560172726526643074499534129878217409046045533656897050117438496357231575999185527675071002803951800635220029015932007465117818739948903750200830856115668691007706836952244842719419452946259275251773298338162389930518838272704908887016474007051397194588396039111216708866214614779627566959335170676055025850932631053641576566165694121420546081043285806783239296799795655191121966377590175780618944910532816988143056757054052679968538901460893571204904394975714081055455240523895653305315517745729334114549756695334171142876080477105070409544777981602152762154610738540163796164295222810243309051503090866674634440359226192530724635477051576515179864461174911975667162597286769079380660782647952944808596310476973939156187472076952935728249061137481887589103973591082872988641958270285169650803792395556363304056290077801453980822097583574309682935697260204862756923865556397686696854239564541407185709940107806536773160263764483443859425726953142964148216209968437587044617613518058779287167853349364533716458676066734216877566181514607693882375533
fac: using pretesting plan: normal
fac: no tune info: using qs/gnfs crossover of 95 digits
div: primes less than 10000
fmt: 1000000 iterations
Total factoring time = 0.4415 seconds

***factors found***

PRP617 = 31093551302922880999883020803665536616272147022877428745314830867519351013248914244880101094365815998050115415308439610066700139164376274980650005150267949853671653233491784289493988946869396093730966325659249796545878080119206283512342980854475734097108975670778836003822789405498941374798016753689377992355122774401780930185598458240894362246194248623911382284169677595864501475308194644140602272961699230282993020507668939980205079239221924230430230318076991507619960330144745307022538024878444458717587446601559546292026245318907293584609320115374632235270795633933755350928537598242214216674496409625928997877221
PRP617 = 31093551302922880999883020803665536616272147022877428745314830867519351013248914244880101094365815998050115415308439610066700139164376274980650005150267949853671653233491784289493988946869396093730966325659249796545878080119206283512342980854475734097108975670778836003822789405498941374798016753689377992355122774401780930185598458240894362246194248623911382284169677595864501475308194644140602272961699230282993020507668939980205079239221924230430230318076991507619960330144745307022538024878444458717587446601559546292026245318907293584609320115374632235270795633933755350928537598242214216674496409625928797450473

ans = 1

```

eof; done processing batchfile报错

rsa.txt用notepad++打开，最后加上换行即可。

## RSA题型分析

### 实验吧RSA

题目链接：[http://www.shiyanbar.com/ctf/1772](http://www.shiyanbar.com/ctf/1772)

openssl分析公钥，得到N,E

```
openssl rsa -pubin -text -modulus -in public.pem

```

运行结果

```
徐超@yoona MINGW64 ~/Desktop/RSA
$ openssl rsa -pubin -text -modulus -in public.pem
Public-Key: (256 bit)
Modulus:
    00:a4:10:06:de:fd:37:8b:73:95:b4:e2:eb:1e:c9:
    bf:56:a6:1c:d9:c3:b5:a0:a7:35:28:52:1e:eb:2f:
    b8:17:a7
Exponent: 65537 (0x10001)
Modulus=A41006DEFD378B7395B4E2EB1EC9BF56A61CD9C3B5A0A73528521EEB2FB817A7
-----BEGIN PUBLIC KEY-----
MDwwDQYJKoZIhvcNAQEBBQADKwAwKAIhAKQQBt79N4tzlbTi6x7Jv1amHNnDtaCn
NShSHusvuBenAgMBAAE=
-----END PUBLIC KEY-----
writing RSA key

```

Modulus 是n的值，Exponent是E的值。

msieve分解N的值

```
msieve.exe 0xA41006DEFD378B7395B4E2EB1EC9BF56A61CD9C3B5A0A73528521EEB2FB817A7 -v

```

运行结果

```
E:\ctf工具\ctf工具\加解密工具\RSA专用工具\msieve153>msieve153.exe 0xA41006DEFD378B7395B4E2EB1EC9BF56A61CD9C3B5A0A73528521EEB2FB817A7 -v

Msieve v. 1.53 (SVN 1005)
Wed Feb 28 23:00:33 2018
random seeds: f922db80 d61d4d18
factoring 74207624142945242263057035287110983967646020057307828709587969646701361764263 (77 digits)
searching for 15-digit factors
commencing quadratic sieve (77-digit input)
using multiplier of 7
using generic 32kb sieve core
sieve interval: 12 blocks of size 32768
processing polynomials in batches of 17
using a sieve bound of 921409 (36471 primes)
using large prime bound of 92140900 (26 bits)
using trial factoring cutoff of 26 bits
polynomial 'A' values have 10 factors
restarting with 19771 full and 188452 partial relations

36803 relations (19771 full + 17032 combined from 188452 partial), need 36567
sieving complete, commencing postprocessing
begin with 208223 relations
reduce to 51729 relations in 2 passes
attempting to read 51729 relations
recovered 51729 relations
recovered 38607 polynomials
attempting to build 36803 cycles
found 36803 cycles in 1 passes
distribution of cycle lengths:
   length 1 : 19771
   length 2 : 17032
largest cycle: 2 relations
matrix is 36471 x 36803 (5.3 MB) with weight 1102159 (29.95/col)
sparse part has weight 1102159 (29.95/col)
filtering completed in 3 passes
matrix is 24858 x 24922 (4.0 MB) with weight 836197 (33.55/col)
sparse part has weight 836197 (33.55/col)
saving the first 48 matrix rows for later
matrix includes 64 packed rows
matrix is 24810 x 24922 (2.6 MB) with weight 611256 (24.53/col)
sparse part has weight 439905 (17.65/col)
commencing Lanczos iteration
memory use: 2.7 MB
lanczos halted after 394 iterations (dim = 24805)
recovered 14 nontrivial dependencies
p39 factor: 258631601377848992211685134376492365269
p39 factor: 286924040788547268861394901519826758027
elapsed time 00:00:06

```

知道p，q，e的值，python脚本生成私钥

```
import math
import sys
from Crypto.PublicKey import RSA

keypair = RSA.generate(1024)

keypair.p = 0xD7DB8F68BCEC6D7684B37201385D298B
keypair.q = 0xC292A272E8339B145D9DF674B9A875D5
keypair.e = 65537

keypair.n = keypair.p * keypair.q
Qn = long((keypair.p-1) * (keypair.q-1))

i = 1
while (True):
    x = (Qn * i ) + 1
    if (x % keypair.e == 0):
        keypair.d = x / keypair.e
        break
    i += 1

private = open('private.pem','w')
private.write(keypair.exportKey())
private.close()

```

注：生成的私钥会默认base64加密。

密钥解密密文

```
➜  RSA openssl rsautl -decrypt -in flag.enc -inkey private.pem -out flag.txt
➜  RSA cat flag.txt 
ISG{256bit_is_weak}

```

### RSA roll

题目给出一个txt文件，内容如下：

```
{920139713,19}

704796792
752211152
274704164
18414022
368270835
483295235
263072905
459788476
483295235
459788476
663551792
475206804
459788476
428313374
475206804
459788476
425392137
704796792
458265677
341524652
483295235
534149509
425392137
428313374
425392137
341524652
458265677
263072905
483295235
828509797
341524652
425392137
475206804
428313374
483295235
475206804
459788476
306220148

```

可知 N = 920139713 E = 19

**方法一 RSA-tool 2**

分解N得到p,q 选对进制，生成D(私钥) 点击test用私钥解密密文

**方法二 python脚本**

分解N得到p,q

```
n = 2
while (n<920139713): 
    if (920139713%n == 0):
       print n,920139713/n
    n = n + 1

```

解出私钥d

```
p = int(input("p:"))
q = int(input("q:"))
e = int(input("e:"))
#print(type(e))
L = (p-1)*(q-1)
i = 0
while True:
    if (1+L*i)%e == 0:
        break
    i+=1
    #print(i)

print("d is %s"%((1+L*i)/e))

```

私钥 d 解密密文(密文保存为rsa.txt并去掉开头两行)

```
n = 920139713
d = 96849619
result = []
with open("rsa.txt") as f:
    for i in f:
        result.append(chr(pow(int(i),d,n)))

print(result)

```

运行结果

```
C:\Python27\python.exe F:/code/learning/ctf/rsa_py/rsa_1.py
['f', 'l', 'a', 'g', '{', '1', '3', '2', '1', '2', 'j', 'e', '2', 'u', 'e', '2', '8', 'f', 'y', '7', '1', 'w', '8', 'u', '8', '7', 'y', '3', '1', 'r', '7', '8', 'e', 'u', '1', 'e', '2', '}']

进程已结束,退出代码0

```

得到flag为flag{13212je2ue28fy71w8u87y31r78eu1e2}

## 常用工具下载地址

RSA-tool 2 [http://www.skycn.net/soft/appid/39911.html](http://www.skycn.net/soft/appid/39911.html)

msieve [https://sourceforge.net/projects/msieve/](https://sourceforge.net/projects/msieve/)

yafu [https://sourceforge.net/projects/yafu/](https://sourceforge.net/projects/yafu/)

后续遇到相应题型还会在我的博客补充[www.ixuchao.cn](http://www.ixuchao.cn)

联系方式：

**qq：**755563428

**email：**755563428@qq.com

phone:15615833854