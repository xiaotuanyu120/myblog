---
title: Hexo的分类和标签设置
date: 2016-07-06 08:33:00
categories: hexo
tags: [hexo]
---
## 新建页面命令
``` bash
$ hexo n "test"
INFO  Created: /data/web/hexo_blog/source/_posts/test.md
## 文件内容是从scaffolds/post.md模版中来的
$ cat /data/web/hexo_blog/source/_posts/test.md
---
title: test
date: 2016-07-06 08:35:32
categories:
tags:
---

$ cat /data/web/hexo_blog/scaffolds/post.md
---
title: {{ title }}
date: {{ date }}
categories:
tags:
---
```
<!--more-->

## 设置分类列表
&emsp;&emsp;在我们编辑文章分类时，编写的<code>categories:</code>内容会包含到我们的访问路径中去。
例如：
``` bash
categories: 博客
```
在生成页面后，分类列表会出现<code>博客</code>这个选项，路径是：
``` bash
categories/博客
```

若想区分路径名称和分类名称，可在<code>\_config.yml</code>中配置：
``` bash
# Category & Tag
default_category: uncategorized
category_map:
        博客: blog
tag_map:
```
<code>category_map:</code>配置的是分类map，语法为"分类名称: 访问路径"

## 设置标签
&emsp;&emsp在编辑文章时，<code>tags: </code>后面配置的是标签，如果有多个标签，可以用以下办法来设置：
```
tags: [tag1,tag2,...tagn]
```

```
tags:
- tag1
- tag2
- ...
- tagn
```
