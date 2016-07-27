---
title: shell脚本：防止ssh暴力破解系统
date: 2016-07-27 14:25:00
categories: shell
tags: shell

---
**脚本需求**
需要解决两个问题
1、对服务器现存用户的密码暴力破解
2、对服务器用户名称的暴力破解

<!--more-->

**实现思路**
使用到两个文件：
- 用户登录安全日志：/var/log/secure
- 访问登录限制配置文件：/etc/hosts.deny

在secure文件中分析两种情况的报错信息，筛选出来，并统计出信息中ip出现的次数
将超过指定次数的ip增加到hosts.deny

**脚本内容**
```bash
#!/bin/bash

# author: zpw
# date: 2016-07-27

deny_file=./test.txt
ip_deny=`/bin/grep 'Failed password' /var/log/secure\
         |/bin/grep -oE '[[:digit:]]+\.[[:digit:]]+\.[[:digit:]]+\.[[:digit:]]+'\
         |/bin/awk '{++ip[$1]} END {for(i in ip) if(ip[i] > 2) print i}'`

for i in $ip_deny
do
    ip_exist=`grep "$i" $deny_file`
    /usr/bin/test -z "$ip_exist" && echo "sshd:$i" >> $deny_file
done
```
实际生产环境中，只要把deny_file变量和if判断中的次数修改成合适的文件和值就可以