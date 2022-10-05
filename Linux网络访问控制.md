# Linux网络访问控制
## 访问控制
###  系统防火墙
允许优先于拒绝
vi /etc/hosts.deny
vi /etc/hosts.allow
### syncookie
目的：
打开syncookie 缓解syn flood攻击
检查方法：
执行cat/proc/sys/net/ipv4/tcp_syncookies命令查看是否为1
加固方法：
执行命令echo 1>/proc/sys/net/ipv4/tcp_syncookies

/proc/sys/net/ipv4/ 下有一个等待队列大小的值，可以调整其值。

### ICMP
目的：
不响应ICMP请求，避免信息泄露
检查方法：
执行cat/proc/sys/net/ipv4/icmp_echo_ignore_all 命令查看值是否为1
加固方法：执行命令
echo 1>/proc/sys/net/ipv4/icmp_echo_ignore_all
###  FTP
目的：
保护ftp传输安全
加固方法：
vi/etc/vsftpd/vsftpd.conf
1.anonymous_enable=NO：禁止匿名登录服务
2.ftpd banner=Welcome：禁止显示banner信息
3.修改ftpusers和user_list文件里的用户信息
4.限制用户访问目录：
    1.chroot list enable=YES
    2.chroot list file=/etc/vsftpd/chroot list
    3.新建/etc/vsftpd/chroot list，添加用户名，如：user
     表示user登录FTP后，只允许在user用户的home目录中活动
5.listen_address=1.1.1.1：修改监听地址
6.listen_port=8888：修改默认端口

## iptables防火墙
### iptables结构
iptables将防火墙的功能分成多个tables
Filter：数据包过滤
NAT:Network Address Translation/网络地址转换
Managle：用于修改一些特殊的规则
Raw：用来决定是否对数据包进行状态跟踪。（不常用）
tables又包含多个chains
5条默认基础操作chains 
PREROUTING：路由之前，刚到达的数据包。（nat）
INPUT：通过路由，目的为地为本机的数据包。（filter）
FORWARD：需要通过本地系统进行转发的数据包。（filter）
OUTPUT：由本机产生，向外转发，处于POSTROUTING之前的数据包。（nat和filter）
POSTROUTIONG：通过路由后，即将离开系统的数据包。（nat）
允许用户自行定义chains

### iptables语法
iptables[-ttable]<action>[pattern][-jtarget]
action包括：
-ACCEPT：未经禁止全部许可A chain：在chain中增添一条规则
-Dchain：在chain中删除一条规则
Lchain：列出chain中的规则
-Fchain：清空chain中的规则
-Pchain：为chain指定新的默认策略，可以是：
DROP：未经许口全部禁止

 iptables语法pattern包括：
-s<ip地址>：来源地址
-d<ip地址>：目标地址p<协议>：指定协议，可以是tcp/udp/icmp
-dport<端囗>：目标端口，需指定-p L--sport<端口>：来源端口，需指定-p 
target包括：
DROP：禁止
ACCEPT：许可

默认table和chain filter table用于过滤数据包的接送
chain INPUT：设定远端访问主机时的规则来源是远端访问者，目标是本地主机chain OUTPUT：设定主机访问远端主机的规则来源是本地主机，目标是远端被访问主机chain FORWARD：
设定主机为其他主机转发数据包时的规则来源是请求转发的主机，目标是远端被访问的主机

### iptables配置实例
1、规则清零：
iptables[-FXZ]
F：清除所有的已制定的规则//-x：除掉所有用户“自定义”的chain/-z：将所有的chain的计数与流量统计都归零
2、默认策略：
iptables-P INPUT DROP iptables-P OUTPUT ACCEPT iptables-P FORWARD ACCEPT
3、回应数据包：
iptables-A INPUT-m state--state RELATED，ESTABLISHED-jACCEPT
4、添加自定义规则：
iptables-A INPUT-p tcp--dport 80-j ACCEPT iptables-A INPUT-p tcp-s 192.168.1.0/24-dport 22-jACCEPT
iptables-A INPUT-p tcp-s 192.168.1.0--dport 21-jACCEPT 
iptables-A OUTPUT-p tcp-d 192.168.1.0---dport 3389-jDROP
.……
service iptables save
service iptables restart

## firewalld
### firewalld使用
查看状态：#systemctl status firewalld 或者firewall-cmd--state
启动：#systemctl start firewalld
停止：#systemctl stop firewalld
使能：#systemctl enable firewalld
禁用：#systemctl disable firewalld

firewalld-cmd参数：
-get-default-zone查访默认的区域名称
-set-default-zone=<区域名称>设置默认的区域，使其永久生效
-get-services显示预定义的服务
-get-active-zones显示当前正在使用的区域、来源地址和网卡名称
-add-source=将源自此IP或子网的流量导向指定的区域
-remove-source=不再将源自此IP或子网的流量导向这个区域
-add-interface=<网卡名称>将源自该网卡的所有流量都导向某个指定区域
-change-interface=<网卡名称>将某个网卡与区域进行关联
-list-all显示当前区域的网卡配置参数、资源、端口以及服务等信息
-list-all-zones显示所有区域的网卡配置参数、资源、端口以及服务等信息

 例：
firewalld-cmd-add-source=192.168.1.1
firewalld-cmd--zone=drop--add-source=192.168.1.1
firewalld-cmd--zone=public--add-port=8080
firewalld-cmd--zone=public--remove-port=8080
firewall-cmd--zone=drop--list-ports
firewall-cmd--zone=drop-remove-protocol=icmp禁ping
firewall-cmd--zone=drop--add-forward-port=port=888：proto=tcp:toport=22

firewalld-config  可视化界面

rich rules当基本firewalld语法规则不能满足要求时，可以使用的更复杂的规则就是rich rules选项：
-add-rich-rules向指定区域中添加rule（永久添加需要-permanent）
-remove-rich-rules从指定区域删除rule（永久删除需要-permanent）
-query-rich-rules查询rule是否添加到指定区域，如果存在则返回0，否则返回1。
-list-rich-rules输出指定区域的所有富规则

--add-rich-rules family：指定该规则仅应用于IPv4数据包，否则该规则将同时应用于IPv4和IPv6。
source address：提供数据包必须具有的源地址，才能使用源地址触发规则。
service，指定规则的服务类型，在本例中为ssh。
reject/drop/accept，提供了数据包与规则匹配时要执行的操作。
![例子](_v_images/20200512150103501_16841.png)
```
firewall-cmd--permanent --zone=block --add-rich-rule='rule family=ipv4 source
address=192.168.0.11/32 reject'

```
rich规则的执行逻辑如下：
1.端口转发和伪装规则
1.日志规则
2.accept规则
3.drop/reject规则


### 例子
每分钟允许2个新连接访问ftp服务
firewall-cmd--add-rich-rule="rule service name=ftp limit value=2/m accept"
允许新的ipv4和ipv6连接ftp，并使用日志和审核，每分钟允许访问一次
firewall-cmd--add-rich-rule="rule service name=ftp log limit value="1/m"audit accept"
拒绝来自192.168.2.0/24网段的连接，10秒后自动取消
firewall-cmd--add-rich-rule="rule family=ipv4 source address=192.168.2.0/24 reject"--
timeout=10允许ipv6地址为2001：db8：：/64子网的主机访问dns服务，并且每小时审核一次，300秒后自动取消firewall-cmd--add-rich-rule="rule family=ipv6 source address="2001：db8：：/64"service name="dns"audit limit value="1/h"reject"--timeout=300将来自192.168.1.0/24网段访问本机8e端口的流量转发到本机的22端口
firewall-cmd--zone=drop--add-rich-rule="rule family=ipv4 source address=192.168.1.0/24
forward-port port=80 protocol=tcp to-port=22"
将来自192.168.2.0/24网段访问本地80端口的流量转发到192.168.1.1主机的22端口白，e firewall-cmd--zone=drop--add-rich-rule="rule family=ipv4 source address=192.168.2.0/24
forward-port port=8e protocol=tcp to-port=22 to-addr=192.168.1.1"
允许新的ipv4和ipv6连接ftp，并使用日志和审核，每分钟允许访问一次
firewall-cmd--add-rich-rule="rule service name=ftp log limit value="1/m"audit accept"
拒绝来自192.168.2.e/24网段的连接，10秒后自动取消
firewall-cmd--add-rich-rule="rule family=ipv4 source address=192.168.2.0/24 reject"--
timeout=10允许ipv6地址为2001：db8：：/64子网的主机访问dns服务，并且每小时审核一次，300秒后自动取消firewall-cmd--add-rich-rule="rule family=ipv6 source address="2001：db8：：/64"service name="dns"audit limit value="1/h"reject"--timeout=300将来自192.168.1.e/24网段访问本机8e端口的流量转发到本机的22端口
firewall-cmd--zone=drop--add-rich-rule="rule family=ipv4 source address=192.168.1.0/24
forward-port port=8e protocol=tcp to-port=22"
将来自192.168.2.e/24网段访问本地8e端口的流量转发到192.168.1.1主机的22端口firewall-cmd--zone=drop--add-rich-rule="rule family=ipv4 source address=192.168.2.0/24
forward-port port=8e protocol=tcp to-port=22 to-addr=192.168.1.1"



## apt-get（apt）的基本使用
1、安装软件 
apt-get（apt） install softname1 softname2...
2、卸载软件 
apt-get（apt） remove softname1 softname2...
3、卸载并清除配置 
apt-get（apt） remove --purge softname1
4、更新软件信息数据库
apt-get（apt） update
5、进行系统升级 
apt-get（apt） upgrade
6、修正依赖关系 
apt-get（apt） -f install
7、获取包的相关信息
apt-cache show sofname1（获取如说明、大小、版本等）
8、搜索软件包 
apt-cache search softname1 softname2...

## dpkg的基本使用
1、安装软件 
dpkg -i xxx.deb （dpkg -i /share/google-chrome-stable_current_amd64.deb）
注意：如果通过dpkg –i安装软件后由于依赖关系没有安装成功,可通过apt-get –f install解决

2、安装一个目录下面所有的软件包
dpkg -R 目录路径 （dpkg -R /usr/local/src）

3、删除软件包 
dkpg -r xxx.deb

4、连同配置文件一起删除
dpkg -r --purge xxx.deb

5、查看软件包信息 
dpkg -info xxx.deb

6、查看文件拷贝详情 
dpkg -L xxx.deb

7、查看系统中已安装软件包信息
dpkg -l （ii表示安装成功，iU表示未安装成功）

8、查看已安装包的详细情况 
dpkg -s 安装包名称（包括版本和依赖之类的）
