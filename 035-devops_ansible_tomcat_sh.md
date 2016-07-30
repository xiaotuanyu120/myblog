---
title: ansible roles进化1、tomcat安装脚本
date: 2016-07-30 16:01:00
categories: devops
tags: [devops,ansible,java]

---
## 脚本所需文件
包括jre7u80、tomcat8.5、主要安装脚本、java环境脚本
``` bash
tree
.
├── apache-tomcat-8.5.4.tar.gz
├── install_tomcat.sh
├── java-env-7u80.sh
└── jre-7u80-linux-x64.tar.gz
```

<!--more-->

## 脚本内容
**主要安装脚本"install_tomcat.sh"**

``` bash
# author: zack
# date: 2016-07-30
# for auto deploy jre and tomcat

# ENV SETTING
export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
basedir=`readlink -f $(dirname $0)`
# setting for user customize
jre_tar=jre-7u80-linux-x64.tar.gz
jre_folder=jre1.7.0_80
sys_jrenv_script=java-env-7u80.sh
tomcat_tar=apache-tomcat-8.5.4.tar.gz
tomcat_folder=apache-tomcat-8.5.4

jre_base=/usr/local/$jre_folder
tomcat_base=/usr/local/tomcat

# DEPLOY JRE
[ -d "${basedir}/${jre_folder}" ] && rm -rf ${basedir}/${jre_folder}
tar zxf $jre_tar
[ -d "$jre_base" ] && rm -rf $jre_base
mv ${basedir}/${jre_folder} $jre_base

# JRE ENV SETTING
sys_jrenv=/etc/profile.d/java-env.sh
/bin/cp ${basedir}/${sys_jrenv_script} $sys_jrenv
sed -i "s#%JREBASE%#${jre_base}#g" $sys_jrenv
source $sys_jrenv


# DEPLOY TOMCAT
[ -d "${basedir}/${tomcat_folder}" ] && rm -rf ${basedir}/${tomcat_folder}
tar zxf $tomcat_tar
[ -d "$tomcat_base" ] && rm -rf $tomcat_base
mv ${basedir}/${tomcat_folder} $tomcat_base
# prepare daemon script
/bin/cp ${tomcat_base}/bin/catalina.sh /etc/init.d/tomcat
sed -i "2a # chkconfig: 2345 63 37" /etc/init.d/tomcat
sed -i "3a . /etc/init.d/functions" /etc/init.d/tomcat
sed -i "4a JAVA_HOME=${jre_base}" /etc/init.d/tomcat
sed -i "5a CATALINA_HOME=${tomcat_base}" /etc/init.d/tomcat
chmod 755 /etc/init.d/tomcat
```


**java环境脚本"java-env-7u80.sh"**

``` bash
JAVA_HOME=%JREBASE%
JRE_HOME=${JAVA_HOME}/jre
PATH=$PATH:${JAVA_HOME}/bin:${JRE_HOME}/bin
CLASSPATH=${JAVA_HOME}/lib:${JRE_HOME}/lib
```
此脚本其实是一个模版脚本，%JREBASE%需要在主脚本中进行修改之后，复制到/etc/profile.d/中在系统启动时进行初始化使用