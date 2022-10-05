# PHP反序列化
序列化和反序列化的目的是使得程序间传输对象会更加方便。 序列化是将对象转换为字符串以便存储传输的一种方式。而反序列化恰好就是序列化的逆过程,反序列化会将字符串转换为对象供程序使用。 在PHP中序列化和反序列化对应的函数分别为serialize()和unserialize()。反序列化本身并不危险,但是如果反序列化时,传入反序列化函数的参数可以被用户控制那将会是一件非常危险的事情。不安全的进行反序列化造成的危害是非常大的。
## 原理
- unserialize() 对单一的已序列化的变量进行操作，将其转换回 PHP 的值。返回的是转换之后的值，可为 integer、float、string、array 或 object。如果传递的字符串不可解序列化，则返回 FALSE。若被解序列化的变量是一个对象，在成功地重新构造对象之后，PHP 会自动地试图去调用 __wakeup() 成员函数（如果存在的话）
- unserialize函数是与serialize函数相对应的，它们两个的作用就是将变量进行序列化与反序列化。
- unserialize工作的过程。unserialize再回复变量之前，会根据serialize生成的字符串中的变量信息，重新创造一个变量，并为它赋值。

## 利用
利用unserialize的条件要具备一下几点，
1.unserialize函数的参数可控
2.脚本中存在一个构造函数、析构函数、__wakeup()函数中有向php文件中写数据的操作的类
3.所写的内容需要有对象中的成员变量的值

利用的思想就是通过本地构造一个和脚本中符合条件类同名的类，并对能够写入php文件的成员变量赋值，内容为将要执行的php脚本代码（例 如：phpinfo()）。然后，本地实例化这个类，并通过调用serialize函数将实例化的对象转换为字符串。最后，将获得的字符串作为 unserialize的参数进行传递。

## 实例
![反序列化漏洞点](_v_images/20201227120455781_17007.png =1244x)

![利用](_v_images/20201227120359403_26770.png =1244x)
最关键的代码就是,在进行初始化的时候，将$filename赋值为pctf.php
`function __construct($filename = 'pctf.php') { $this -> file = $filename; } 
`
最后得到的序列话的值是：
`O:6:"Shield":1:{s:4:"file";s:8:"pctf.php";}`

http://web.jarvisoj.com:32768/index.php?class=O:6:%22Shield%22:1:{s:4:%22file%22;s:8:%22pctf.php%22;} 
