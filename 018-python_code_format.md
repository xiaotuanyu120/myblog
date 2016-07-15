---
title: python中的编码格式相关
date: 2016-07-14 20:31:00
categories: python
tags: [python,unicode]
---
## 编码的相关内容
### 扩展阅读：
- https://www.zhihu.com/question/23374078
- https://docs.python.org/2/howto/unicode.html

### 个人总结：
- ascii是美国人创建的标准，8位，使用的是0-127，每个字节一个英文字符
- GB2312,GBK,GB18030是中国大陆创建的汉字标准，每两个字节一个中文字符
- BIG5是台湾创建的汉字标准，每两个字节一个中文字符
- 其他国家和地区也陆续有一些字符集标准
- 国际ISO组织为了解决字符集的混乱情况，推出了unicode标准

以上全部都是字符集，而UTF标准分为UTF-8和UTF-16，属于信道编码，为了更好的传输和存储。总结来讲，就是UNICODE是标准，UTF-8是其一种编码方式。
<!--more-->
---

## python的默认编码
- python2中默认编码是ASCII(在当时是主流)
- python3中默认编码是UNICODE(因为ASCII已经满足不了需求了)

### 如何修改默认编码
``` python
>>> import sys
>>> sys.getdefaultencoding()
'ascii'
>>> reload(sys)
<module 'sys' (built-in)>
>>> sys.setdefaultencoding('utf-8')
>>> sys.getdefaultencoding()
'utf-8'
```
以上代码修改了python默认的编码方式，但是因为ascii和utf-8的编码范围不一致，可能会导致系统在处理特殊字符时发生错误，详情见下面链接
扩展链接:[立即停止使用 setdefaultencoding('utf-8')， 以及为什么](http://blog.ernest.me/post/python-setdefaultencoding-unicode-bytes)

### 如何声明python源码的encodings
有时候我们需要在编写的python程序中使用ASCII编码外的字符，但python2默认的编码是ASCII，这样就会引起报错，所以python允许声明python源码文件使用的编码格式，但需要注意的是，这只是声明了源码所用的编码格式，并无他意。

具体用法如下：
``` python
# -*- coding: utf-8 -*-
```
- 需要处在python代码文件的前两行
- 其他语法见pep8规范：[pep-0263](https://www.python.org/dev/peps/pep-0263/)