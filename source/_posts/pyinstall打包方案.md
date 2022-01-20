---
title: pyinstall打包方案
date: 2021-12-23 10:37:20
tags: python
categories: 编程类
description: pyinstall打包的一点经验
---

## pyinstall打包方案

### 简单打包

安装pyinstaller

    pip install pyinstaller

切到需要打包的文件`Target.py`目录下，运行

    pyinstaller [<args>] Target.py

常用参数用法：

    --distpath <path>: 打包到哪个目录下
    
    -w: 指定生成 GUI 软件，也就是运行时不打开控制台
    
    -c: 运行时打开控制台
    
    -i <Icon File>: 指定打包后可执行文件的图标
    
    --clean: 在构建之前清理PyInstaller缓存并删除临时文件

打包类型分为单个文件和文件夹：

    -D: 创建包含可执行文件的单文件夹包，同时会有一大堆依赖的 dll 文件，这是默认选项
    
    -F: 只生成一个 .exe 文件，如果项目比较小的话可以用这个，但比较大的话就不推荐

运行时可能报文件丢失的错误，因为打包的时候配置文件之类的并不会一并打包过去，我们手动复制进打包目录下就行。

### 文件过大问题

如果是在Anacanda的环境下进行打包，会把很多不相干的库也打包进来，造成文件变得很大，随随便便就是几百兆，因此，我们需要用纯净的环境进行打包，来减少那些不相干的东西。

Pipenv 是一款管理虚拟环境的命令行软件，简单来讲，它可以创建一个只在某个目录下的局部 Python 环境，而这个环境是可以和全局环境脱离开的。

首先，安装pipenv

    pip install pipenv

运行

    pipenv install [--python 3.9.7]

install后面的参数可以缺省或者指定你安装的python版本

![install](pipenv_install.png)

这样虚拟环境就创建成功了，使用命令`pip list`可以发现里面只有很少的库。

在虚拟环境下安装pyinstaller和依赖库：

    pipenv install pyinstaller

    pipenv install [libname]

安装完成后先运行一下，如果没问题，再进行打包。

打包完毕后，发现，文件少了很多，从几百兆减少到几十兆，这样我们就打包完成了。

## 其它注意事项

打包过程中可能会遇到没有upx的问题，可以再github上面下载下来,复制upx.exe放到pyinstaller.exe对应的文件目录下。
