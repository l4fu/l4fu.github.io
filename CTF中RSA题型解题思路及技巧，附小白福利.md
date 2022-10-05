# CTF中RSA题型解题思路及技巧，附小白福利
## 0x01 RSA算法简介

**为了方便小白咀嚼后文，这里先对RSA密钥体制做个简略介绍（简略因为这不是本文讨论的重点）**

> 1. 选择两个大素数p和q，计算出模数N = p * q
>     
> 2. 计算φ = (p−1) * (q−1) 即N的欧拉函数，然后选择一个e (1<e<φ)，且e和φ互质
>     
> 3. 取e的模反数为d，计算方法: e * d ≡ 1 (mod φ)
>     
> 4. 对明文m进行加密：c = pow(m, e, N)，得到的c即为密文
>     
> 5. 对密文c进行解密，m = pow(c, d, N)，得到的m即为明文
>     

整理一下得到我们需要认识和记住的参数

> - p 和 q ：大整数N的两个因子（factor）
>     
> - N：大整数N，我们称之为模数（modulus
>     
> - e 和 d：互为模反数的两个指数（exponent）
>     
> - c 和 m：分别是密文和明文，这里一般指的是一个十进制的数
>     

然后我们一般称

> - （N，e）：公钥
>     
> - （N，d）：私钥
>     

## 0x02 CTF中的RSA题型

CTF中的RSA题目一般是将flag进行加密，然后把密文（即c）和其他一些你解题需要的信息一起给你，你需要克服重重难关，去解密密文c，得到flag（即m），一般有下列题型

### **公钥加密文**

这是CTF中最常见最基础的题型，出题人会给你一个公钥文件（通常是以.pem或.pub结尾的文件）和密文（通常叫做flag.enc之类的），你需要分析公钥，提取出（N，e），通过各种攻击手段恢复私钥，然后去解密密文得到flag。

### **文本文档**

对于第一种题型，耿直点的出题人直接给你一个txt文本文档，里面直接写出了（N，e，c）所对应的十进制数值，然后你直接拿去用就行了。当然也不都是给出（N，e，c）的值，有时还会给出其他一些参数，这时就需要思考，这题具体考察的什么攻击方法

### **pcap文件**

有时出题人会给你一个流量包，你需要用wireshark等工具分析，然后根据流量包的通信信息，分析题目考察的攻击方法，你可以提取出所有你解题需要用到的参数，然后进行解密

### **本地脚本分析**

题目会给你一个脚本和一段密文，一般为python编写，你需要逆向文件流程，分析脚本的加密过程，写出对应的解密脚本进行解密

### **远程脚本利用**

这种题型一般难度较大。题目会给你一个运行在远程服务器上的python脚本和服务器地址，你需要分析脚本存在的漏洞，确定攻击算法，然后编写脚本与服务器交互，得到flag

## 0x03 RSA的常见攻击方法

> 看不懂的别着急，直接跳过看后面的小白福利环节

### 当模数N过小时

RSA的非对称体制是建立在大整数分解难题上的，所以最基本的攻击方法就是当模数N过小时，我们可以写个脚本直接爆破他的因子，如

```
N = 1211809

for i in range(2, N):

    if N % i == 0:

        print i

```

那么靠爆破来分解大整数N，我们可以分解多大的呢？

> 2009年12月12日，编号为 RSA-768 （768bits,232 digits）数也被成功分解。---百度百科

然而现在一般RSA在实际应用里都是2048位的，在CTF中出现的也不会太小，一般是不可能让你爆破分解的，都是要用到一些攻击算法的，下面我来介绍下这些算法

### 分解大整数的一些算法

如果说N小了容易被分解，那么N越大就越安全吗？不然，RSA密钥的安全不只和模数N有关，与它的指数：e和d也息息相关

这里假设我们从题目获得了公钥（N，e）和待解密的密文c，由RSA的加解密过程，我们知道，如果要解密密文，我们要得到e的模反数d，而d是要我们去求解的。

这里讨论我们如何知道什么时候该用什么算法，不进行数学证明及原理分析。下面我例举其中几个比较常见且容易理解的，因为作为最后的小白福利，我已经把这些算法都集成进去了，感兴趣的可以查看源码

### **Wiener's attack**

[https://en.wikipedia.org/wiki/Wiener%27s_attack](https://en.wikipedia.org/wiki/Wiener%27s_attack)

当 d < (1/3) N^(1/4)时，我们可以通过Wiener's attack分解得到d

**费马分解**

当大整数N的两个因子p和q相近时，我们可以通过费马分解的办法很快分解大整数

**Small q**

当q较小时，即|p-q|较大时，我们可以直接爆破因子

**Boneh Durfee Method**

当d满足 d ≤ N^0.292 时，我们可以利用该方法分解N，理论上比wiener attack要强一些。

**yafu**

yafu使用最强大的现代算法去自动化的分解输入的整数。大多数的算法都实现了多线程，让yafu能充分利用多核处理器（算法包括 SNFS, GNFS, SIQS, 和 ECM）。

**factordb**

如果对一个大整数用一些特殊算法也分解不了的时候，我们可以在 [http://factordb.com/](http://factordb.com/) 中查询一下数据库，说不定就能找到其因子

### 其他一些题型

有些题会给你一些随机生成的大整数，一般来说是无法直接分解的，不过一般会给我们其他切入点

**Rabin算法**

当e等于2时

**小公钥指数攻击**

当e十分小时，比如e等于3时

**d泄露攻击**

如果我们知道一组过期的（N，e1，d1）和一组由新的e2组成的公钥及其加密的密文（N，e2，c），我们可以由（e1，d1）得到模数N的两个因子p和q，再去算e2的模反数d2，去解密密文

**共模攻击**

使用相同的模数 N 、不同的私钥，加密同一明文消息

**模不互素**

两个公钥的N不互素时

**Known High Bits Factor Attack**

我们知道模数N其中一个因子的高比特位时

**Stereotyped messages**

如果你知道明文中最重要的部分，您可以使用此方法找到消息的其余部分。

**Basic Broadcast Attack**

同一个加密指数e和不同且互素的模数N加密了同一个密文，并发送给了其他e个用户

## 0x04 小白福利环节

上面有一堆让人头大的算法，比如分解一个大整数可能就有十来种算法，当然不是每个算法都能成功分解我们题目给的大整数的，如果我们挨个尝试，不仅浪费精力还浪费时间，等你找到正确的算法，已经与一血，二血，三血无缘了。所以我们需要一个自动化的工具，来帮助我们尝试可能的算法，并恢复出私钥或者解密密文。于是我就用几天时间写了这么一款小工具。

### 项目地址

[https://github.com/D001UM3/CTF-RSA-tool](https://github.com/D001UM3/CTF-RSA-tool)

### 环境依赖

**安装libnum**

```
git clone https://github.com/hellman/libnum.git

cd libnum

python setup.py install

```

**安装gmpy2**

参考原文：[https://www.cnblogs.com/pcat/p/5746821.html](https://www.cnblogs.com/pcat/p/5746821.html)

原文里面有的版本过老，会安装失败，可以参考我的安装过程：[https://d001um3.github.io/2018/01/24/CTF-RSA-tool-install/](https://d001um3.github.io/2018/01/24/CTF-RSA-tool-install/)

**克隆仓库，安装依赖**

```
git clone https://github.com/D001UM3/CTF-RSA-tool.git

cd CTF-RSA-tool

pip install -r "requirements.txt"

```

**安装sagemath（可选）**

> 安装sagemath的以支持更多的算法，提高解题成功率，嫌麻烦也可以不安装

官网：[http://www.sagemath.org](http://www.sagemath.org)

我的安装过程：[https://d001um3.github.io/2017/12/06/sage/](https://d001um3.github.io/2017/12/06/sage/)

### 几个解题例子（更多参考项目目录下example.txt）

在大多数情况下，你只需要把题目给的信息输入给脚本，脚本就会自动完成剩下的工作如：

1：题目给了一个公钥文件和密文，直接用 `--key` 或 `-k` 指定公钥，用 --decrypt 指定密文文件就行了

2：题目给了你如（N，e，c）的十进制值，分别通过`-N`，`-e`，`-c`输入就行了

3：上面那种情况，如果题目是把这些参数都写入一个文本文件，如txt中，直接用 `--input` 或 `-i` 指定文本文件就行了

具体的例子：

**Wiener's attack**

`python solve.py --verbose --private -i examples/wiener_attack.txt`

或者通过命令行，只要指定对应参数就行了

`python solve.py  --verbose --private -N 460657813884289609896372056585544172485318117026246263899744329237492701820627219556007788200590119136173895989001382151536006853823326382892363143604314518686388786002989248800814861248595075326277099645338694977097459168530898776007293695728101976069423971696524237755227187061418202849911479124793990722597 -e 354611102441307572056572181827925899198345350228753730931089393275463916544456626894245415096107834465778409532373187125318554614722599301791528916212839368121066035541008808261534500586023652767712271625785204280964688004680328300124849680477105302519377370092578107827116821391826210972320377614967547827619`

**利用 factordb.com 分解大整数**

`python solve.py --verbose  -k examples/jarvis_oj_mediumRSA/pubkey.pem --decrypt examples/jarvis_oj_mediumRSA/flag.enc`

**small q attack**

`python solve.py --verbose --private -k examples/small_q.pub`

**费马分解（p&q相近时）**

`python solve.py --verbose --private -i examples/closed_p_q.txt`

**Common factor between ciphertext and modulus attack（密文与模数不互素）**

`python solve.py --verbose -k examples/common_factor.pub --decrypt examples/common_factor.cipher --private`

**small e**

`python solve.py --verbose -k examples/small_exponent.pub  --decrypt examples/small_exponent.cipher`

**Rabin 算法 （e == 2）**

`python solve.py --verbose -k examples/jarvis_oj_hardRSA/pubkey.pem --decrypt examples/jarvis_oj_hardRSA/flag.enc`

**Small fractions method when p/q is close to a small fraction**

`python solve.py --verbose -k examples/smallfraction.pub  --private`

**Known High Bits Factor Attack**

`python solve.py --verbose --private -i examples/KnownHighBitsFactorAttack.txt`

**d泄漏攻击**

`python solve.py --verbose --private -i examples/d_leak.txt`

**模不互素**

`python solve.py --verbose --private -i examples/share_factor.txt`

**共模攻击**

`python solve.py --verbose --private -i examples/share_N.txt`

**Basic Broadcast Attack**

`python solve.py --verbose --private -i examples/Basic_Broadcast_Attack.txt`

### 再列举几个实用的小功能

**输入N与e创建公钥**

`python solve.py -g --createpub  -N your_modulus -e your_public_exponent -o public.pem`

**查看密钥文件**

`python solve.py -g --dumpkey --key examples/smallfraction.pub`

**将加密文件转为十进制（方便写入文本，配合`-i`需要）**

`python solve.py -g --enc2dec examples/jarvis_oj_hardRSA/flag.enc`

**下面来介绍下我写这个工具的思路**

### 这个工具如何工作

根据题目给的参数类型，自动判断应该采用哪种攻击方法，并尝试得到私钥或者明文，从而帮助CTFer快速拿到flag或解决其中的RSA考点

### 大体思路

**判断输入**

首先，识别用户的输入，可以是证书 pem 文件，也可以通过命令行参数指定`n`，`e`等变量的值，甚至可以通过命令行指定题目所给的txt文件并自动识别里面的变量

**判断攻击算法**

根据取到的参数类型及数量，选取可能成功的方法并采用一定的优先级逐个尝试。

如大整数分解的题型：给了一个公钥和一个加密的密文，我们需要先分解大整数N，然后得到私钥再去解密。考点在于大整数分解，脚本的关键代码在CTF-RSA-tool/lib/factor_N.py中的solve函数

```
def solve(N, e, c, sageworks):

    if sageworks:

        return pastctfprimes(N) or noveltyprimes(N) or wiener_attack(N, e) or factordb(N) or comfact_cn(N, c) or smallq(N) or p_q_2_close(N) or boneh_durfee(N, e) or smallfraction(N) or None

    else:

        return pastctfprimes(N) or noveltyprimes(N) or wiener_attack(N, e) or factordb(N) or comfact_cn(N, c) or smallq(N) or p_q_2_close(N) or None

```

**选择输出**

CTFer可以通过命令行选择是输出私钥还是输出解密后的密文，还是一起输出，不过非 `--input`（文本文档自动识别攻击） 的情况下，请至少选择 `--private`（打印得到的私钥） 或 `--decrypt`（解密一个加密的文件） 或 `--decrypt_int`（解密一个十进制数） 中的一个。

## 0x05.还是没有得到flag

首先如果题型是大整数分解的话，你还可以尝试使用其他工具如 **yafu** 来分解，如果还是不能分解，你就得再好好看看这题是不是另有切入点了。

其次，如果是一些你没见过的题型的话，建议通过百度，谷歌，github搜一下，一般还是能搜到的类似的题目甚至原题的

最后，如果还是做不出来，先女装一下再去做题，听大佬们说有buff加成

## 0x06.写在最后

> 1：大部分算法都是参照网上，我只是做了下集成汇总，并尽量使其易懂易用，代码不精，大佬轻喷。
> 
> 2：还有很多实用的算法（有关 Coppersmith 的算法和一些分解大整数的算法）暂时没有加进去，一是精力有限，二是能力有限，三是没遇到相关题型，以后会不断完善。
> 
> 3：大佬们觉得好用的话，点个 star 收藏一下，也欢迎各位来 contribute 和提 issues