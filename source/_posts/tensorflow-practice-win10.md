---
title: tensorflow practice (win10)
date: 2021-12-27 11:04:10
updated: 2021-12-27 11:04:10
tags: 
	- tensorflow
	- python
categories: 深度学习
cover: cover.png
description: 在tensorflow框架下练习深度学习(一)
keywords: tensorflow
---

# 环境搭建

# 安装cpu版本的tensorflow

>安装配置官网讲的很详细，可以自行查看[windows下构建tensorflow](https://tensorflow.google.cn/install/source_windows)

>或者参考github上的install guide:

To install the current release, which includes support for CUDA-enabled GPU cards (Ubuntu and Windows):

	$ pip install tensorflow

A smaller CPU-only package is also available:

	$ pip install tensorflow-cpu

# test

	$ python

	>>> import tensorflow as tf
	>>> tf.add(1, 2).numpy()
	3
	>>> hello = tf.constant('Hello, TensorFlow!')
	>>> hello.numpy()
	b'Hello, TensorFlow!'


----------
>自己搭建环境可能会有各种问题，建议直接在anaconda上新建一个tensorflow的环境使用。

>环境搭建好了之后，就可以根据官网上面的详细[教程](https://tensorflow.google.cn/tutorials/quickstart/beginner?hl=zh_cn)，一步步进行学习了

![](start-practice.png)

