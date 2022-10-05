# RSA 算法科普及简单 Python 实现
最近做几个 ctf 的 crypto 题，好几道 RSA 都没法用以前无脑工具破搞定了，所以深入研究了一下原理和一些小 trick，把自己的一点体会分享出来，共同学习。

# 1 算法基本思路

RSA 算法是一种非对称加密算法，是现在广泛使用的公钥加密算法，主要应用是加密信息和数字签名。详情请看[维基](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95)

## 1.1 公钥与私钥的生成

1.  随机挑选两个大质数 p 和 q，构造 `N = p*q`；
2.  计算欧拉函数 `φ(N) = (p-1) * (q-1)`；
3.  随机挑选 e，使得 `gcd(e, φ(N)) = 1`，即 e 与 φ(N) 互素；
4.  计算 d，使得 `e*d ≡ 1 (mod φ(N))`，即 d 是 e 的乘法逆元。

此时，公钥为（e, N），私钥为（d, N），公钥公开，私钥自己保管。

## 1.2 加密信息

1.  待加密信息（明文）为 M，M < N；（因为要做模运算，若 M 大于 N，则后面的运算不会成立，因此当信息比 N 要大时，应该分块加密）
2.  密文 `C = Me mod N`
3.  解密 `Cd mod N = (Me)d mod N = Md*e mod N` ；

要理解为什么能解密？要用到欧拉定理（其实是费马小定理的推广）

```
aφ(n) ≡ 1 (mod n)
```

再推广：

```
aφ(n)*k ≡ 1 (mod n)
```

得：

```
aφ(n)*k+1 ≡ a (mod n)
```

注意到 `e*d ≡ 1 mod φ(N)`，即：`e*d = 1 + k*φ(N)`。  
因此，`Md*e mod N = M1 + k*φ(N) mod N = M`  
简单来说，别人用我的公钥加密信息发给我，然后我用私钥解密。

## 1.3 数字签名：

1.  密文 `C = Md mod N`
2.  解密 `M = Ce mod N = (Md)e mod N = Md*e mod N = M`；（原理同上）

简单来说，我用自己的密钥加密签名，别人用我的公钥解密可以看到这是我的签名。注意，这个不具有隐私性，即任何人都可以解密此签名。

算法的安全性：基于大整数 N 难以分解出 p 和 q，构造 φ(N)；或由 N 直接构造 φ(N) 同样难。

# 2 Python 实现基本原理

```
import random

def fastExpMod(b, e, m):
    """
    e = e0*(2^0) + e1*(2^1) + e2*(2^2) + ... + en * (2^n)

    b^e = b^(e0*(2^0) + e1*(2^1) + e2*(2^2) + ... + en * (2^n))
        = b^(e0*(2^0)) * b^(e1*(2^1)) * b^(e2*(2^2)) * ... * b^(en*(2^n))

    b^e mod m = ((b^(e0*(2^0)) mod m) * (b^(e1*(2^1)) mod m) * (b^(e2*(2^2)) mod m) * ... * (b^(en*(2^n)) mod m) mod m
    """
    result = 1
    while e != 0:
        if (e&1) == 1:
            # ei = 1, then mul
            result = (result * b) % m
        e >>= 1
        # b, b^2, b^4, b^8, ... , b^(2^n)
        b = (b*b) % m
    return result

def primeTest(n):
    q = n - 1
    k = 0
    #Find k, q, satisfied 2^k * q = n - 1
    while q % 2 == 0:
        k += 1;
        q /= 2
    a = random.randint(2, n-2);
    #If a^q mod n= 1, n maybe is a prime number
    if fastExpMod(a, q, n) == 1:
        return "inconclusive"
    #If there exists j satisfy a ^ ((2 ^ j) * q) mod n == n-1, n maybe is a prime number
    for j in range(0, k):
        if fastExpMod(a, (2**j)*q, n) == n - 1:
            return "inconclusive"
    #a is not a prime number
    return "composite"

def findPrime(halfkeyLength):
    while True:
        #Select a random number n
        n = random.randint(0, 1<<halfkeyLength)
        if n % 2 != 0:
            found = True
            #If n satisfy primeTest 10 times, then n should be a prime number
            for i in range(0, 10):
                if primeTest(n) == "composite":
                    found = False
                    break
            if found:
                return n

def extendedGCD(a, b):
    #a*xi + b*yi = ri
    if b == 0:
        return (1, 0, a)
    #a*x1 + b*y1 = a
    x1 = 1
    y1 = 0
    #a*x2 + b*y2 = b
    x2 = 0
    y2 = 1
    while b != 0:
        q = a / b
        #ri = r(i-2) % r(i-1)
        r = a % b
        a = b
        b = r
        #xi = x(i-2) - q*x(i-1)
        x = x1 - q*x2
        x1 = x2
        x2 = x
        #yi = y(i-2) - q*y(i-1)
        y = y1 - q*y2
        y1 = y2
        y2 = y
    return(x1, y1, a)

def selectE(fn, halfkeyLength):
    while True:
        #e and fn are relatively prime
        e = random.randint(0, 1<<halfkeyLength)
        (x, y, r) = extendedGCD(e, fn)
        if r == 1:
            return e

def computeD(fn, e):
    (x, y, r) = extendedGCD(fn, e)
    #y maybe < 0, so convert it
    if y < 0:
        return fn + y
    return y

def keyGeneration(keyLength):
    #generate public key and private key
    p = findPrime(keyLength/2)
    q = findPrime(keyLength/2)
    n = p * q
    fn = (p-1) * (q-1)
    e = selectE(fn, keyLength/2)
    d = computeD(fn, e)
    return (n, e, d)

def encryption(M, e, n):
    #RSA C = M^e mod n
    return fastExpMod(M, e, n)

def decryption(C, d, n):
    #RSA M = C^d mod n
    return fastExpMod(C, d, n)


#Unit Testing
(n, e, d) = keyGeneration(1024)
#AES keyLength = 256
X = random.randint(0, 1<<256)
C = encryption(X, e, n)
M = decryption(C, d, n)
print "PlainText:", X
print "Encryption of plainText:", C
print "Decryption of cipherText:", M
print "The algorithm is correct:", X == M
```

# 3 实例

## 3.1 veryeasyRSA

> 已知 RSA 公钥生成参数：  
> p = 3487583947589437589237958723892346254777  
> q = 8767867843568934765983476584376578389  
> e = 65537  
> 求 d =  
> 请提交 PCTF {d}

对上述代码稍作修改，取有用的扩展欧拉和计算 d 的两个函数，构造如下代码，即可解出答案。

```
# coding = utf-8
def computeD(fn, e):
    (x, y, r) = extendedGCD(fn, e)
    #y maybe < 0, so convert it
    if y < 0:
        return fn + y
    return y
def extendedGCD(a, b):
    #a*xi + b*yi = ri
    if b == 0:
        return (1, 0, a)
    #a*x1 + b*y1 = a
    x1 = 1
    y1 = 0
    #a*x2 + b*y2 = b
    x2 = 0
    y2 = 1
    while b != 0:
        q = a / b
        #ri = r(i-2) % r(i-1)
        r = a % b
        a = b
        b = r
        #xi = x(i-2) - q*x(i-1)
        x = x1 - q*x2
        x1 = x2
        x2 = x
        #yi = y(i-2) - q*y(i-1)
        y = y1 - q*y2
        y1 = y2
        y2 = y
    return(x1, y1, a)
p = 3487583947589437589237958723892346254777
q = 8767867843568934765983476584376578389
e = 65537
n = p * q
fn = (p - 1) * (q - 1)
d = computeD(fn, e)
print d
```

## 3.2 easyRSA

> 还记得 veryeasy RSA 吗？是不是不难？那继续来看看这题吧，这题也不难。  
> 已知一段 RSA 加密的信息为：0xdc2eeeb2782c 且已知加密所用的公钥：  
> (N=322831561921859 e = 23)  
> 请解密出明文，提交时请将数字转化为 ascii 码提交  
> 比如你解出的明文是 0x6162，那么请提交字符串 ab  
> 提交格式：PCTF {明文字符串}

依然是一样的思路，取有用的函数，稍作修改。

```
# coding = utf-8
def fastExpMod(b, e, m):
    """
    e = e0*(2^0) + e1*(2^1) + e2*(2^2) + ... + en * (2^n)
    b^e = b^(e0*(2^0) + e1*(2^1) + e2*(2^2) + ... + en * (2^n))
        = b^(e0*(2^0)) * b^(e1*(2^1)) * b^(e2*(2^2)) * ... * b^(en*(2^n))
    b^e mod m = ((b^(e0*(2^0)) mod m) * (b^(e1*(2^1)) mod m) * (b^(e2*(2^2)) mod m) * ... * (b^(en*(2^n)) mod m) mod m
    """
    result = 1
    while e != 0:
        if (e&1) == 1:
            # ei = 1, then mul
            result = (result * b) % m
        e >>= 1
        # b, b^2, b^4, b^8, ... , b^(2^n)
        b = (b*b) % m
    return result
def computeD(fn, e):
    (x, y, r) = extendedGCD(fn, e)
    #y maybe < 0, so convert it
    if y < 0:
        return fn + y
    return y
def extendedGCD(a, b):
    #a*xi + b*yi = ri
    if b == 0:
        return (1, 0, a)
    #a*x1 + b*y1 = a
    x1 = 1
    y1 = 0
    #a*x2 + b*y2 = b
    x2 = 0
    y2 = 1
    while b != 0:
        q = a / b
        #ri = r(i-2) % r(i-1)
        r = a % b
        a = b
        b = r
        #xi = x(i-2) - q*x(i-1)
        x = x1 - q*x2
        x1 = x2
        x2 = x
        #yi = y(i-2) - q*y(i-1)
        y = y1 - q*y2
        y1 = y2
        y2 = y
    return(x1, y1, a)
def decryption(C, d, n):
    #RSA M = C^d mod n
    return fastExpMod(C, d, n)
p = 13574881
q = 23781539
n = p * q
fn = (p - 1) * (q - 1)
e = 23
d = computeD(fn, e)
C = int('0xdc2eeeb2782c', 16)
M = decryption(C, d, n)
flag = str(hex(M))[2:-1]
print d
print flag.decode('hex')
```

# RSA 常用工具
介绍一下 RSA 中常用的两款工具 RSAtool 和 yafu。

## 1 RSAtool

没有找到 `RSAtool` 自己的网站或是官方一点的下载地址，搜索引擎大致看了看也没看到比较好的介绍或是使用说明，只好自己随便写写了。

`RSAtool` 是一个非常方便实用的小工具，可以用来计算 `RSA` 中的几个参数、生成密钥、加解密，一些不太复杂的破解工作也可以用它。

我们找一道题为例，来看看 `RSAtool` 的基本用法。

> 还记得 veryeasy RSA 吗？是不是不难？那继续来看看这题吧，这题也不难。  
> 已知一段 RSA 加密的信息为：0xdc2eeeb2782c 且已知加密所用的公钥：  
> (N=322831561921859 e = 23)  
> 请解密出明文，提交时请将数字转化为 ascii 码提交  
> 比如你解出的明文是 0x6162，那么请提交字符串 ab  
> 提交格式：PCTF {明文字符串}

这道题可以用 Python 算出来，在上一篇 `RSA` 简单介绍里已经说过了，其实用 `RSAtool` 可以更方便，因为不用自己去写脚本。

![RSAtool 界面](https://gitee.com/l4fu/img_dm5vdGVfaW1n/raw/master/vnotes/sec_note/rsa%20算法科普及简单%20python%20实现.md/186710217233140.png "RSAtool 界面")

图中的 `P`、`Q`、`R`、`D`、`E` 分别就是 `RSA` 算法中的 `p`、`q`、`N`、`d`、`e`，右上角选择进制，注意不要弄错，`e` 只有十六进制可用，所以这里要把 `23` 换成 `17`。

将 `N=322831561921859` 填入，左下角有一个 `Factor N` 的按钮，这是分解 `N` 的意思，点一下，会自动开始分解因数，得到 `P=13574881`、`Q=23781539`，再点一下 `Calc. D`，计算出 `d=42108459725927`，这时可以看到 `Test` 按钮不再是灰色，表明可以使用简单的加解密功能，点它，弹出一个框。

![RSAtool 加解密功能](https://gitee.com/l4fu/img_dm5vdGVfaW1n/raw/master/vnotes/sec_note/rsa%20算法科普及简单%20python%20实现.md/169660217221602.png "RSAtool 加解密功能")

第一个框是明文，第二个框是密文，输入明文 `6162`，点击 `Encrypt`，得到密文 `178401292768926`，这时就可以使用解密功能（好像必须先用一次加密才行）。

密文 `0xdc2eeeb2782c`，换算十进制 `242094131279916`，点 `Decrypt`，直接得到字符串 `3a5Y`。

## 2 yafu

`yafu` 主要是对比较大的 `N` 进行分解，效率比普通的方法快很多。比如分解 `N = 87924348264132406875276140514499937145050893665602592992418171647042491658461`。  
用 `RSAtool` 会崩掉。。。

![yafu 界面](https://gitee.com/l4fu/img_dm5vdGVfaW1n/raw/master/vnotes/sec_note/rsa%20算法科普及简单%20python%20实现.md/150560217248353.png "yafu 界面")

用法很简单，就是 `factor()` 就好了。  
![4.jpg](https://gitee.com/l4fu/img_dm5vdGVfaW1n/raw/master/vnotes/sec_note/rsa%20算法科普及简单%20python%20实现.md/127420217230565.png "4.jpg")

过几分钟会出结果。

```
P39 = 275127860351348928173285174381581152299
P39 = 319576316814478949870590164193048041239
```