---
title: redis降级后启动报错
date: 2016-09-28 11:06:00
categories: linux
tags: [redis,error]
---
### 报错过程及信息
主机上安装了redis3.x版本，后来降级到2.8.x版本，发现直接执行"redis-server 配置文件"无报错，但使用启动脚本启动时报错如下
"# Can't handle RDB format version 7
# Fatal error loading the DB: Invalid argument. Exiting."

### 参考资料
https://gitlab.com/gitlab-org/omnibus-gitlab/issues/1401
https://gitlab.com/gitlab-org/omnibus-gitlab/commit/824530c76b54f6f9216fde2aaead09dd4584e3cb
从以上资料中可大概总结到，此问题是因为不同版本的redis采用了不同的rdb版本，预计在3.2.2版本之后解决此问题。
**参考解决办法**
1、删除dump.rdb文件，在/etc/redis/下找到此文件，但删除后错误仍在
2、去配置文件修改dumpfile名称，dbfilename dump2.rdb，此方法可解决