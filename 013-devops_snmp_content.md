---
title: SNMP初识
date: 2016-07-08 11:10:00
categories: devops
tags: [devops,snmp]
---
## 什么是snmp？
Simple Network Management Protocol (SNMP) is a popular protocol for network management. It is used for collecting information from, and configuring, network devices, such as servers, printers, hubs, switches, and routers on an Internet Protocol (IP) network.
<!--more-->
## snmp存在的版本，及其不同
* SNMP version 1:
> * the oldest flavor.
> * Easy to set up – only requires a plaintext community.
> 
>   `downsides`
> * it does not support 64 bit counters, only 32 bit counters
> * it has little security.

* SNMP version 2c:
> * v2c is identical to version 1, except it adds support for 64 bit counters.
> * Most devices support snmp V2c nowadays, and generally do so automatically.
> 
>   `downsides`
> * no

* SNMP version 3:
> * adds security to the 64 bit counters.
> * SNMP version 3 adds both encryption and authentication, which can be used together or separately. 
> * Setup is more complex than just defining a community string
