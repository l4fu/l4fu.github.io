# Centos8如何安装Xfce桌面 
## 安装xfce4

这里使用的xfce4相关的安装包都是从Fedora28系统中下载的，装在Centos8中或许有些不稳定，目前发现装上之后Firewalld有问题，如果不妨碍使用，可以禁用Firewalld。

生产环境就不要这样玩了，本实验纯属测试一下而已。

Xfce4相关的安装包我已经放在网盘里了： [https://share.weiyun.com/VqEEZio1](https://share.weiyun.com/VqEEZio1)， 下载到本地，然后上传到Centos8中。

```
# 安装lrzsz，将下载好的 压缩包上传到系统中。 
[root@localhost ~\]# dnf -y install lrzsz tar 
[root@localhost ~\]# rz # 解压文件，并进入文件夹，开始安装 
[root@localhost ~\]# tar xvf Xfce_fd28.tar.gz  
[root@localhost ~\]# cd Xfce 
[root@localhost Xfce\]# dnf install -y * --skip-broken --nobest

```
![Centos8如何安装Xfce桌面Centos8如何安装Xfce桌面](_v_images/20200902164600795_26111.png "Centos8如何安装Xfce桌面Centos8如何安装Xfce桌面")

![Centos8如何安装Xfce桌面Centos8如何安装Xfce桌面](_v_images/20200902164600558_10781.png "Centos8如何安装Xfce桌面Centos8如何安装Xfce桌面")

**设置开机启动到图形界面**
```
#开机启动lightdm显示管理器 
[root@localhost ~]# systemctl enable lightdm 
# 默认启动图形界面 
[root@localhost ~]# systemctl set-default graphical.target  
Removed /etc/systemd/system/default.target. Created symlink /etc/systemd/system/default.target ¡ú /usr/lib/systemd/system/graphical.target. 
关闭防火墙 
[root@localhost ~]# systemctl disable firewalld Removed /etc/systemd/system/multi-user.target.wants/firewalld.service. 
Removed /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```