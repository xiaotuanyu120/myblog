## 一,安装
``` bash
# 安装ansible
yum install epel-release
yum install -y ansible libselinux-python

# 创建virtualenv环境
mkdir /root/ansible-tmp
cd /root/ansible-tmp
virtualenv venv -p /usr/local/py27/bin/python2.7
. ./venv/bin/activate

# 安装ansible需要的python模块
pip install paramiko PyYAML Jinja2 httplib2 six
```

<!--more-->

## 二，编辑hosts文件，sshkey拷贝及ping测试
``` bash
# 编辑默认的hosts文件
cat /etc/ansible/hosts
[web]
10.10.180.4
[web02]
10.10.180.6
# 如果不使用这个文件，需要用-i参数指定自定义的hosts文件


# ssh访问，创建key信任
ssh-keygen -b 1024 -t rsa
scp /root/.ssh/id_rsa.pub root@10.10.180.4:/root/.ssh/authorized_keys
scp /root/.ssh/id_rsa.pub root@10.10.180.6:/root/.ssh/authorized_keys

# 使用ping测试
ansible all -m ping
10.10.180.6 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
10.10.180.4 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## 三，playbook编写
### 1.shell命令模块


``` bash
cat createfile.yml
---
- hosts: all
  remote_user: root

  tasks:
  - name: shell command test
    shell: echo "command test"


ansible-playbook shell.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.6]
ok: [10.10.180.4]

TASK [shell command test] ******************************************************
changed: [10.10.180.6]
changed: [10.10.180.4]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=1    unreachable=0    failed=0
10.10.180.6                : ok=2    changed=1    unreachable=0    failed=0
```

### 2.file模块
参考链接：[file模块](http://docs.ansible.com/ansible/file_module.html)

创建文件
``` bash
cat createfile.yml
---
- hosts: all
  remote_user: root

  tasks:
  - name: shell command test
    file: path=/tmp/createfile-{{ item }} state=touch mode="u=rw,g=r,o=r"
    with_items:
      - 01
      - 02


ansible-playbook createfile.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.6]
ok: [10.10.180.4]

TASK [shell command test] ******************************************************
changed: [10.10.180.4] => (item=1)
changed: [10.10.180.6] => (item=1)
changed: [10.10.180.4] => (item=2)
changed: [10.10.180.6] => (item=2)

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=1    unreachable=0    failed=0
10.10.180.6                : ok=2    changed=1    unreachable=0    failed=0
```

检查文件是否存在
``` bash
cat checkfilexist.yml
---
- hosts: all
  remote_user: root

  tasks:
  - name: shell command test
    file: path=/tmp/createfile-{{ item }} state=file
    with_items:
      - 01
      - 02

ansible-playbook checkfilexist.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.4]
ok: [10.10.180.6]

TASK [shell command test] ******************************************************
ok: [10.10.180.6] => (item=1)
ok: [10.10.180.4] => (item=1)
ok: [10.10.180.4] => (item=2)
ok: [10.10.180.6] => (item=2)

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=0    unreachable=0    failed=0
10.10.180.6                : ok=2    changed=0    unreachable=0    failed=0
```
可以通过在playbook中加入mode，来改变现存文件的权限

删除指定文件
``` bash
cat checkfilexist.yml
---
- hosts: all
  remote_user: root

  tasks:
  - name: shell command test
    file: path=/tmp/createfile-{{ item }} state=file
    with_items:
      - 01
      - 02

ansible-playbook deletefile.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.4]
ok: [10.10.180.6]

TASK [shell command test] ******************************************************
changed: [10.10.180.6] => (item=1)
changed: [10.10.180.4] => (item=1)

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=1    unreachable=0    failed=0
10.10.180.6                : ok=2    changed=1    unreachable=0    failed=0

# 检查文件是否删除
ansible-playbook checkfilexist.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.4]
ok: [10.10.180.6]

TASK [shell command test] ******************************************************
failed: [10.10.180.6] (item=1) => {"failed": true, "item": 1, "msg": "file (/tmp/createfile-1) is absent, cannot continue", "path": "/tmp/createfile-1", "state": "absent"}
failed: [10.10.180.4] (item=1) => {"failed": true, "item": 1, "msg": "file (/tmp/createfile-1) is absent, cannot continue", "path": "/tmp/createfile-1", "state": "absent"}
ok: [10.10.180.6] => (item=2)
ok: [10.10.180.4] => (item=2)

NO MORE HOSTS LEFT *************************************************************
        to retry, use: --limit @checkfilexist.retry

PLAY RECAP *********************************************************************
10.10.180.4                : ok=1    changed=0    unreachable=0    failed=1
10.10.180.6                : ok=1    changed=0    unreachable=0    failed=1
```

### 3.copy模块（内含yum模块和selinux模块内容）

扩展链接：
> [copy模块](http://docs.ansible.com/ansible/copy_module.html)
[selinux模块](http://docs.ansible.com/ansible/selinux_module.html)
[yum模块](http://docs.ansible.com/ansible/yum_module.html)

拷贝文件和关闭selinux
``` bash
cat copy.yml
---
 - hosts: all
   remote_user: root

   tasks:
   - name: yum install libselinux-python
     yum: name=libselinux-python state=latest
   - name: disable selinux
     selinux: state=disabled
   - name: copy file test
     copy: src=/usr/local/src/get-pip.py dest=/usr/local/src/ group=root owner=root mode=0644
# 之所以提前安装libselinux-python是为了避免copy模块出错

ansible-playbook copy.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.6]
ok: [10.10.180.4]

TASK [yum install libselinux-python] *******************************************
changed: [10.10.180.4]
changed: [10.10.180.6]

TASK [disable selinux] *********************************************************
changed: [10.10.180.6]
changed: [10.10.180.4]

TASK [copy file test] **********************************************************
changed: [10.10.180.4]
changed: [10.10.180.6]

PLAY RECAP *********************************************************************
10.10.180.4                : ok=4    changed=3    unreachable=0    failed=0
10.10.180.6                : ok=4    changed=3    unreachable=0    failed=0
```

### 4.setup模块

扩展链接：[setup模块](http://docs.ansible.com/ansible/setup_module.html)
```bash
ansible all -m setup -a "filter=ansible_interfaces"
10.10.180.6 | SUCCESS => {
    "ansible_facts": {
        "ansible_interfaces": [
            "lo",
            "eth1"
        ]
    },
    "changed": false
}
10.10.180.4 | SUCCESS => {
    "ansible_facts": {
        "ansible_interfaces": [
            "lo",
            "eth1"
        ]
    },
    "changed": false
}
```
setup模块默认打印出目标机器的完整信息，可通过filter筛选出自己想要得到的

## 四，参数
### 1.check模式
``` bash
# 先在目标机器创建文件
ansible-playbook createfile.yml

# 用check参数来执行删除该文件操作(输出和正常操作一样)
ansible-playbook deletefile.yml --check

# 检查文件是否存在
ansible-playbook checkfilexist.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.4]
ok: [10.10.180.6]

TASK [shell command test] ******************************************************
changed: [10.10.180.6] => (item=1)
changed: [10.10.180.4] => (item=1)
changed: [10.10.180.6] => (item=2)
changed: [10.10.180.4] => (item=2)

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=1    unreachable=0    failed=0
10.10.180.6                : ok=2    changed=1    unreachable=0    failed=0
# 文件并没有被删除，因为上面的删除操作只是check，并没有实际进行删除操作
```

### 2.关闭gather_facts
ansible默认启用gather_facts，这里可以选择关闭，加快速度

``` bash
# 未关闭gather_facts，用时4.66s
time ansible-playbook checkfilexist.yml

PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [10.10.180.6]
ok: [10.10.180.4]

TASK [shell command test] ******************************************************
ok: [10.10.180.6] => (item=1)
ok: [10.10.180.4] => (item=1)
ok: [10.10.180.4] => (item=2)
ok: [10.10.180.6] => (item=2)

PLAY RECAP *********************************************************************
10.10.180.4                : ok=2    changed=0    unreachable=0    failed=0
10.10.180.6                : ok=2    changed=0    unreachable=0    failed=0


real    0m4.655s
user    0m2.057s
sys     0m1.764s

# 给yml文件加入"gather_facts: no"，用时2.47s
time ansible-playbook checkfilexist.yml

PLAY [all] *********************************************************************

TASK [shell command test] ******************************************************
ok: [10.10.180.4] => (item=1)
ok: [10.10.180.6] => (item=1)
ok: [10.10.180.4] => (item=2)
ok: [10.10.180.6] => (item=2)

PLAY RECAP *********************************************************************
10.10.180.4                : ok=1    changed=0    unreachable=0    failed=0
10.10.180.6                : ok=1    changed=0    unreachable=0    failed=0


real    0m2.469s
user    0m1.454s
sys     0m0.977s
```