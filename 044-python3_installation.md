---
title: python3.5.2 installation on centos6.5
date: 2016-08-11 21:11:00
categories: python
tags: python
---
## 安装python3
``` bash
yum groupinstall "Development tools" -y
yum install python-devel openssl-devel

wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tar.xz
tar -Jxf Python-3.5.2.tar.xz
cd Python-3.5.2
./configure --prefix=/usr/local/python3
make && make install
```

会自动安装pip和setuptools等包