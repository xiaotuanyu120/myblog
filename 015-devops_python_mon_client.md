---
title: 监控cpu状态并输出json数据的python实例
date: 2016-07-11 16:40:00
categories: devops
tags: [devops,python,json,cpu]
---
## 监控cpu状态脚本

### 脚本目标：
* 监控cpu状态
* 输出json结果
<!--more-->
### 脚本内容
``` python
#!/root/env26/bin/python
# -*- coding: utf-8 -*-

"for monitor host's information"

import sys
import psutil
import json


def _cpu(args, args_num):
    result = []
    for i in range(args_num):
        try:
            getattr(psutil, args[i])
        except AttributeError as e:
            print e
            continue
        result.append(getattr(psutil, args[i])())
    return result


def json_data(sysinfo_list):
    return json.dumps(sysinfo_list)


if __name__ == "__main__":
    args_len = len(sys.argv)
    if args_len > 2:
        args_num = args_len - 2
        args = [sys.argv[x] for x in range(2, args_len)]
        result = _cpu(args, args_num)
        print type(json_data(result)), json_data(result)
    else:
        print "ERROR: too less arguments!"
```

### 脚本解析
* 脚本环境用的virtualenv函数创造的env26环境
* 脚本经过pep8语法检查
* 函数名称以"\_"代表是个私有函数
* 使用getattr()自省函数来检查输入的方法是否存在，并把结果以list方式返回，这样的话就支持任意的方法输入
* json.dumps()方法支持将数据类型转换成json类型字符串
* 使用方法上，设想第一个参数是指定方法对象，此例中只有一个对象，就是cpu，以后方便扩展
* 操作逻辑上，需要控制传参个数，最少三个参数(0,1,2)