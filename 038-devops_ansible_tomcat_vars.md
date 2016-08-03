---
title: ansible roles进化3、tomcat安装之变量
date: 2016-08-03 15:14:00
categories: devops
tags: [devops, ansible, tomcat]

---
## 前情回顾
第一阶段：我们创建了“shell脚本+文件”来一键安装tomcat
第二阶段：我们使用ansible来简单的通过拷贝shell脚本和文件到目地主机，并且通过执行shell脚本的方式完成了tomcat的一键安装

本次，我们需要把在shell脚本中原本的变量设定，从shell脚本中抽离出来，使用ansible的roles变量管理代替。

<!--more-->

## 文件结构
第一级目录：site.yml主文件，roles目录
第二级目录：tomcat_install角色目录(在site.yml文件中指定此角色)
第三级目录：
- files目录:tomcat安装文件、jre安装文件
- handlers目录：未使用
- tasks目录： main.yml指定了拷贝任务（此次增加了template任务）和执行脚本的任务
- templates目录：将上次的shell脚本和java环境脚本拷贝过来将作为Jinja2模版，变量在vars目录中指定
- vars目录：指定专属tomcat_install角色的变量

``` bash
tree
.
├── hosts
├── README.md
├── roles
│   └── tomcat_install
│       ├── files
│       │   ├── apache-tomcat-8.5.4.tar.gz
│       │   └── jre-7u80-linux-x64.tar.gz
│       ├── handlers
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   ├── install_tomcat.sh
│       │   └── java-env-7u80.sh
│       └── vars
│           └── main.yml
└── site.yml

7 directories, 9 files
```

## 与上篇[进化2](http://blog.xiao5tech.com/2016/07/30/036-devops_ansible_tomcat_roles/)的不同之处
使用了vars目录中的变量去修改了templates目录中的安装脚本和环境设定脚本，将原本在shell脚本中指定的变量设定抽离到了ansible中

## 关键文件内容
**hosts文件**
``` bash
[java]
10.10.180.4
10.10.180.6
```
与上次相比未修改，仅用来指定了远程主机分组和ip信息
关于inventory中设定变量，参考：[inventory变量](http://docs.ansible.com/ansible/intro_inventory.html)

**site.yml文件内容**
``` bash
---
- hosts: java
  remote_user: root

  roles:
    - tomcat_install
```
与前次未更改，只是用来指定了tomcat_install这个角色，ansible会自动去寻找roles目录下的tomcat_install目录，然后执行该目录下的tasks目录下的main.yml

**tasks/main.yml文件内容**
``` bash
---
- name: copy jre
  copy: src=jre-7u80-linux-x64.tar.gz dest=/usr/local/src

- name: copy tomcat
  copy: src=apache-tomcat-8.5.4.tar.gz dest=/usr/local/src

- name: copy java env init scripts
  template: src=java-env-7u80.sh dest=/etc/profile.d/java-env.sh

- name: initial java env vars
  shell: source /etc/profile.d/java-env.sh

- name: copy tomcat installation scripts
  template: src=install_tomcat.sh dest=/usr/local/src

- shell: sh ./install_tomcat.sh
  args:
    chdir: /usr/local/src/
```
此文件是tomcat_install角色的主要任务文件
- 文件拷贝，jre安装文件和tomcat安装文件
- template文件的变量内容填充和拷贝，变量在vars/main.yml中获得
- source了java变量的脚本文件
- 执行了tomcat安装脚本


**templates/install_tomcat.sh文件内容**
``` bash
{% raw %}
cat roles/tomcat_install/templates/install_tomcat.sh
****
#!/bin/bash

# author: zack
# date: 2016-07-30
# for auto deploy jre and tomcat

# ENV SETTING
export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
basedir=`readlink -f $(dirname $0)`
# setting for user customize
jre_tar={{ tomcat85.jre7u80.jre_tar }}
jre_folder={{ tomcat85.jre7u80.jre_folder }}
tomcat_tar={{ tomcat85.tomcat_tar }}
tomcat_folder={{ tomcat85.tomcat_folder }}

jre_base=/usr/local/$jre_folder
tomcat_base=/usr/local/tomcat

# DEPLOY JRE
[ -d "${basedir}/${jre_folder}" ] && rm -rf ${basedir}/${jre_folder}
tar zxf $jre_tar
[ -d "$jre_base" ] && rm -rf $jre_base
mv ${basedir}/${jre_folder} $jre_base

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
****
{% endraw %}
```
{% raw %}
只改动了"# setting for user customize"这些变量设定部分
拿{{ tomcat85.jre7u80.jre_tar }}举例，{{}}代表了中间的内容是变量，此变量意味着一个tomcat85的dict，包含一个jre7u80的子dict，拿的是其jre_tar这个key对应的值
{% endraw %}


**templates/java-env-7u80.sh文件内容**
``` bash
{% raw %}
cat roles/tomcat_install/templates/java-env-7u80.sh
****
JAVA_HOME={{ tomcat85.jre7u80.java_home }}
JRE_HOME=${JAVA_HOME}/jre
PATH=$PATH:${JAVA_HOME}/bin:${JRE_HOME}/bin
CLASSPATH=${JAVA_HOME}/lib:${JRE_HOME}/lib
****
{% endraw %}
```
和上面一样，此处只修改了一个变量设定

**vars/main.yml文件内容**
``` bash
---
tomcat85:
  tomcat_tar: apache-tomcat-8.5.4.tar.gz
  tomcat_folder: apache-tomcat-8.5.4
  jre7u80:
    java_home: /usr/local/jre1.7.0_80
    jre_folder: jre1.7.0_80
    jre_tar: jre-7u80-linux-x64.tar.gz
```
此处按照yaml文件语法，是一个dict套dict的写法，指定了变量设定


## 执行结果
``` bash
ansible-playbook -i hosts site.yml

PLAY [java] ********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.6]
ok: [10.10.180.4]

TASK [tomcat_install : copy jre] ***********************************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : copy tomcat] ********************************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : copy java env init scripts] *****************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : initial java env vars] **********************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : copy tomcat installation scripts] ***********************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : command] ************************************************
changed: [10.10.180.6]
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=7    changed=6    unreachable=0    failed=0
10.10.180.6                : ok=7    changed=6    unreachable=0    failed=0
```
