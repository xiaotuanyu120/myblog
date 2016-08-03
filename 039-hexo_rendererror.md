---
title: hexo博客：render error
date: 2016-08-03 16:25:00
categories: hexo
tags: hexo

---
## 问题
**执行命令：**
``` bash
hexo g
```
**错误信息**
``` bash
Template render error: (unknown path) [Line 27, Column 80]
  unexpected token: }}
```

## 解决方案
> Escape Contents
> Hexo uses Nunjucks to render posts (Swig was used in older version, which share a similar syntax). Content wrapped with {{ }} or {% %} will get parsed and may cause problems. You can wrap sensitive content with the raw tag plugin.
> 
> {% raw %}
> Hello {{ sensitive }}
> {% endraw %}

按照官方解决方案，{{  }}和{%  %}这种符号需要用{% raw %} {% endraw %}转义