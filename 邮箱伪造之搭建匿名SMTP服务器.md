# 邮箱伪造之搭建匿名SMTP服务器
使用postfix搭建匿名SMTP服务器

环境：CentOS7   

## 1、安装postfix**

#安装postfix

yum install postfix

## 2、修改main.cf配置文件**

```
vi /etc/postfix/main.cf

# 75行:设置myhostname     
myhostname = mail.test.com     

\# 83行: 设置域名     
mydomain = test.com     

\# 99行: 设置myorigin    
myorigin = $mydomain     

\# 116行: 默认是localhost，修改成all    
inet_interfaces = all     

\# 119行: 推荐ipv4，如果支持ipv6，则可以为all     
inet_protocols = ipv4     

\# 165行: 设置mydestination    
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain     

\# 264行: 指定内网和本地的IP地址范围     
mynetworks = 192.168.0.0/16，127.0.0.0/8    

\# 419行: 取消注释，邮件保存目录    
home_mailbox = Maildir/     

\# 572行: 取消注释，设置banner。    
smtpd_banner = $myhostname ESMTP
```
## 3、启动postfix服务**

```
systemctl start postfix
```

#关闭防火墙
```
systemctl stop firewalld.service
```
## 使用Python脚本发送伪造邮件
使用第三方邮件服务器，往往会受限于SMTP服务商的限制，但也有一定的好处，这些权威的邮件服务商的地址往往会被大部分邮件服务商加入白名单。不同的邮箱系统，接收邮件安全策略是不同；不同的SMTP服务商，发送邮件的限制也是不一样，具体会发生什么样的化学作用，还需具体进一步去测试。

国内主流的邮箱有：QQ邮箱（qq和foxmail）、网易邮箱（包括163、126和yeah邮箱）、新浪邮箱、搜狐闪电邮箱、移动139邮箱、电信189邮箱等等。
国外的第三方SMTP服务商：SendGrid、mailgun

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
import smtplib
from email.mime.text import MIMEText
from email.header import Header
sender = 'admin@test.com'

receivers = ['a*****t@163.com']

message = MIMEText('Just for test', 'plain', 'utf-8')
message['From'] = Header("admin")   # 发送者
message['To'] =  Header("test")        # 接收者

subject = 'SMTP 邮件测试'
message['Subject'] = Header(subject, 'utf-8')

try:
    smtpObj = smtplib.SMTP('localhost')
    smtpObj.sendmail(sender, receivers, message.as_string())
    print "邮件发送成功"
except smtplib.SMTPException:
    print "Error: 无法发送邮件"
```


## 通过telnet使用smtp协议发送邮件

telnet localhost 25

[![.png](_v_images/20200521110001410_1674.png "20.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/05/20.png) 

测试邮箱成功接收到邮件

[![.png](_v_images/20200521110000805_26633.png "21.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/05/21.png)

## 使用mail发送邮件

#安装mailx

`yum install mailx  
`
#发送邮件测试

`echo "email content" | mail -s "title" a*****t@163.com`

[![.png](_v_images/20200521110413321_9490.png "19-1024x300.png")](https://secpulseoss.oss-cn-shanghai.aliyuncs.com/wp-content/uploads/2020/05/19.png) 

查看邮件发送记录：

```
tail -f /var/log/maillog

Apr 28 09:27:14 centos postfix/smtpd\[108012\]: connect from localhost\[127.0.0.1\]

Apr 28 09:27:15 centos postfix/smtpd\[108012\]: 0170D403916: client=localhost\[127.0.0.1\]

Apr 28 09:27:15 centos postfix/cleanup\[108015\]: 0170D403916: [message-id=<20200428012715.0170D403916@mail.abc.com>](mailto:message-id=%3C20200428012715.0170D403916@mail.abc.com%3E)

Apr 28 09:27:15 centos postfix/qmgr\[39469\]: 0170D403916: from=<root@test.com>, size=716, nrcpt=1 (queue active)

Apr 28 09:27:15 centos postfix/smtpd\[108012\]: disconnect from localhost\[127.0.0.1\]

Apr 28 09:27:15 centos postfix/smtp\[108016\]: connect to mx3.qq.com\[240e:ff:f101:10::127\]:25: Network is unreachable

Apr 28 09:27:16 centos postfix/smtp\[108016\]: 0170D403916: to=<a*****t@163.com>, relay=mx3.qq.com\[58.251.110.111\]:25, delay=1.5, delays=0.03/0.03/0.37/1, dsn=2.0.0, status=sent (250 Ok: queued as )

Apr 28 09:27:16 centos postfix/qmgr\[39469\]: 0170D403916: removed
```

从邮件日志看到status=sent，确认邮件发送成功。




