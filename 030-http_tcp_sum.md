---
title:
date:
categories:
tags":

---
## 查看http连接及其状态的原理
netstat工具是一个可以查看网络连接详细情况的工具
**命令:**<code>netstat -tn</code>
- -t参数代表了只查看tcp连接
- -n参数代表了数字化显示域名，既显示其ip

``` bash
netstat -nt|head
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 103.228.28.230:3306         103.228.28.230:53608        ESTABLISHED 
tcp        0      0 103.228.28.230:3306         61.14.162.11:42321          ESTABLISHED 
tcp        0      0 103.228.28.230:22           61.147.73.28:3912           ESTABLISHED 
tcp        0      0 103.228.28.230:22           61.14.162.11:1429           ESTABLISHED 
tcp        0     64 103.228.28.230:22           61.14.162.7:52327           ESTABLISHED 
tcp        0      0 103.228.28.230:22           61.14.162.11:40227          ESTABLISHED 
tcp        0      0 103.228.28.230:3306         61.14.162.11:6266           ESTABLISHED 
tcp        0      0 103.228.28.230:3306         61.14.162.11:41942          ESTABLISHED 
```
输出结果中最后一段就是tcp连接的状态

两种筛选办法
 - 1.利用sort和uniq
``` bash
netstat -nt | grep "^tcp"|awk '{print $NF}'|sort -nr|uniq -c
      1 TIME_WAIT
     18 ESTABLISHED
```
 - 2.利用awk的数组
``` bash
netstat -nt | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,state[key]}'
ESTABLISHED 18
```
