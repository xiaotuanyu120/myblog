---
title: python中unicode转str
date: 2016:07-15 23:20:00
categories: python
tags: [python,unicode,str]

---
## dict与json互相转换
python中的对象转换成json时，官方的提示如下

Python|JSON
-|-
dict|object
list, tuple|array
str, unicode|string
int, long, float|number
True|true
False|false
None|null

json对象转换成python对象时，官方的提示如下

JSON	|Python
-|-
object	|dict
array	|list
string	|unicode
number (int)	|int, long
number (real)	|float
true	|True
false	|False
null	|None

<!--more-->

让我们尝试转换dict类型
``` python
>>> d1 = {"a": "a1", "b": "b1", "c": "c1"}
>>> d2 = json.loads(json.dumps(d1))
>>> d1
{'a': 'a1', 'c': 'c1', 'b': 'b1'}
>>> d2
{u'a': u'a1', u'c': u'c1', u'b': u'b1'}
>>> type(d1.keys()[0])
<type 'str'>
>>> type(d2.keys()[0])
<type 'unicode'>
```
从上面的例子我们可以看出，当dict转换成json，再转换回来时，key的类型从str变换为unicode，因为，python是先把str转换成unicode，再转换成json，而json转换回python对象，也是默认转换成unicode。

有时我们希望当它转换回来依然保持str类型，那么我们就必须按照下面的办法了。

## unicode转换成str
句法举例：
``` python
>>> a = "this is a string"
>>> b = json.loads(json.dumps(a))
>>> b
u'this is a string'
>>> b.encode('utf-8')
'this is a string'
>>> type(b.encode('utf-8'))
<type 'str'>
```
通过这样显式的转码，就可以把unicode类型转换成str，但是如果对象是个dict，我们该怎么办呢？

这时候我们就需要创建一个函数来解决这个问题
```python
# 创建函数
>>> def _encode_it(data):
...     if isinstance(data, unicode):
...         return data.encode('utf-8')
...     elif isinstance(data, list):
...         return [_encode_it(x) for x in data]
...     elif isinstance(data, dict):
...         return dict([(_encode_it(key), _encode_it(value)) for key, value in data.iteritems()])
...     return data_handle
...

# 创建一个复杂的dict嵌套测试
>>> dict_test = {"webserver": {"service": ["nginx", "tomcat"], "port": [80, 8080]}}
>>> jd = json.loads(json.dumps(dict_test))
>>> jd
{u'webserver': {u'port': [80, 8080], u'service': [u'nginx', u'tomcat']}}

# 执行函数，查看效果
>>> _encode_it(jd)
{'webserver': {'port': [80, 8080], 'service': ['nginx', 'tomcat']}}
>>> type(_encode_it(jd).keys()[0])
<type 'str'>
>>> type(jd.keys()[0])
<type 'unicode'>
```