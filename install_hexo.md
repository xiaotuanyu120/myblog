---
title: 制作hexo博客
---
More info：[hexo 官方文档](https://hexo.io/docs/index.html)

## 安装 Git
    # yum install -y git-core

## 安装 Node.js
使用脚本clone nvm的repository到~/.nvm下，
并且修改了~/.bash_profile文件的PATH，然后使用nvm命令安装Node.js
<!--more-->
    # wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
    # source ~/.bash_profile
    # nvm install 4

## 安装 Hexo
    # npm install -g hexo-cli

## 初始化
    # mkdir /data/web/hexo_blog
    # hexo init /data/web/hexo_blog
    
初始化之后的文件内容
.
├── _config.yml
├── node_modules
│   ├── hexo
│   ├── hexo-generator-archive
│   ├── hexo-generator-category
│   ├── hexo-generator-index
│   ├── hexo-generator-tag
│   ├── hexo-renderer-ejs
│   ├── hexo-renderer-marked
│   ├── hexo-renderer-stylus
│   └── hexo-server
├── package.json
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source
│   └── _posts
└── themes
└── landscape

## 生成静态文件
    # hexo generate
    ## 生成md文件的静态文件

## 启动服务
    # nohup hexo server -s &
    
    "-s" 只提供静态文件给予访问
    "nohup" 会将命令执行的stdout重定向到nohup.out文件中

## nginx虚拟主机配置
    server {
        listen 80;
        server_name linux.xiao5tech.com xiao5tech.com;
        root /data/web/hexo_blog/public;
        index index.html;
    }


## 安装主题Next：
[next主题安装](http://theme-next.iissnan.com/getting-started.html)

## 如何在主页显示文章摘要，而不是显示全文
这个需要在md文件中添加<!--more-->，然后重新"hexo generate"静态文件
