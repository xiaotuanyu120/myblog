---
title: python模块easysnmp
date: 2016-07-08 11:22:00
categories: devops
tags: [devops,python,snmp]
---
## 什么是easysnmp？
&emsp;&emsp;easysnmp是官方[Net-SNMP Python Bindings](http://net-snmp.sourceforge.net/wiki/index.php/Python_Bindings)的一个分支，致力于让net-snmp更python。

## easysnmp的特点
* 传统的net-snmp很优秀，但是太不python和缺少合适的unit去测试和文档化；
* [PySNMP](http://pysnmp.sourceforge.net/)是用纯净的python创造的，然而它有很大的性能问题。easysnmp大概是其的4倍速度；
  <!--more-->

## 安装easysnmp

### 安装环境底包
```
yum install gcc python-devel net-snmp-devel -y
```

### 安装Net-SNMP 5.7.x，easysnmp目前只支持这个版本
``` bash
wget http://tenet.dl.sourceforge.net/project/net-snmp/net-snmp/5.7.3/net-snmp-5.7.3.tar.gz
tar zxf net-snmp-5.7.3.tar.gz
cd net-snmp-5.7.3

./configure
# configure的过程中会有几次交互，默认或者按照自己情况配置即可

```

### 安装easysnmp
```
pip install easysnmp
```

## easysnmp import error

### 错误信息：
``` python
In [1]: import easysnmp
---------------------------------------------------------------------------
ImportError                               Traceback (most recent call last)
<ipython-input-1-d560d4059750> in <module>()
----> 1 import easysnmp

/root/env27/lib/python2.7/site-packages/easysnmp/__init__.py in <module>()
----> 1 from .easy import (  # noqa
      2     snmp_get, snmp_set, snmp_set_multiple, snmp_get_next, snmp_get_bulk,
      3     snmp_walk
      4 )
      5 from .exceptions import (  # noqa

/root/env27/lib/python2.7/site-packages/easysnmp/easy.py in <module>()
      1 from __future__ import unicode_literals
      2
----> 3 from .session import Session
      4
      5

/root/env27/lib/python2.7/site-packages/easysnmp/session.py in <module>()
      6 # Don't attempt to import the C interface if building docs on RTD
      7 if not os.environ.get('READTHEDOCS', False):  # noqa
----> 8     from . import interface
      9
     10 from .exceptions import (

ImportError: /root/env27/lib/python2.7/site-packages/easysnmp/interface.so: undefined symbol: netsnmp_transport_config_compare

```

### 排查过程
网上大部分人说是版本没有安装5.7.x版本的原因，但博主的问题显然不在这里

去看了一下snmpget命令的调用库文件
```
ldd /root/env27/bin/python
        linux-vdso.so.1 =>  (0x00007fffeed54000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00000031c8200000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00000031c7a00000)
        libutil.so.1 => /lib64/libutil.so.1 (0x00000031c9a00000)
        libz.so.1 => /lib64/libz.so.1 (0x00000031c8e00000)
        libm.so.6 => /lib64/libm.so.6 (0x00000031c8a00000)
        libc.so.6 => /lib64/libc.so.6 (0x00000031c7e00000)
        /lib64/ld-linux-x86-64.so.2 (0x00000031c7600000)
```
发现并没有调用snmp的库文件

### 解决办法
```
echo "export LD_PRELOAD=/usr/local/lib/libnetsnmp.so" /etc/profile
source /etc/profile
ldd /root/env27/bin/python
        linux-vdso.so.1 =>  (0x00007fff8c5ff000)
        /usr/local/lib/libnetsnmp.so (0x00007f102458c000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00000031c8200000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00000031c7a00000)
        libutil.so.1 => /lib64/libutil.so.1 (0x00000031c9a00000)
        libz.so.1 => /lib64/libz.so.1 (0x00000031c8e00000)
        libm.so.6 => /lib64/libm.so.6 (0x00000031c8a00000)
        libc.so.6 => /lib64/libc.so.6 (0x00000031c7e00000)
        librt.so.1 => /lib64/librt.so.1 (0x00000031c8600000)
        libcrypto.so.10 => /usr/lib64/libcrypto.so.10 (0x0000003e01a00000)
        /lib64/ld-linux-x86-64.so.2 (0x00000031c7600000)
```