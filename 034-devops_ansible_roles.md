---
title: ansible：什么是ansible roles
date: 2016-07-30 14:23:00
categories: devops
tags: [devops,ansible]

---
## 什么是roles？
当我们处理大型的项目部署或管理时，需要创建的playbook可能异常庞大，逻辑也十分复杂，这时候我们可能会希望将这些tasks、vars、files分离出来管理，虽然ansible提供了include的功能，可以在playbook中引入其他playbook的内容，但是roles更具层次性和结构性的帮助我们来管理playbook

<!--more-->

## roles的文件结构
1，第一级：role目录
2，第二级：角色目录
3，第三级：每个角色目录中分别包含files、handlers、meta、tasks、templates、vars、defaults（并不是全部都是必须的，也可为空目录）

**第三级目录详细介绍**
files目录：存放copy或script等模块调用的文件，并不需要指定相对或绝对路径，各角色会自动在这个目录下寻找文件；
tasks目录：默认文件是main.yml，用以定义tasks，此文件可以include同目录中的其他task文件；
templates目录：template模块会自动在此目录中寻找jinja2模版文件；
handlers目录：默认文件是main.yml，用以定义handlers，此文件可以include同目录中的其他handler文件；
vars目录：默认文件是main.yml，用以定义此角色用到的变量；
meta目录：默认文件是main.yml，用以定义此角色的特殊设定及其依赖关系；
defaults目录：默认文件是main.yml，为角色设定默认变量时使用此目录；
