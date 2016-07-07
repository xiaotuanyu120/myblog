---
title: RRDtool实例：监控网卡流量
date: 2016-07-07 21:20:00
categories: devops
tags: [devops,rrdtool]
---
## 创建create.py，生成rrd数据库文件

* 定义两个DS，分别为网卡流入和流出
* 定义三个CF，分别为AVERAGE,MAX,MIN
* 每个CF有4个RRA，以AVERAGE举例，其含义分别为：
  &emsp;&emsp;每隔5分钟(1\*300s)存储一次数据，存600笔，即2.08天
    每隔30分钟(6\*300s)存储一次数据，存700笔，即14.58天(2周)
  &emsp;&emsp;每隔2小时(24\*300s)存储一次数据，存775笔，即64.58天(2个月)
  &emsp;&emsp;每隔24小时(288\*300s)存储一次数据，存797笔，即797天(2年)
  <!--more-->
#### create.py内容
``` python
#!/root/env27/bin/python
# -*- coding: utf-8 -*-

import rrdtool
import time

cur_time = str(int(time.time()))
rrd = rrdtool.create(
    'Flow.rrd',
    '--step',
    '300',
    '--start',
    cur_time,
    'DS:eth0_in:COUNTER:600:0:U',
    'DS:eth0_out:COUNTER:600:0:U',
    'RRA:AVERAGE:0.5:1:600',
    'RRA:AVERAGE:0.5:6:700',
    'RRA:AVERAGE:0.5:24:775',
    'RRA:AVERAGE:0.5:288:797',
    'RRA:MAX:0.5:1:600',
    'RRA:MAX:0.5:6:700',
    'RRA:MAX:0.5:24:775',
    'RRA:MAX:0.5:444:797',
    'RRA:MIN:0.5:1:600',
    'RRA:MIN:0.5:6:700',
    'RRA:MIN:0.5:24:775',
    'RRA:MIN:0.5:444:797',
)
if rrd:
    print rrdtool.error()
```
PS: rrd若成功，返回NONE，bool值为假，不打印；若失败，返回值非空，bool值为真，则打印错误。


## 创建update.py，更新数据

使用psutil模块来获取网卡的进出流量，然后通过rrdtool的updatev方法写入到Flow.rrd数据库文件

#### 提前安装psutil模块
``` bash
pip install psutil
```

#### update.py内容
``` python
#!/usr/env27/bin/python
# -*- coding: utf-8 -*-

import rrdtool
import time
import psutil

total_input_traffic = psutil.net_io_counters()[1]
total_output_traffic = psutil.net_io_counters()[0]
starttime = int(time.time())
update = rrdtool.updatev(
    '/root/rrdtool/Flow.rrd',
    '%s:%s:%s' % (
        str(starttime),
        str(total_input_traffic),
        str(total_output_traffic)
    ),
)

if __name__ == "__main__":
    print starttime
    print total_input_traffic
    print total_output_traffic
```

#### 创建定时任务，crontab配置如下
``` bash
crontab -l
*/5 * * * * /root/env27/bin/python /root/rrdtool/update.py > /dev/null 2>&1
```


## 创建graph.py，生成图形

初次运行此脚本时，可能会遇到找不到字体的错误，见本站下一篇博客的解决方法

#### graph.py内容
``` python
#!/root/env27/bin/python
# -*- coding: utf-8 -*-

import rrdtool
import time

title = "Server network traffic flow (" + \
    time.strftime('%Y-%m-%d', time.localtime(time.time())) + ")"
rrdtool.graph(
    "Flow.png",
    "--start",
    "-1d",
    "--vertical-label=Bytes/s",
    "--x-grid",
    "MINUTE:12:HOUR:1:HOUR:1:0:%H",
    "--width", "650",
    "--height", "230",
    "--title", title,
    "DEF:inoctets=Flow.rrd:eth0_in:AVERAGE",
    "DEF:outoctets=Flow.rrd:eth0_out:AVERAGE",
    "CDEF:total=inoctets,outoctets,+",
    "LINE1:total#FF8833:Total traffic",
    "AREA:inoctets#00FF00:In traffic",
    "LINE1:outoctets#0000FF:Out traffic",
    "HRULE:6144#FF0000:Alarm value\\r",
    "CDEF:inbits=inoctets,8,*",
    "CDEF:outbits=outoctets,8,*",
    "COMMENT:\\r",
    "COMMENT:\\r",
    "GPRINT:inbits:AVERAGE:Avg In traffic\: %6.2lf %Sbps",
    "COMMENT:    ",
    "GPRINT:inbits:MAX:Max In traffic\: %6.2lf %Sbps",
    "COMMENT:    ",
    "GPRINT:inbits:MIN:Min In traffic\: %6.2lf %Sbps\\r",
    "COMMENT:    ",
    "GPRINT:outbits:AVERAGE:Avg Out traffic\: %6.2lf %Sbps",
    "COMMENT:    ",
    "GPRINT:outbits:MAX:Max Out traffic\: %6.2lf %Sbps",
    "COMMENT:    ",
    "GPRINT:outbits:MIN:Min Out traffic\: %6.2lf %Sbps\\r",
)
```