---
title: RRDtool初探-语法
date: 2016-07-07 10:51:00
categories: devops
tags: [devops,rrdtool]
---
## rrdcreate语法
```
create filename [--start|-b start time] [--step|-s step] [DS:ds-name:DST:heartbeat:min:max] [RRA:CF:xff:steps:rows]
```
* filename - 创建的rrdtool数据库文件，默认后缀是.rrd；
* --start - 指定rrdtool第一条记录的起始时间，必须是timestamp格式；
* --step - 指定rrdtool接受数据的基础间隔时间，默认300s；
* DS:ds-name - 用于定义data source，DS是固定标识，ds-name是该数据源的变量名称；
* DST - 用于定义数据源的类型，包括：
&emsp;&emsp;COUNTER，递增类型，用差值除以时间间隔，储存该值
&emsp;&emsp;DERIVE，可递增可递减类型，用差值除以时间间隔，储存该值
&emsp;&emsp;ABSOLUTE，假定前一个时间点的值是0，用差值除以时间间隔，储存该值
&emsp;&emsp;GAUGE，储存实测数值
&emsp;&emsp;[更详细介绍](http://www.jianshu.com/p/b925b1584ab2)
* heartbeat - 心跳值，定义了两个数据库updates之间的最大间隔时间，超过的话储存unknown，未超过时取前后等时间段内的平均值，例如600，未超过时取前300和后300的平均值；
* min:max - 分别是最小值，最大值，超出此范围储存unknown；
* RRA - 定义了一个存档，相当于一个表，保存不同间隔的统计结果数据；
* CF - (consolidation function)统计合并数据，包括AVERAGE,MAX,MIN,LAST(最新值)；
* xff - 定义一个0-1之间的数值，被赋值为UNKNOW的PDP(基本数据点)除以CDP(合并数据点)大于等于此值，则储存为UNKNOW；
* steps - 定义多少个PDP用来构建CDP；
* rows - 定义在一个RRA存档中保留多少次生成的数据；
对于steps和rows可用后缀代替纯数字，s (seconds), m (minutes), h (hours), d (days), w (weeks), M (months), 和 y (years)。

## rrdupdate语法
```
update|updatev filename [--template|-t ds-name[:ds-name]...] N|timestamp:value[:value...] timestamp:value[:value...] ...]
```
* filename - rrd文件名称；
* -t ds-name[:ds-name] - 指定要更新的DS名称；
* N|timestamp - 表示数据采集的时间戳，N代表的是当前时间戳；
* value[:value] - 表示更新的数据值，多个DS则多个值。

## rrdgraph语法
```
graph|graphv filename [-s|--start seconds] [-e|--end seconds] [-x|--x-grid x-axis grid and label] [-y|--y-grid y-axis grid and label] [--alt-y-mrtg] [--alt-autoscale] [--alt-autoscale-max] [--units-exponent] value [-v|--vertical-label text] [-w|--width pixels] [-h|--height pixels] [-i|--interlaced] [-f|--imginfo formatstring] [-a|--imgformat GIF|PNG|GD] [-B|--backgroud value] [-O|--overlay value] [-U|--unit value] [-z|--lazy] [-o|--logarithmic] [-u|--upper-limit value] [-l|--lower-limit value] [-g|--no-legend] [-r|--rigid] [--step value] [-b|--base value] [-c|--color COLORTAG#rrggbb] [-t|--title title] [DEF:vname=rrd:ds-name:CF] [CDEF:vname=rpn-expression] [PRINT:vname:CF:format] [GPRINT:vname:CF:format] [COMMENT:text] [HRULE:value#rrggbb[:legend]] [VRULE:time#rrggbb[:legend]] [LINE{1|2|3}:vname[#rrggbb[:legend]]] [AREA:vname[#rrggbb[:legend]]] [STACK:vname[#rrggbb[:legend]]]
```
* filename - 指定输出图像的文件名，默认是PNG格式；
* --start - 指定起始时间；
* --end - 指定结束时间；
* --x-grid - 控制X轴网格线刻度、标签的位置；
* --y-grid - 控制Y轴网格线可读、标签的位置；
* --vertical-label - 指定Y轴的说明文字；
* --width pixels - 指定图表宽度(像素)；
* --height pixels - 指定图表高度(像素)；
* --imgformat - 指定图像格式(GIF|PNG|GD)；
* --background - 指定图像背景颜色，支持#rrggbb表示法；
* --upper-limmit - 指定Y轴数据值上限；
* --lower-limmit - 指定Y轴数据值下限；
* --no-legend - 取消图表下方的图例；
* --rigid - 严格按照upper-limit与lower-limmit来绘制；
* --title - 图表顶部的标题；
* DEF:vname=rrd:ds-name:CF - 指定绘图用到的数据源；
* CDEF:vname=rpn-expression - 合并多个值； 
* GPRINT:vname:CF:format - 图表的下方输出最大值、最小值、平均值等；
* COMMENT:text - 指定图表中输出的一些字符串；
* HRULE:value#rrggbb - 用于在图表上面绘制水平线；
* VRULE:time#rrggbb - 用于在图表上面绘制垂直线；
* LINE{1|2|3}:vname - 使用线条来绘制数据图表，{1|2|3}表示线条的粗细；
* AREA:vname - 使用面积图来绘制数据图表。

## rrdfetch语法
```
fetch filename CF [--resolution|-r resolution] [--start|-s start] [--end|-e end]
```
* filename - 指定要查询的rrd文件名；
* CF - 包括ASVERAGE,MAX,MIN,LAST，要求必须是建库时RRA中定义的类型，否则会报错；
* --start --end - 指定查询记录的开始与结束时间，默认可省略。