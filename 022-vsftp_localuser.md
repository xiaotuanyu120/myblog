---
title: centos下vsftpd搭建(yum+本地用户)
date: 2016-07-16 14:12:00
categories: ftp
tags: [linux,ftp,vsftp]

---
## 安装vsftp
``` bash
yum install vsftpd
```

## 配置vsftp
打开/etc/vsftpd/vsftpd.conf
``` bash
# 关闭匿名用户
anonymous_enable=NO

# 确保本地用户可以登录
local_enable=YES

# 限制用户访问指定目录以外的目录
chroot_local_user=YES

# 配置ftp目录
local_root=/var/vsftp
```

<!--more-->

## 准备用户和ftp目录
``` bash
useradd vsftp
passwd vsftp

# 按照上面local_root的配置创建相应目录
mkdir /var/vsftp
chown :vsftp /var/vsftp
chmod 775 /var/vsftp

# 检查目录权限和属组
ll -d /var/vsftp
drwxrwxr-x. 2 root vsftp 4096 Jul 16 14:59 /var/vsftp
```

## 配置selinux和iptables
``` bash
setenforce 0
vim /etc/selinux/config
	******************
    SELINUX=disabled
    ******************
iptables -I INPUT -p tcp --dport 21 -j ACCEPT
iptables -I OUTPUT -p tcp --sport 20 -j ACCEPT
vim /etc/sysconfig/iptables-config
	******************
    IPTABLES_MODULES="ip_conntrack_ftp"
    ******************
service iptables save
service iptables restart
```