---
title: 安装andriod虚拟机genymotion
date: 2016-08-29 21:38:00
categories: fedora
tags: [fedora, genymotion, android]
---
## 安装virtualbox
在virtualbox官网下载最新virtualbox的rpm包，并安装
``` bash
sudo dnf install ./VirtualBox-5.1-5.1.4_110228_fedora24-1.x86_64.rpm
```

<!--more-->

## 安装genymotion

安装虚拟化环境底包
``` bash
sudo dnf groupinstall "Virtualization"
sudo dnf install kernel-devel gstreamer-plugins-base libpng12
```

Genymotion还需要libjpeg8和libdouble-conversion.
可以通过下面的链接下载它们 [RPMPbone](http://rpm.pbone.net/index.php3/stat/4/idpl/23363141/dir/opensuse/com/libjpeg8-8.4.0-6.23.x86_64.rpm.html) and [RPMFind](https://www.rpmfind.net/linux/rpm2html/search.php?query=libdouble-conversion.so.1%28%29%2864bit%29).

在genymotion官网注册账号，并免费下载linux版本，并安装
``` bash
cd /home/zack/Downloads
sudo mv genymotion-2.7.2-linux_x64.bin /opt
cd /opt
sudo chmod +x genymotion-2.7.2-linux_x64.bin
sudo /opt/genymobile/genymotion/genymotion
```

## 解决报错
- 启动genymotion时提示，无法加载virtualbox
- 检查virtualbox，发现正常安装，但是<code>VirtualBox restart</code>时报错，没有加载virutalbox kernel mode
- 于是按照提示执行
``` bash
sudo /sbin/vboxconfig
# 报错，提示安装kernel-core-devel
# 检查kernel-devel，发现版本时4.6，而uname -r出现的信息时4.5.5
# 原来错误就是安装的kernel-devel的版本和实际的linux kernel不匹配
```

**解决办法：**
``` bash
sudo dnf install kernel-devel-$(uname -r)

# 此处重装virtualbox的rpm包
sudo dnf reinstall ./VirtualBox-5.1-5.1.4_110228_fedora24-1.x86_64.rpm

# 如果依然有问题，执行以下命令来重新编译vboxkernel module
sudo /sbin/vboxconfig
```
