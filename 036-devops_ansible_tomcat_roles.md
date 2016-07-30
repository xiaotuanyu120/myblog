---
title: ansible roles进化2、ansible远程执行tomcat安装脚本
date: 2016-07-30 16:53:00
categories: devops
tags: [devops,ansible,tomcat]

---
## 文件结构
第一级目录：site.yml主文件，roles目录
第二级目录：tomcat_install角色目录(在site.yml文件中指定此角色)
第三级目录：
- files目录：tomcat源文件、jre源文件、tomcat安装脚本、jre环境脚本模版
- handlers目录：未使用
- tasks目录：main.yml指定了拷贝任务和执行脚本的任务

``` bash
tree .
.
├── hosts
├── roles
│   └── tomcat_install
│       ├── files
│       │   ├── apache-tomcat-8.5.4.tar.gz
│       │   ├── install_tomcat.sh
│       │   ├── java-env-7u80.sh
│       │   └── jre-7u80-linux-x64.tar.gz
│       ├── handlers
│       └── tasks
│           └── main.yml
└── site.yml

```

<!--more-->

## 与直接脚本执行的区别
- 仅仅是用ansible拷贝了文件过去，然后执行了脚本，其他与直接执行脚本并无区别


## ansible中具体文件内容
**hosts文件内容**
``` bash
[java]
10.10.180.4
10.10.180.6
```
指定了远程主机分组和ip信息

**site.yml文件内容**
``` bash
---
- hosts: java
  remote_user: root

  roles:
    - tomcat_install
```
只是用来指定了tomcat_install这个角色，ansible会自动去寻找roles目录下的tomcat_install目录，然后执行该目录下的tasks目录下的main.yml

**tasks/main.yml文件内容**
``` bash
---
- name: copy jre
  copy: src=jre-7u80-linux-x64.tar.gz dest=/usr/local/src

- name: copy tomcat
  copy: src=apache-tomcat-8.5.4.tar.gz dest=/usr/local/src

- name: copy java env init scripts
  copy: src=java-env-7u80.sh dest=/usr/local/src

- name: copy tomcat installation scripts
  copy: src=install_tomcat.sh dest=/usr/local/src

- shell: sh ./install_tomcat.sh
  args:
    chdir: /usr/local/src/

# shell模块是用来在远程主机上执行脚本用的
# 与script模块区别是，后者执行的是ansible服务端脚本
# chdir参数是指，执行此脚本前，先去到指定的目录
```
此文件指定了需要执行的tasks
- 首先拷贝安全所需的文件到目标机器的/usr/local/src中
- 执行脚本


## 执行结果
``` bash
ansible-playbook -i hosts site.yml

PLAY [java] ********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.6]
ok: [10.10.180.4]

TASK [tomcat_install : copy jre] ***********************************************
changed: [10.10.180.4]
changed: [10.10.180.6]

TASK [tomcat_install : copy tomcat] ********************************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : copy java env init scripts] *****************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : copy tomcat installation scripts] ***********************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [tomcat_install : command] ************************************************
changed: [10.10.180.4]
changed: [10.10.180.6]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=6    changed=5    unreachable=0    failed=0
10.10.180.6                : ok=6    changed=5    unreachable=0    failed=0
```
