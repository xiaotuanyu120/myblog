---
title: unicode vs str (in python2)
date: 2016-07-15 14:53:00
categories: python
tags: [python,str,unicode]

---
## python3中的unicode
在python3中，unicode相当于python2中的str，用来取代python2中的str来操作text，而str则改名为bytes。
之所以会发生这样的改变，是因为，在python2的设计中，str其实是a sequence of bytes。我们知道，在计算机中，用来储存的只有bytes，这种用来储存的bytes是否可以被当作一个string，在编程语言届有两个阵营，像java和c是不承认的它们是一个东西的，而python2认为，bytes就是text，它们的操作应该是一致的。并且为了python2的语法简洁，python2会在特殊情况下自动去转换其类型，例如bytes和text的拼接。这样则产生了一个问题，python需要一个默认的encoding，于是它选择了当时的ASCII。而ASCII由于编码范围太小，基本无法满足现在的需求。
于是python3中，用unicode取代了ASCII成为了新的默认encoding，基本可以满足所有的转码需求，并且，默认用它取代了str。
需要注意的是unicode is not encoded。

## python3之前的string类型
在python3之前，有两种string类型，分为"plain string"和"unicode strings"。而这两种string类型的父类是basestring，结构如下
```
          object
             |
             |
         basestring
            / \
           /   \
         str  unicode
```
之所以在python会出现这种情况，根源是因为python2设计的时候，unicode还没有出现。在python3中已经统一了这个问题。
```python
>>> string1 = "I am a plain string"
>>> string2 = u"I am a unicode string"
>>> isinstance(string1, str)
True
>>> isinstance(string2, str)
False
>>> isinstance(string1, unicode)
False
>>> isinstance(string2, unicode)
True
>>> isinstance(string1, basestring)
True
>>> isinstance(string2, basestring)
True
```

## 如何正确的处理python2中的编码问题
- 上一节博客中提到过，不推荐通过修改默认编码来解决，会引起潜在的编码处理上的错误
- 推荐统一转换成unicode进行处理，显式的decode和encode字符串