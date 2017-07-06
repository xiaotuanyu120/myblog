---
title: ansible python api 2.0研究
date: 2016-10-05 09:57:00
categories: devops
tags: [devops,ansible,ansible_api]
---
### ansible api 2.0 官方示例
[官方示例链接](http://docs.ansible.com/ansible/developing_api.html)
[精品教程讲解](http://swatij.me/python/ansible/programmatic-ansible-with-python)

ansible api 1.0使用非常方便和简单，但是2.0为了解耦和其他原因，将一些原生类提取出来直接作为api使用，ansible本身也是靠这些原生类驱动，可参考源码文件去了解以下api
源码文件所在路径为
``` bash
/path/to/python\'s basedir/lib/python2.7/site-packages/ansible/cli/adhoc.py
```

**下面为对官方示例的注释**

<!--more-->

``` python
#!/usr/bin/env python

#
# 导入ansible所需模块
# DataLoader, 负责数据解析
# VariableManager, 负责存储各类变量
# Inventory, 负责初始化hosts
# Play, 负责初始化playbook
# TaskQueueManager, 负责初始化执行对象, 其run()函数负责执行play
#
# '''
# TaskQueueManager(**, inventory=Inventory(**, host_list=hostsfile), options=options)
# .run(Play().load(dict, variable_manager=VariableManager(), loader=DataLoader())
# '''
#
import json
from collections import namedtuple
from ansible.parsing.dataloader import DataLoader
from ansible.vars import VariableManager
from ansible.inventory import Inventory
from ansible.playbook.play import Play
from ansible.executor.task_queue_manager import TaskQueueManager
from ansible.plugins.callback import CallbackBase

#
# 用于执行结果的调用，使用示例如下
# TaskQueueManager(**, stdout_callback=results_callback, **).run(***)
#
class ResultCallback(CallbackBase):
    """A sample callback plugin used for performing an action as results come in

    If you want to collect all results into a single object for processing at
    the end of the execution, look into utilizing the ``json`` callback plugin
    or writing your own custom callback plugin
    """
    def v2_runner_on_ok(self, result, **kwargs):
        """Print a json representation of the result

        This method could store the result in an instance attribute for retrieval later
        """
        host = result._host
        print json.dumps({host.name: result._result}, indent=4)

#
# 初始化options(tuple)
# 初始化passwords(dict)
# 实例化数据解析器DataLoader()为loader
# 实例化变量存储器VariableManager()为variable_manager
#
Options = namedtuple('Options', ['connection', 'module_path', 'forks', 'become', 'become_method', 'become_user', 'check'])
# initialize needed objects
variable_manager = VariableManager()
loader = DataLoader()
options = Options(connection='local', module_path='/path/to/mymodules', forks=100, become=None, become_method=None, become_user=None, check=False)
passwords = dict(vault_pass='secret')

#
# 结果回调类实例化
#
# Instantiate our ResultCallback for handling results as they come in
#
results_callback = ResultCallback()

#
# 指定loader, variable_manager和host_list, 来实例化Inventory()为inventory
# 把inventory传递给variable_manager管理
#
# create inventory and pass to var manager
#
inventory = Inventory(loader=loader, variable_manager=variable_manager, host_list='localhost')
variable_manager.set_inventory(inventory)

#
# 创建playbook
#
# create play with tasks
#
play_source =  dict(
        name = "Ansible Play",
        hosts = 'localhost',
        gather_facts = 'no',
        tasks = [
            dict(action=dict(module='shell', args='ls'), register='shell_out'),
            dict(action=dict(module='debug', args=dict(msg='{{shell_out.stdout}}')))
         ]
    )
play = Play().load(play_source, variable_manager=variable_manager, loader=loader)

#
# 通过TaskQueueManager().run()执行ansible，具体语法如下
# "TaskQueueManager(指定inventory，loader，variable_manager, options，passwords, stdout_callback).run(play)"
#
# actually run it
#
tqm = None
try:
    tqm = TaskQueueManager(
              inventory=inventory,
              variable_manager=variable_manager,
              loader=loader,
              options=options,
              passwords=passwords,
              stdout_callback=results_callback,  # Use our custom callback instead of the ``default`` callback plugin
          )
    result = tqm.run(play)
finally:
    if tqm is not None:
        tqm.cleanup()
```

---

## 基于官方示例的ansible api 2.0的自我封装实现
### 0. 项目地址
https://github.com/xiaotuanyu120/ansible_api_2
> 以下文档是基于20170706时的项目状态做的讲解，最新文档参照git项目中的readme文档

### 1. 获取项目文件
``` bash
git clone https://github.com/xiaotuanyu120/ansible_api_2.git
pip install -r ansible_api_2/requirements.txt
```
当获取了ansible_api_2的项目代码后，你可以将ansible_api_2目录中的conf,pb_data,playbook这三个目录mv到你自己的项目下合适的位置上。

关于上面三个目录，下面是详细介绍：
- conf, 里面存放的是playbook运行需要的options的一些配置信息，以json文件格式存放，配置可以参照`conf/default_ansible_option.json`内容
- playbook, 主要的项目代码，里面是ansible_api_2封装的自己的runner
- pb_data, 这个目录用于存放playbook运行所需要的yaml文件,roles目录和变量文件等，就和本地使用ansible所创建的目录一样


### 2. 关于传入给Runner的数据
#### 1) ansible_option
ansible_option是ansible需要的options参数，主要包含了一些类似于"become"、"become_method"、"private_key_file"等配置。  
可接收json数据或者json文件。  
> 详细配置可参照`conf/default_ansible_option.json`  
> 推荐将配置文件以json格式放置在conf下面,conf目录最好和playbook目录同级

#### 2) run_data
run_data是ansible需要的extra_vars参数，run_data必须是dict类型。

#### 3) host_list
host_list是ansible需要的inventory数据，可接收json数据，list或inventory文件  
json格式类似于:  
'[{"groupA": ["192.168.0.11", "w1 ansible_host=192.168.0.22 ansible_port=222"]}, {"groupB": ["192.168.0.11"]}, "192.168.0.1"]'
> 如果是inventory文件的话，推荐放置在pb_data目录下

#### 4) playbooks
playbooks是ansible需要的playbook文件，仅可接收yaml文件
> 推荐放置在pb_data目录下

### 3. DEMO
以下是一个调用示例
``` python
from playbook import Runner


ansible_option = "conf/your_ansible_option.json"
run_data = {'role': 'your_role', 'host': 'your_host'}
host_list='pb_data/hosts'
playbooks="pb_data/test.yml"

runner = Runner(ansible_option=ansible_option,
                run_data=run_data,
                host_list=host_list,
                playbooks=playbooks)

runner.run()
```
