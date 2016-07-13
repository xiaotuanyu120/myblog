---
title: python获取本机的ip地址
date: 2016-07-13 15:30:00
categories: python
tags: python
---
## socket模块获取ip地址过程详解
``` python
import socket
```
> socket模块提供了socket操作和相关功能

<!--more-->

``` python
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```
> socket.socket()是用来返回一个socket对象
> 第一个参数：
> * socket.AF_UNIX
> * socket.AF_INET (默认)
> * socket.AF_INET6
>   指定了连接的地址和协议族
>
> 第二个参数：
> * socket.SOCK_STREAM (默认)
> * socket.SOCK_DGRAM
> * 其他几种基本没用
>   指定了连接的类型

``` python
s.connect(("8.8.8.8", 80))
```
> connect方法是创建一个到目标地址的连接
> 之所以要创建一个连接，是因为localhost可能会有多个网卡和多个ip地址（例如127.0.0.1），连接创建之后，真正去连接外网的ip就会被筛选出来了

``` python
print(s.getsockname()[0])
```
> getsockname方法返回socket连接的地址和端口

``` python
s.close()
```
> 最后关闭此连接

## 本机ip地址获取，实际应用写法
``` python
import socket


def get_ip():
    ip = [(s.connect(('8.8.8.8', 80)), s.getsockname()[0], s.close()) for s in [socket.socket(socket.AF_INET, socket.SOCK_DGRAM)]][0][1]
    return ip
```