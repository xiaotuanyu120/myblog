---
title: python自省：让与字符串同名的函数执行
date: 2016-07-20 14:15:00
categories: python
tags: [python,自省]

---
很多时候我们会遇到这样的情况，需要让与字符串同名的函数执行。
例如，我们有一系列函数
`test01 test02 ...`
我们需要在判断用户输入的字符串后，执行相应的test函数。

**创建一个模块**
``` bash
vim test.py
	*****************************
	def test01():
		print "this is test01"


	def test02():
		print "this is test02"


	def test03():
		print "this is test03"
	*****************************
```

<!--more-->

**AttributeError**
若我们直接给变量赋值字符串，然后尝试执行该变量试图去调用该函数，我们会遇到下面的错误
``` python
>>> import test
>>> test.test01()
this is test01
>>> a = 'test01'
>>> test.a()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'module' object has no attribute 'a'
```

**正确方式**
应该是用python的自省方式来解决这个问题
``` python
# 利用自省函数getattr创建一个自检执行函数
def run_func(obj, func):
    try:
        func_run = getattr(obj, func)
    except AttributeError as e:
        print e
        return e
    func_run()

# 尝试用字符串变量执行函数
>>> import test
>>> a = "test02"

>>> run_func(test, a)
this is test02
```

**扩展：批量找出可执行的函数**
``` python
# 用dir()函数找出所有属性
>>> dir(test)
['__builtins__', '__doc__', '__file__', '__name__', '__package__', 'test01', 'test02', 'test03']

# 用callable()函数来判断该属性是否可调用执行
>>> callable(test.test01)
True

# 用for循环来检测所有的属性，筛选出可执行的函数
>>> for i in dir(test):
...     if callable(getattr(test, i)):
...         print i
...
test01
test02
test03
```