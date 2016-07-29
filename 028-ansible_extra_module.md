---
title: ansible扩展：模块
date: 2016-07-29 21:41:00
categories: devops
tags: [devops,python,ansible]

---
## 模块-扩展
### 1.service模块
参考链接：[service模块文档](http://docs.ansible.com/ansible/service_module.html)

 在httpd未启动时，启动它
<code>service: name=httpd state=started</code>

 在httpd运行时，关闭它
<code>service: name=httpd state=stopped</code>

 重启httpd服务
<code>service: name=httpd state=restarted</code>

 重新加载httpd服务
<code>service: name=httpd state=reloaded</code>

 配置httpd服务开机启动
<code>service: name=httpd enabled=yes</code>

 基于/usr/bin/foo来启动foo服务
<code>service: name=foo pattern=/usr/bin/foo state=started</code>

 为eth0网卡，重启network服务
<code>service: name=network state=restarted args=eth0</code>

<!--more-->

**实例：安装httpd并启动它**
``` bash
cat httpd_install.yml
---
- hosts: web01
  remote_user: root

  tasks:
  - name: install httpd
    yum: name=httpd state=installed

  - name: start httpd
    service: name=httpd state=started

ansible-playbook httpd_install.yml

PLAY [web01] *******************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.4]

TASK [install httpd] ***********************************************************
changed: [10.10.180.4]

TASK [start httpd] *************************************************************
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=3    changed=2    unreachable=0    failed=0
```

### 2.cron模块
参考链接：[cron模块](http://docs.ansible.com/ansible/cron_module.html)

创建计划任务"0 5,2 * * ls -alh > /dev/null"
<code>cron: name="check dirs" minute="0" hour="5,2" job="ls -alh > /dev/null"</code>

若任务存在，在crontab中删除它
<code>cron: name="an old job" state=absent</code>

创建计划任务，重启时执行某个任务"@reboot /some/job.sh"
<code>cron: name="a job for reboot" special_time=reboot job="/some/job.sh"</code>

**实例：**
``` bash
cat cron.yml
---
- hosts: web01
  remote_user: root
  gather_facts: no

  tasks:
  - name: add crontab rule test
    cron: name="test" job="echo 'cron test'"

ansible-playbook cron.yml

PLAY [web01] *******************************************************************

TASK [add crontab rule test] ***************************************************
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=1    changed=1    unreachable=0    failed=0

# 实际目标机器中的规则内容是
crontab -l
#Ansible: test
* * * * * echo 'cron test'
```

### 3.yum模块
参考链接：[yum模块](http://docs.ansible.com/ansible/yum_module.html)

安装软件
<code>yum: name=httpd state=latest</code>

卸载软件
<code>yum: name=httpd state=absent</code>

指定repo安装软件
<code>yum: name=httpd enablerepo=testing state=present</code>

更新所有软件
<code>yum: name=* state=latest</code>

从网络rpm文件url安装(也可以是本地rpm文件路径)
<code>yum: name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present</code>

group安装软件
<code>yum: name="@Development tools" state=present</code>

**实例：**
``` bash
cat yum.yml
---
- hosts: web01
  gather_facts: no
  remote_user: root

  tasks:
  - name: yum install test
    yum: name=vim state=latest

  - name: yum groupinstall test
    yum: name="@Development tools" state=present

 ansible-playbook yum.yml

PLAY [web01] *******************************************************************

TASK [yum install test] ********************************************************
ok: [10.10.180.4]

TASK [yum groupinstall test] ***************************************************
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=1    unreachable=0    failed=0
```

### 4.user&group模块
参考链接：http://docs.ansible.com/ansible/user_module.html
参考链接：http://docs.ansible.com/ansible/group_module.html

增加用户，指定shell，指定主组和附属组，指定uid
<code>- user: name=james shell=/bin/bash group=james uid=1041 groups=admins,developerss</code>

删除用户，并删除家目录
<code>- user: name=johnd state=absent remove=yes</code>

增加group
<code>- group: name=somegroup state=present</code>

删除group
<code>- group: name=somegroup state=absent</code>

``` bash
# 增加组和用户
cat user_group.yml
---
- hosts: web01
  gather_facts: no
  remote_user: root

  tasks:
   - name: add group admins
     group: name=admins state=present

   - name: add group stuffs
     group: name=stuffs state=present

   - name: add user zs
     user: name=zs shell=/bin/bash groups=admins,stuffs

   - name: add user ls
     user: name=ls groups=admins,stuffs append=yes

ansible-playbook user_group.yml

PLAY [web01] *******************************************************************

TASK [add group admins] ********************************************************
changed: [10.10.180.4]

TASK [add group stuffs] ********************************************************
changed: [10.10.180.4]

TASK [add user zs] *************************************************************
changed: [10.10.180.4]

TASK [add user ls] *************************************************************
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=4    changed=4    unreachable=0    failed=0

# 删除用户和组
cat del_user_group.yml
---
- hosts: web01
  gather_facts: no
  remote_user: root

  tasks:
   - name: del user zs
     user: name=zs state=absent remove=yes

   - name: del user ls
     user: name=ls state=absent remove=yes

   - name: del group admins
     group: name=admins state=absent

   - name: del group stuffs
     group: name=stuffs state=absent

ansible-playbook del_user_group.yml

PLAY [web01] *******************************************************************

TASK [del user zs] *************************************************************
changed: [10.10.180.4]

TASK [del user ls] *************************************************************
changed: [10.10.180.4]

TASK [del group admins] ********************************************************
changed: [10.10.180.4]

TASK [del group stuffs] ********************************************************
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=4    changed=4    unreachable=0    failed=0
```