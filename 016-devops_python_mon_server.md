---
title: flask服务端与客户端传递json数据
date: 2016-07-13 09:22:00
categories: devops
tags: [devops,python,flask,json]
---
## 客户端程序
``` python
#!/root/env26/bin/python
# -*- coding: utf-8 -*-

"for monitor host's information"

import sys
import psutil
import json
import urllib2


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


def json_post(url, sysinfo_list):
    jdata = json.dumps(sysinfo_list)
    req = urllib2.Request(url, jdata)
    response = urllib2.urlopen(req)
    return response.read()


if __name__ == "__main__":
    args_len = len(sys.argv)
    if args_len > 2:
        args_num = args_len - 2
        args = [sys.argv[x] for x in range(2, args_len)]
        result = _cpu(args, args_num)
        url = "http://127.0.0.1:5000/"
        json_post(url, result)
    else:
        print "ERROR: too less arguments!"
```
<!--more-->


## 服务端程序
``` python
#!/root/env26/bin/python
# -*- coding: utf-8 -*-

from flask import Flask, request

app = Flask(__name__)

@app.route('/', methods=['POST'])
def get_data():
    data = request.get_data()
    # data = request.data
    print data
    return "request: " + str(data)


if __name__ == '__main__':
    app.run()
```


## 代码解析
&emsp;&emsp;客户端使用urllib2.urlopen()发送urllib2.Request(url, jdata)到服务端

&emsp;&emsp;处理请求的函数，return的值不可以是None，若是None，会报错"ValueError: View function did not return a response"

&emsp;&emsp;服务端接受json字符串有几种方法：
* request.data，但此种方法有限制，按照官方说法(Contains the incoming request data as string in case it came with a mimetype Flask does not handle.)，按照字面意思，只有特定的mimetype下，request.data才能拿到请求的值，一般情况下都是none;
> 一般需要对代码进行如下处理
> ``` python
> # 客户端
> def json_post(url, sysinfo_list, header):
>     jdata = json.dumps(sysinfo_list)
>     req = urllib2.Request(url, jdata, headers=header)
>     response = urllib2.urlopen(req)
>     return response.read()
>
> # if块中，自定义header并传入到urllib2.Request()的headers参数中
>         headers = {
>     'Content-type': 'binary/octet-stream',
> }
>         json_post(url, result, header=headers)
> ```
>
> ``` python
> # 服务端
> data = request.data
> return "request: " + str(data)
> ```


* get_json()，在"mimetype"是"application/json"或force参数是True时，才会拿到请求字符串的值
> ``` python
> # 客户端
> def json_post(url, sysinfo_list, header):
>     jdata = json.dumps(sysinfo_list)
>     req = urllib2.Request(url, jdata, headers=header)
>     response = urllib2.urlopen(req)
>     return response.read()
>
> # if块中，自定义header传入到urllib2.Request()的headers参数中
>         headers = {
>     'Content-type': 'application/json',
> }
>
>         json_post(url, result, header=headers)
> ```
>
> ``` python
> # 服务端
>     data = request.get_json()
>     return "request: " + str(data)
> ```


* get_data()，此方法可以完全的拿到请求，但需要注意的是，调用此请求的时候要检查请求内容的长度，不然有一定风险引起内存问题

* 详细可参见官网文档：http://flask.pocoo.org/docs/0.11/api/#flask.request