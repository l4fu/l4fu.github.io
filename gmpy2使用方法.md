# gmpy2使用方法
## 基本使用

本文只做简单介绍。以下代码均在Python 3.4中运行。初始化一个大整数，只需要

```python
import gmpy2
n=gmpy2.mpz(1257787) #初始化
gmpy2.is_prime(n) #概率性素性测试

n=100
x=59
e=1
a=1156165169
b=78451
M=56231351516
gmpy2.mpz(n)#初始化一个大整数
gmpy2.mpfr(x)# 初始化一个高精度浮点数x
d = gmpy2.invert(e,n) # 求逆元，de = 1 mod n
print(d)
C = gmpy2.powmod(M,e,n)# 幂取模，结果是 C = (M^e) mod n
gmpy2.is_prime(n) #素性检测
gmpy2.gcd(a,b)  #欧几里得算法，最大公约数
gmpy2.gcdext(a,b)  #扩展欧几里得算法
gmpy2.iroot(x,n) #x开n次根

```

这里跟C/C++是平行的，其实括号里边的参数，可以是整型，也可以是字符串。gmpy2中不仅集成了大整数mpz，还有高精度浮点数mpfr，用法跟C/C++都是平行的。下面是一些基本计算

```python
a+2 #求和，结果是一个mpz
a-2 #求差，结果是一个mpz
a*2 #求积，结果是一个mpz
a/2 #求商，结果是一个mpfr
a//2 #求商，结果是一个mpz
a**2 #求平方，结果是一个mpz
a%2 #求模，结果是一个mpz
```

这些运算符的重载既直观又方便，其中将数字2换成mpz类型变量也是可以的，这自然不用多说。事实上，运算a+2，就是先将2转换为mpz中的2，然后执行mpz中的加法。

**与Python自身比较**

Python本身支持高精度大整数运算，一般的数字比较小的范围足够了，只是并非“快速”，读者只需要分别运行以下两句代码

```python
gmpy2.mpz(1257787)**123456 #gmpy2计算
1257787**123456 #Python自带计算
```

就能感觉到了（当然，可能读者的配置还不错，两者并没有明显差别，那么请把指数扩大十倍~笔者的笔记本配置不高，见笑了^_^）。
## gmpy2常见函数使用
### 1.初始化大整数

```
import gmpy2
gmpy2.mpz(909090)

#result:mpz(909090)
```

### 2.求大整数a,b的最大公因数

```
import gmpy2
gmpy2.gcd(6,18)

#result:mpz(6)
```

### 3.求大整数x模m的逆元y

```
import gmpy2
#4*6 ≡ 1 mod 23
gmpy2.invert(4,23)

#result:mpz(6)

```
### 4.检验大整数是否为偶数

```
import gmpy2
gmpy2.is_even(6)

#result:True

-----------
import gmpy2
gmpy2.is_even(7)

#result:False
```

### 5.检验大整数是否为奇数

```
import gmpy2
gmpy2.is_odd(6)

#result:False
-----------
import gmpy2
gmpy2.is_odd(7)

#result:True
```

### 6.检验大整数是否为素数

```
import gmpy2
gmpy2.is_prime(5)

#result:True
```

### 7.求大整数x开n次根

```
import gmpy2
gmpy2.iroot(81,2)

#result:(mpz(9),True)
```
### 8.求大整数x的y次幂模m取余

```
import gmpy2
#2^4 mod 5 
gmpy2.powmod(2,4,15)
#result:mpz(1)
```

## 尝试大数分解

下面来一段自己编写的大数分解的代码：

```python
from gmpy2 import *
import time
start=time.clock()
n = mpz(63281217910257742583918406571)
x = mpz(2)
y = x**2 + 1
for i in range(n):
    p = gcd(y-x,n)
    if p != 1:
        print(p)
        break
    else:
        y=(((y**2+1)%n)**2+1)%n
        x=(x**2+1)%n
end=time.clock()
print(end-start)
```

该代码调用了gmpy2，算法则是Pollard rho方法，只是最单纯地运行，并没做任何优化。该算法在我的笔记本上分解

> 63281217910257742583918406571 = 125778791843321 × 503115167373251

耗时84秒（获得一个因子即停止）。当然这不是什么了不起的成绩，这个数字在Mathematica和PARI/GP中都是瞬间完成的。这个脚本只是纯粹练习和测试Python对gmpy2的调用。

关于Pollard rho方法的步骤：

> 1、给定合数nn，以及x0=2,y0=x20+1x0=2,y0=x02+1；
> 
> 2、计算p=Gcd(x0−y0,n)p=Gcd(x0−y0,n)，如果p非1，则结果为nn的一个因子，停止；
> 
> 3、如果p=1，则按照xn+1=x2n+1,yn+1=(y2n+1)2+1xn+1=xn2+1,yn+1=(yn2+1)2+1，然后重复第2步。

其中，为了减少计算量，每一步xn,ynxn,yn的计算，都要作模nn运算，以降低计算量。上述步骤只是最简单的计算部分，还没有处理各种可能出现的异常，比如出现n|(xn−yn)n|(xn−yn)等；也没有对某些细节进行优化，请谨慎引用。

## gmpy2.invert方法的15个代码示例
这些例子默认根据受欢迎程度排序。您可以为喜欢或者感觉有用的代码点赞，您的评价将有助于我们的系统推荐出更棒的Python代码示例。

### 示例1: invert

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def invert(a, b):
    """
    The multiplicitive inverse of a in the integers modulo b.

    :return int: x, where a * x == 1 mod b
    """
    if HAVE_GMP:
        s = int(gmpy2.invert(a, b))
        # according to documentation, gmpy2.invert might return 0 on
        # non-invertible element, although it seems to actually raise an
        # exception; for consistency, we always raise the exception
        if s == 0:
            raise ZeroDivisionError('invert() no inverse exists')
        return s
    else:
        r, s, _ = extended_euclidean_algorithm(a, b)
        if r != 1:
            raise ZeroDivisionError('invert() no inverse exists')
        return s % b 
```

开发者ID:data61，项目名称:python-paillier，代码行数:21，代码来源:[util.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fdata61%2Fpython-paillier)


### 示例2: build_table

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def build_table(h, g, p, B):
    table, z = {}, h
    g_inverse = invert(g, p)
    table[h] = 0
    for x1 in range(1, B):
        z = t_mod(mul(z, g_inverse), p)
        table[z] = x1
    return table 
```

开发者ID:mithi，项目名称:simple-cryptography，代码行数:10，代码来源:[mitm.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fmithi%2Fsimple-cryptography)


### 示例3: compute_d

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def compute_d(e, N, p, q):
    # d * e mod phi(N) = 1
    # where phi(N) = N - p - q + 1
    phiN = phi(N, p, q)
    d = invert(mpz(e), mpz(phiN))
    x = mul(mpz(d), mpz(e))
    assert t_mod(x, mpz(phiN)) == 1
    return d.digits() 
```

开发者ID:mithi，项目名称:simple-cryptography，代码行数:10，代码来源:[publickeysystem.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fmithi%2Fsimple-cryptography)


### 示例4: sign

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def sign(self, m, k):
        h = int(hashlib.md5(m).hexdigest(), 16)
        r = pow(self.g, k, self.p) % self.q
        s = int(((self.x * r + h) * gmpy2.invert(k, self.q)) % self.q)
        return (r, s) 
```

开发者ID:ashutosh1206，项目名称:Crypton，代码行数:7，代码来源:[encrypt.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fashutosh1206%2FCrypton)

### 示例5: verify

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def verify(self, m, r, s):
        if 0 < r and r < self.q and 0 < s and s < self.q:
            h = int(hashlib.md5(m).hexdigest(), 16)
            w = gmpy2.invert(s, self.q)
            u1 = (h * w) % self.q
            u2 = (r * w) % self.q
            v = ((pow(self.g, u1, self.p) * pow(self.y, u2, self.p)) % self.p) % self.q
            return v == r
        return None 
```

开发者ID:ashutosh1206，项目名称:Crypton，代码行数:11，代码来源:[encrypt.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fashutosh1206%2FCrypton)

### 示例6: crt

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def crt(list_a, list_m):
    """
    Reference: https://crypto.stanford.edu/pbc/notes/numbertheory/crt.html
    Returns the output after computing Chinese Remainder Theorem on

    x = a_1 mod m_1
    x = a_2 mod m_2
    ...
    x = a_n mod m_n

    input parameter list_a = [a_1, a_2, ..., a_n]
    input parameter list_m = [m_1, m_2, ..., m_n]

    Returns -1 if the operation is unsuccessful due to some exceptions
    """
    try:
        assert len(list_a) == len(list_m)
    except:
        print "[+] Length of list_a should be equal to length of list_m"
        return -1
    for i in range(len(list_m)):
        for j in range(len(list_m)):
            if GCD(list_m[i], list_m[j])!= 1 and i!=j:
                print "[+] Moduli should be pairwise co-prime"
                return -1
    M = 1
    for i in list_m:
        M *= i
    list_b = [M/i for i in list_m]
    assert len(list_b) == len(list_m)
    try:
        list_b_inv = [int(gmpy2.invert(list_b[i], list_m[i])) for i in range(len(list_m))]
    except:
        print "[+] Encountered an unusual error while calculating inverse using gmpy2.invert()"
        return -1
    x = 0
    for i in range(len(list_m)):
        x += list_a[i]*list_b[i]*list_b_inv[i]
    return x % M 
```

开发者ID:ashutosh1206，项目名称:Crypton，代码行数:41，代码来源:[hastad_unpadded.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fashutosh1206%2FCrypton)

### 示例7: neg_pow

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def neg_pow(a, b, n):
    assert b < 0
    assert GCD(a, n) == 1
    res = int(gmpy2.invert(a, n))
    res = pow(res, b*(-1), n)
    return res

# e1 --> Public Key exponent used to encrypt message m and get ciphertext c1
# e2 --> Public Key exponent used to encrypt message m and get ciphertext c2
# n --> Modulus
# The following attack works only when m^{GCD(e1, e2)} < n 
```

开发者ID:ashutosh1206，项目名称:Crypton，代码行数:13，代码来源:[exploit.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Fashutosh1206%2FCrypton)

### 示例8: invert

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def invert(a, b):
    """return int: x, where a * x == 1 mod b
    """    
    x = int(gmpy2.invert(a, b))
   
    if x == 0:
        raise ZeroDivisionError('invert(a, b) no inverse exists')
    
    return x 
```

开发者ID:FederatedAI，项目名称:FATE，代码行数:11，代码来源:[gmpy_math.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FFederatedAI%2FFATE)


### 示例9: EEA


```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def EEA(n1, n2):
    print('x*y == 1 (mod p)')
    print('y = invmod(x, p)')
    print('\n\033[1;34m[*]\033[0m Calculating...\n')
    n3 = gmpy2.invert(n1, n2)
    n3 = str(n3)
    n3 = n3.replace('mpz(', '')
    n3 = n3.replace(')', '')
    print('\033[1;34m[*]\033[0m ' + str(n1) + ' * ' + str(n2) + ' == 1 (mod ' + str(n2) + ')')
    print('\033[1;32m[+]\033[0m invmod(' + str(n1) + ', ' + str(n2) + ') = ' + str(n3)) 
```

开发者ID:lockedbyte，项目名称:cryptovenom，代码行数:12，代码来源:[main.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flockedbyte%2Fcryptovenom)

### 示例10: inverse



```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def inverse(n1,n2):
    try:
        n3 = gmpy2.invert(n1,n2)
    except ZeroDivisionError:
        n3 = 'ERR'
    return n3 
```

开发者ID:lockedbyte，项目名称:cryptovenom，代码行数:8，代码来源:[main.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flockedbyte%2Fcryptovenom)


### 示例11: embedPriv

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def embedPriv(p, q, e):
    p = bytes_to_long(long_to_bytes(p))
    q = bytes_to_long(long_to_bytes(q))
    e = bytes_to_long(long_to_bytes(e))
    n = p * q
    n = bytes_to_long(long_to_bytes(n))
    phi = (p - 1) * (q - 1)
    d = gmpy2.invert(e, phi)
    d = bytes_to_long(long_to_bytes(d))
    key = RSA.construct((n,e,d,p,q))
    key = RSA._RSAobj.exportKey(key).decode('utf-8')
    return key 
```

开发者ID:lockedbyte，项目名称:cryptovenom，代码行数:14，代码来源:[main.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flockedbyte%2Fcryptovenom)


### 示例12: RSAsolverpqec

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def RSAsolverpqec(p, q, e, c):

    print('\033[1;34m[*]\033[0m Calculating Plain-Text')
    n = int(p) * int(q)
    
    phi = (int(p) - 1) * (int(q) - 1)
    
    d = gmpy2.invert(int(e), phi)
    
    m = pow(int(c),int(d),int(n))
    
    return m 
```

开发者ID:lockedbyte，项目名称:cryptovenom，代码行数:14，代码来源:[main.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flockedbyte%2Fcryptovenom)

### 示例13: fermatAttack

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def fermatAttack(n, e):
    try:
        (p, q) = Fermat.fermat(n, e)
    except:
        print("\n\033[1;31m[-]\033[0m This RSA key is not valid for a Fermat Attack\n")
        exit()
        phi = indicatrice_euler(p,q)
        d = gmpy2.invert(e, phi)       
        return d 
```

开发者ID:lockedbyte，项目名称:cryptovenom，代码行数:11，代码来源:[main.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flockedbyte%2Fcryptovenom)

### 示例14: invert

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def invert(x, m):
        """Return y such that x*y == 1 (mod m), assuming m is prime.

        Raises ZeroDivisionError if no inverse exists.
        """
        if m == 2:
            y = x%2
        else:
            y = pow(x, m-2, m)
        if y == 0:
            raise ZeroDivisionError

        return y 
```

开发者ID:lschoe，项目名称:mpyc，代码行数:15，代码来源:[gmpy.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2Flschoe%2Fmpyc)


### 示例15: neg_pow

```
# 需要导入模块: import gmpy2 [as 别名]
# 或者: from gmpy2 import invert [as 别名]
def neg_pow(a, b, n):
        assert b < 0
        assert GCD(a, n) == 1
        res = int(gmpy2.invert(a, n))
        res = pow(res, b*(-1), n)
        return res 
```

开发者ID:X-Vector，项目名称:X-RSA，代码行数:8，代码来源:[RSA\_common\_modulus.py](https://vimsky.com/cache/index.php?source=https%3A%2F%2Fgithub.com%2FX-Vector%2FX-RSA)


##  RSA gmpy2函数实现.
### 一.pow:
#### 1 .分奇偶性去用递归函数迭代计算：
```
def mypow(x:float,y:int) ->float:
   if y == 0:
       return 1
   if y%2 == 0:
        return mypow(x*x,y//2)
   if y%2 == 1:
        return x*mypow(x,y-1)
```

#### 2 .快速幂方法：
计算a的b次方，需要考虑：

1、b==0 的话，返回1

2、b<0的情况，设置一个flag，用于返回数据的时候进行判断

3、b>0的情况：

```
def Power(self, base, exponent):
        # write code here
        flag=1
        re=1
        tmp=base
        if exponent==0:   #等于0的情况
            return 1
        if exponent<0:   #小于0的时候，设置一个flag,用于返回的时候做判断
            flag=0
            exponent=abs(exponent)
        while exponent>0:    #大于0的情况
            if exponent&1==1:
                re=re*tmp     #如果是奇数，奇数减一就是偶数
            exponent>>=1      #右移动一位就是除以2
            tmp=tmp*tmp       #2^6=(2*2)^3
        return re if flag else 1/re
```
 
### 二.powmod
​ 即在pow的属性上再加上一个输入值 为mod 的东西，再在输出时 原来pow得出的值取mod

### 三.long_to_bytes
1.long：长整数类型，无限大小
2.bytes：是指一堆字节的集合，十六进制表现形式，两个十六进制数构成一个 byte ，以 b 开头的字符串都是 bytes 类型。
```
def long_to_bytes(a:int) -> str:
    m=hex(a)
    q=m[2:]
    t=bytes.fromhex(q)
    return t
```
### 四.产生大素数
欧拉筛法代码：
```
def get_prime(n):
    isPrime= [False for i in range(2, n+3)]
    prime = []
    for i in range(2, n):
        if isPrime[i] is False:
            prime.append(i)
        for j in prime:
            if i * j > n:
                break
            isPrime[i * j] = True
            if i % j == 0:
                break
    return prime
print(get_prime(120))
```

### 五.通过Miller-Rabin素数检测方法实现大素数的随机产生
#### 1.两个基础定理
a.费马小定理：当  为质数，有 ，不过反过来不一定成立，也就是说，如果  ,  互质，且 ，不能推出  是质数，比如  数（这个就自行百度吧）
b.二次探测：如果  是一个素数，, 则方程  的解为  或 
#### 2.两个基本定理的证明
![70](_v_images/20210827091317637_14863.png)

#### 3.算法流程：
（1）对于偶数和 0，1，2 可以直接判断。

（2）设要测试的数为 ，我们取一个较小的质数 ，设 ，满足 （其中  是奇数）。

（3）我们先算出 ，然后不断地平方并且进行二次探测（进行  次）。

（4）最后我们根据费马小定律，如果最后 ，则说明  为合数。

（5）多次取不同的  进行  素数测试，这样可以使正确性更高

#### 4.备注：
（1）我们可以多选择几个 ，如果全部通过，那么  大概率是质数。

（2） 素数测试中，“大概率”意味着概率非常大，基本上可以放心使用。

（3）当  取遍小等于 30 的所有素数时，可以证明  范围内的数不会出错。

（4）代码中我用的  类型，不过实际上  素数测试可以承受更大的范围。

（5）另外，如果是求一个   类型的平方，可能会爆掉，因此有时我们要用“快速积”，不能直接乘。

#### 5.代码块：
Miller_Rabin素数检测 方法.
```
def Miller_Rabin(n):
    if n == 2 or n == 3:
        return True
    if n & 1 == 0:
        return False
    s, d = 0, n - 1
    while d % 2 == 0:
        s += 1
        d //= 2
    for i in pip._vendor.msgpack.fallback.xrange(80):  # 进行80轮检验
        # xrange() 函数用法与 range 完全相同，所不同的是生成的不是一个数组，而是一个生成器。
        a = randrange(2, n - 1)  # 按递增的方式产生随机数
        # randrange() 方法返回指定递增基数集合中的一个随机数，基数默认值为1。
        k = pow(a, d, n)  # k = a^d mod n
        if k == 1 or k == n - 1:
            continue
        # 对Miller序列测试
        for r in pip._vendor.msgpack.fallback.xrange(s):
            k = pow(k, 2, n)  # k = a ^[(2^r)*d]  mod n
            if k == n - 1:
                break
        else:
            return False
    return True


# 产生大素数
def getPrime(n):
    if n <= 2:
        return False
    while True:
        t = '1'
        for i in range(n - 2):
            t += str(randint(0, 1))
        t += '1'
        t = int(t, 2)
        if Miller_Rabin(t):
            return t
```

### 六.裴蜀定理
1.内容：若a,b是整数,且gcd(a,b)=d，那么对于任意的整数x,y,ax+by都一定是d的倍数，特别地，一定存在整数x,y，使ax+by=d成立。
2.重要推论：a,b互质的充分必要条件是存在整数x,y使ax+by=1
### 七.invert 函数实现：
#### 1.已知
```
d ∗ e ≡ 1 ( m o d φ ( n ) ) d*e≡1(mod φ(n))
d∗e≡1(modφ(n))

( e , φ ( n ) ) = 1 (e,φ(n))=1
(e,φ(n))=1

a = e , y = φ ( n ) a=e,y=φ(n)
a=e,y=φ(n)

e x + φ ( n ) y = 1 ex+φ(n)y=1
ex+φ(n)y=1

e x ≡ 1 ( m o d ϕ ( n ) ) ex \equiv1(mod \phi(n))
ex≡1(modϕ(n))

```
只需解出x的整数解即可

#### 2.所以可以通过推导得出
```
gcd, d, y = exgcd(e, phi)
temp = phi // gcd
# 解出ed=1(mod phi)的最小正整数解，
d = (d % phi + phi) % phi

```
即可以得出d的最小整数解
