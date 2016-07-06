---
title: Hexo创建tags和categories页面
date: 2016-07-06 10:01:00
categories: hexo
tags: [hexo,tags,categories]
---
## 问题现象
&emsp;&emsp;点击博客主页的tags(标签)链接，返回403错误
<!--more-->
## 解决办法-创建tags页面
1. 新建页面tags
``` bash
$ hexo new page "tags"
INFO  Created: /data/web/hexo_blog/source/tags/index.md
```
该语句会在source目录下创建相应目录和index.md文件

1. 编辑tags的index.md文件(/data/web/hexo_blog/source/tags/index.md)
```
---
title: tags
date: 2016-07-06 09:56:51
type: "tags"
comments: false
---
```
主要是增加type和comments
也有人说title必须是Tagcloud，不过我没有修改，保持默认tags的title也生效了

1. 编辑**主题配置文件**(/data/web/hexo_blog/themes/next/\_config.yml)，添加tags路径到menu中，如下:
```
menu:
  home: /
  categories: /categories
  #about: /about
  archives: /archives
  tags: /tags
```

1. 生成静态文件
```
hexo generate
```

ps:同理也可以创建categories页面
```
$ hexo new page "categories"
INFO  Created: /data/web/hexo_blog/source/categories/index.md

$ vim /data/web/hexo_blog/source/categories/index.md
    **************************
    ---
    title: categories
    date: 2016-07-06 10:21:28
    type: "categories"
    comments: false
    ---
    **************************

$ vim /data/web/hexo_blog/themes/next/_config.yml
    **************************
    menu:
      home: /
      categories: /categories
      #about: /about
      archives: /archives
      tags: /tags
    **************************
    
$ hexo g
```