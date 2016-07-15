---
title: dede修改后台admin密码
date: 2016-07-15 15:34:00
categories: other
tags: [cms,dede]

---
## 查找后台url
- 一般默认后台都是域名/dede
- 进入web目录，查看是否修改过dede目录的名称，若换过，就换成域名/新目录名称

> 什么是web目录？
> 就是web服务器配置文件中，指定的web目录路径，以nginx举例是在server块中的root关键字指定的目录
```
server {
    ...
    root /data/www/dede/;
    ...
}
```

## 登录数据库，重置admin密码为123456
``` bash
mysql -u root -p
mysql> use 数据库名称
mysql> update dede_admin set pwd='c3949ba59abbe56e057f' where userid='admin';
mysql> flush tables;
```