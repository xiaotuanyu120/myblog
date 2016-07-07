---
title: RRDtool初探-安装
date: 2016-0706 20:55:00
categories: devops
tags: [devops,rrdtool]
---
## python环境安装

### python2.7安装
``` bash
yum install zip zip-devel openssl openssl-devel -y
wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
tar zxf Python-2.7.12.tgz
cd Python-2.7.12
./configure --prefix=/usr/local/py27/
vim Modules/Setup
    ********************************
    ## 把下面一行反注释
    zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz
    ********************************
make
make install
vim /etc/profile
    ********************************
    ## 在最后增加
    export PATH=$PATH:/usr/local/py27/bin
    ********************************
source /etc/profile
```
<!--more-->
### pip安装
``` bash
wget https://bootstrap.pypa.io/get-pip.py
/usr/local/py27/bin/python2.7 get-pip.py
```

### virtualenv安装及创建虚拟环境env27
``` bash
pip install virtualenv
virtualenv env27 -p /usr/local/py27/bin/python2.7
source ./env27/bin/activate
```

## RRDtool 安装
``` bash
yum install pango-devel libxml2-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker -y
wget  http://oss.oetiker.ch/rrdtool/pub/rrdtool-1.6.0.tar.gz
tar zxf rrdtool-1.6.0.tar.gz
cd rrdtool-1.6.0
./configure --prefix=/usr/local/rrdtool
make
make install
ln -s /usr/local/rrdtool/lib/librrd* /usr/lib64/
```

## RRDtool python模块安装
``` bash
yum install gcc cmake make python-devel cairo-devel libxml2 libxml2-devel pango-devel pango libpng-devel freetype freetype-devel libart_lgpl-devel -y
pip install python-rrdtool
```