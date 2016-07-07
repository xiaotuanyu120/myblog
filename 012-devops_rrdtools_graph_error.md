---
title: rrdtool执行graph，得到图形字体是口口
date: 2016-07-07 21:32:00
categories: devops
tags: [devops,rrdtool]
---
## rrdtool执行graph命令，得到图形字体是口口的问题

#### 执行graph.py报错，提示选择字体失败
``` bash
python graph.py

(process:36975): Pango-WARNING **: failed to choose a font, expect ugly output. engine-type='PangoRenderFc', script='common'

(process:36975): Pango-WARNING **: failed to choose a font, expect ugly output. engine-type='PangoRenderFc', script='latin'
```
<!--more-->
#### 解决方案，安装下面的字体
``` bash
yum install dejavu-lgc-sans-fonts -y
```