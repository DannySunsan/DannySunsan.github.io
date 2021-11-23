---
title: hexo常见问题
date: 2021-11-22 16:46:16
tags:
categories: hexo
---

# 1. 安装 #

1.安装Git
2.安装Node.js
3.安装Hexo
    npm install -g hexo-cli
    hexo init myblog

# 2. 配置 #

    url: https://dannysunsan.github.io/
    permalink: :category/:title/
    pretty_urls:
    trailing_index: false # Set to false to remove trailing 'index.html' from permalinks
    trailing_html: false # Set to false to remove trailing '.html' from permalinks

    default_category: bin
    post_asset_folder: true
    theme: miho
    deploy:
    type: git
        repository: https://github.com/DannySunsan/DannySunsan.github.io.git
        branch: gh-pages

# 3. 部署 #

    hexo clean //清除生成，每次重新部署前需要执行
    hexo g 	   //生成
    hexo d     //执行

# 4. 写作 #

## tags & categories ##

    hexo new page tags
    修改tags/index.md 里面的内容为
    ---
    title: 文章标签
    date: 2021-11-22 14:56:48
    type: "tags"
    ---

    hexo new page categories
    修改categories/index.md 里面的内容为
    ---
    title: 文章分类
    date: 2021-11-22 14:56:48
    type: "categories"
    ---

    hexo new page about
    修改about/index.md 里面的内容为
    ---
    title: 关于
    date: 2021-11-22 14:56:48
    type: "categories"
    ---

    hexo new newpapername

## 内插图片 ##

1.将Hexo目录下的配置文件_config.yml里的psot_asset_folder:设置为true。

2.在Hexo目录下执行:
    npm install https://github.com/CodeFalling/hexo-asset-image
下面的命令有bug
    npm install hexo-asset-image --save

3.执行hexo n "博客名"生成博客，此时在_post目录会生成与 <font face="微软雅黑" size=3 color=#FF0000 >博客同名的文件夹</font>。

4.将博客需要使用的图片移动到<font face="微软雅黑" size=3 color=#FF0000 >博客同名的文件夹</font>中，然后在博客中使用Markdown的格式引用

    ![图片描述文字](图片名.jpg)
