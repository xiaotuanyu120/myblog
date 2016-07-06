---
title: Hexo添加多说评论功能
date: 2016-07-06 10:42:00
categories: hexo
tags: [hexo,duoshuo]
---
参考链接：[next主题-多说功能](http://theme-next.iissnan.com/third-party-services.html)

# next主题启用多说功能
1. NexT 支持 多说 和 DISQUS 评论系统。 当同时设置了 多说 和 DISQUS 时，优先选择多说。 NexT 内置了一套 多说 的样式。
   <!--more-->
2. 访问[多说](http://duoshuo.com/)登录后在首页选择 “我要安装”。
3. 创建站点，填写表单。**多说域名** 这一栏填写的即是你的<code>duoshuo_shortname</code>
4. 创建站点完成后在<code>站点配置文件(\_config.yml)</code>中新增<code>duoshuo_shortname</code>字段，值设置成上一步中的值。
```
...
# duoshuo
duoshuo_shortname: xiao5tech
...
```

重新生成静态文件
```
$ hexo g
```