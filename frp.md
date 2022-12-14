# frp

还是 nps 好用

**以下配置适合0.10.0~0.16.1版本**  
frp客户端与服务端0.9.3及以下版本兼容，0.10.0~0.16.1版本兼容，使用时需要注意  

0.18.0以上版本把配置中的“privilege_”字段去掉即可

## frpc客户端
```

[common]
server_addr = www.yourdomain.com 
#frps服务端地址
server_port = 7000
#frps服务端通讯端口，客户端连接到服务端内网穿透传输数据的端口
privilege_token = frp888
#特权模式密钥，客户端连接到FRPS服务端的验证密钥
log_file = frpc.log
#日志存放路径
log_level = info
#日志记录类别,可选：trace, debug, info, warn, error
log_max_days = 7
#日志保存天数
login_fail_exit = false
#设置为false，frpc连接frps失败后重连，默认为true不重连
protocol = kcp
#KCP协议在弱网环境下传输效率提升明显，但是对frps会有一些额外的流量消耗。服务端须先设置kcp_bind_port = 7000，www.yourdomain.com服务端已设置支持

[http_dsm]
#穿透服务名称,不能和其他已建立的相同，使用公共服务器的建议修改成复杂一点的名称，避免与其他人冲突，很多路由器内置frpc的默认服务名称为[web]，很容易很其他人冲突
type = http
#穿透协议类型，可选：tcp，udp，http，https，stcp，xtcp，这个设置之前必须自行搞清楚应该是什么
local_ip = 192.168.1.2
#本地监听IP，可以是本机IP，也可以是本地的局域网内某IP，例如你的局域网是互通的，你可以在路由器上安装frpc，然后local_ip填的内网其他机器ip，这样也可以把内网其他机器穿透出去
local_port = 5000
#本地监听端口，通常有ssh端口22，远程桌面3389等等
use_compression = true
#对传输内容进行压缩，可以有效减小 frpc 与 frps 之间的网络流量，加快流量转发速度，但是会额外消耗一些 cpu 资源
use_encryption = true
#将 frpc 与 frps 之间的通信内容加密传输
custom_domains = dsm.yourdomain.com
#自定义域名访问穿透服务，一般域名设置了二级域名泛解析以后，这里填*.yourdomain.com即可，*自定义，如果不想用域名或者自行搭建frps没有域名，则穿透协议类型选择tcp，见以下tcp部分详解
#通过app访问的注意，DS file,DS video,DS audio,DS finder里地址栏默认都是5000端口，穿透后地址栏须填写为【穿透域名:80】，DS photo由于本地local_port为80，穿透后也为80的话直接写域名地址即可

[https_dsm]
type = https
local_ip = 192.168.1.2
local_port = 5001
use_compression = true
use_encryption = true
custom_domains =  dsm.yourdomain.com
#以上https配置同http，注意开启https（默认5001端口），证书配置在客户端，无证书的注意浏览器访问时添加信任

[http_transmission]
type = http
local_ip = 192.168.1.2
local_port = 9091
use_compression = true
use_encryption = true
custom_domains = tr.yourdomain.com
#transmission下载客户端

[http_rutorrent]
type = http
local_ip = 192.168.1.2
local_port = 80
use_compression = true
use_encryption = true
custom_domains = rt.yourdomain.com
#rutorrent下载客户端，用Download Station的类似，注意端口


[http_blog]
type = http
local_ip = 192.168.1.2
local_port = 80
use_compression = true
use_encryption = true
custom_domains = blog.yourdomain.com


[http_plex]
type = http
local_ip = 192.168.1.2
local_port = 32400
use_compression = true
use_encryption = true
custom_domains = plex.yourdomain.com
#plex视频服务器

[https_feixun]
privilege_mode = true
type = http
local_ip = 192.168.1.1
#路由器ip
local_port = 80
use_compression = true
use_encryption = true
authentication_timeout = 0
custom_domains = feixun.yourdomain.com
#穿透路由器


[tcp_ssh]
#ssh连接
type = tcp
local_ip = 192.168.1.2
local_port = 22
use_compression = true
use_encryption = true
remote_port = 3463
#远程端口，一般tcp和udp需要设置，不需要设置custom_domain,访问时为【frps服务器地址+远程端口】，没有域名的用这种方式通过【frps服务器地址+远程端口】即可实现访问

[udp]
type = udp
local_ip = 192.168.1.2
local_port = 53
use_compression = true
use_encryption = true
remote_port = 3453
访问时为【frps服务器地址+远程端口】

```
## frps服务端

自行搭建服务端配置参考（必须有公网ip）

  
```
[common]
bind_addr = 0.0.0.0
#服务器IP，0.0.0.0为服务器全局所有IP可用，假如你的服务器有多个IP则可以这样做，或者填写为指定其中的一个服务器IP,支持IPV6
bind_port = 7000
#通讯端口，用于和客户端内网穿透传输数据的端口，可自定义
bind_udp_port = 7001
#UDP通讯端口，用于点对点内网穿透
kcp_bind_port = 7000
#用于KCP协议UDP通讯端口，在弱网环境下传输效率提升明显，但是会有一些额外的流量消耗。设置后frpc客户端须设置protocol = kcp
vhost_http_port = 80
#http监听端口，注意可能和服务器上其他服务用的80冲突，比如centos有些默认有Apache，可自定义
vhost_https_port = 443
#https监听端口，可自定义
dashboard_port = 7500
#通过浏览器查看 frp 的状态以及代理统计信息展示端口，可自定义
dashboard_user = admin
#信息展示面板用户名
dashboard_pwd = admin
#信息展示面板密码
log_max_days = 7
#最多保存多少天日志
privilege_token = frp888
#特权模式认证密钥
privilege_allow_ports = 1-65535
#端口白名单，为了防止端口被滥用，可以手动指定允许哪些端口被使用
max_pool_count = 100
#每个内网穿透服务限制最大连接池上限，避免大量资源占用，可自定义
authentication_timeout = 0
#frpc 所在机器和 frps 所在机器的时间相差不能超过 15 分钟，因为时间戳会被用于加密验证中，防止报文被劫持后被其他人利用,单位为秒，默认值为 900，即 15 分钟。如果修改为 0，则 frps 将不对身份验证报文的时间戳进行超时校验。国外服务器由于时区的不同，时间会相差非常大，这里需要注意同步时间或者设置此值为0
log_file = frps.log
log_level = info

```