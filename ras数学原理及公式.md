# ***数学原理及公式***

## 1.数论

$a\equiv b\mod p$就是$a=k*p+b$。

## 2.欧拉定理相关

$$
N=p*q\\
\varphi(N)=(p-1)*(q-1)\\
a^{\varphi(p)}\equiv1\mod p\\
a^{\varphi(p)+1}\equiv a\mod p\\
(a^e\mod p)^d\mod p=a^{e*d}\mod p
$$

## 3.mod p有限域上的逆元

$$
e*d\equiv1\mod \varphi(N)\\
e*d=n*\varphi(N)+1\\
k*k^{-1}=n*p+1\\
\gcd(k, p)=1
$$

$$
m^e\mod p=c
$$

$$
e_1, e_2\\
s_1*e_1\equiv1\mod e_2\\
s_2*e_2\equiv1\mod e_1\\
e_1*s_1+e_2*s_2 = 1
$$

$$
sk+dr\equiv m\mod p-1\\
sk+dr = m+a*(p-1)
$$

