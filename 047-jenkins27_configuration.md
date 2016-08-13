---
title: jenkins-基本配置
date: 2016-08-12 15:42:00
categories: jenkins
tags: [jenkins,java,linux]
---
### 安全性配置
点击侧边栏 -> manage jenkins -> Configure Global Security
![j1](http://blog.xiao5tech.com/images/047-j1.PNG)
确保以下选项被勾选
- enable security 启用安全模块
- Jenkins’ own user database  使用jenkins自有的用户信息database(禁止注册user)
- Logged-in users can do anything  登录者有全部权限

<!--more-->

### 插件升级及安装
点击侧边栏 -> manage jenkins -> Manage Plugins
![j2](http://blog.xiao5tech.com/images/047-j2.PNG)
此处可执行多种插件操作
- 升级查询及升级
- 卸载
- 搜索插件及安装

### jdk+maven+git准备及环境设定
> **jenkins2.7版本的不同**
这是我接触的第一个jenkins版本，与网上教程不同的是，2.7版本中的jdk、maven、ant等工具的配置从configure system中转移到了Global Tool Configuration中

点击侧边栏 -> manage jenkins -> Global Tool Configuration
![j3](http://blog.xiao5tech.com/images/047-j3.PNG)

**按照下面操作去获取应该要填写的信息**
- jdk

``` bash
# 找出jdk的home信息
readlink -f /usr/bin/java
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.el6_8.x86_64/jre/bin/java
# 所以说，JAVA_HOME的值应该是/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.el6_8.x86_64/jre
```
![j4](http://blog.xiao5tech.com/images/047-j4.PNG)
需要注意的是，不要勾选install automatically

- git

![j5](http://blog.xiao5tech.com/images/047-j5.PNG)
如果git命令是在linux系统的PATH变量中，可以只填写git，也可以写git命令的绝对路径
同样需要注意，不要勾选install automatically

- maven

``` bash
# 安装maven，并找出maven的basedir
vim /etc/profile
**************************************
# 在最后添加
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.el6_8.x86_64/jre
export PATH=$PATH:/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.el6_8.x86_64/jre/bin:/usr/local/maven/bin
**************************************
source /etc/profile

wget http://www-eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar zxf apache-maven-3.3.9-bin.tar.gz
mv apache-maven-3.3.9 /usr/local/maven

mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-11T00:41:47+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_101, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.101-3.b13.el6_8.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-431.el6.x86_64", arch: "amd64", family: "unix"

# 从输出信息知道，maven_home是/usr/local/maven
```
![j6](http://blog.xiao5tech.com/images/047-j6.PNG)

同样需要注意，不要勾选install automatically

至此，基本的maven git jdk就配置配置完毕
