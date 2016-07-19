---
title: 面试题汇总-02
date: 2016-07-19 10:45:00
categories: interview
tags: [interview]

---
## 问题1
**问题列表：**
1. 线上MYSQL数据库，几千万条记录的大表，如果执行DDL操作会出现什么问题？
2. DDL的执行过程？
3. 大表如何加索引，加字段？

### 什么是SQL？
&emsp;&emsp;SQL是structured querry language的缩写，代表结构化查询语言。

### SQL的组成？
**SQL分为四种主要的语句**
- DML (Data Manipulation Language) - 数据操作语句
包括SELECT,INSERT,UPDATE,DELETE
- DDL (Data Definition Language) - 数据定义语句
包括CREATE,ALTER,DROP
- DCL (Data Control Language) - 数据控制语句
包括GRANT,REVOKE
- TCL (Transaction Control Language) - 交互控制语句
包括BEGIN Transaction,COMMIT Transaction,ROLLBACK Transaction

**DDL语句的执行过程？**

**执行过程**
DDL执行时，它会先commit和结束目前所有的数据库操作，它们的改变将变成永久的
然后执行DDL语句，若成功，则commit，若失败则rollback

**DDL执行过程的伪代码(帮助理解)**
```
BEGIN
    commit;
    DDL语句的解析，包括语法解析和语义解析;
    BEGIN
        执行DDL语句;
        commit;
    exception
        when others then
        rollback;
    END;
END;
```

> **扩展链接：**[What Happens to the Current Transaction If a DDL Statement Is Executed?](http://dba.fyicenter.com/faq/mysql/Transaction-Committed-when-DDL-Statement-Executed.html)

## 问题2
**问题列表：**
1. Redis的安全策略一般会考虑哪些？
2. IDC节点公网出口通过NAT访问公网一般需要做哪些配置？

### 什么是redis？
REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。
Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。
它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Map), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

### redis安全
扩展阅读：[官方安全文档](http://redis.io/topics/security)
- 不要直接暴露给internet，最好部署在安全的内网；
- 在web的应用中，对不信任的client，必须要有web程序的中介；
- 在一般的程序应用中，对不信任的client，必须要有ACL,用户输入等相关信息的验证
- 若是单点实例，可以直接部署在client上，在redis.conf中配置"bind 127.0.0.1"；
- 在3.2.0以后，若redis实例绑定了公网ip，使用的是默认的配置，另外没有密码安全限制的情况下，redis会进入protected mode。此模式下，只响应lo网卡的请求，其他请求会返回错误，说明原因和配置提示
- redis.conf可配置认证
- 通过internet或不信任网络访问的情况下，可采用ssl代理加密，官方推荐[spiped](http://www.tarsnap.com/spiped.html)
- 可禁用某些指定的命令
- 小心对待用户的输入，正确的配置用户输入的集合，使用qsort算法来防御用户输入数据从内部攻破redis

## 问题3
**问题列表：**
1. 概述下PYTHON装饰器？
扩展链接：[装饰器最好的解读](http://stackoverflow.com/questions/739654/how-can-i-make-a-chain-of-function-decorators-in-python)

### 个人总结
装饰器可用于在不改变当前函数的情况下，实现扩展该函数的目地，采用的方法是创建一个拥有内置函数的函数。

## 问题4
**问题列表：**
1. saltstack的架构设计有几种？
2. puppet条件语句有几种？

**未研究**，**待研究**

