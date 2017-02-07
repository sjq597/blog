title: mysqldb python 错误汇总分析
date: 2017-02-07 18:10:34
tags: [Linux,Python, MySQL]
categories: Python笔记
---

基本上每次在不同的机器上使用`MySQLdb-python`包都会出莫名奇妙的问题，总结一下，基本上就是`fedora,ubuntu,centos`这三个系统上的各种问题，其他系统我没有试过，所以以上是使用的前提。

前段时间折腾`fedora`上安装这个特别麻烦，主要是安装`mysql`比较麻烦，而且好不容易安装好了，设置密码也非常麻烦，本来就是本机做个简单的测试，太简单的密码还不让过,反正吧，用起来挺麻烦的。后来我就不折腾了，又换回了`ubuntu`了。
问题就出在这里，之前一直是用的`virtualenv`虚拟`python`环境做项目开发，`python`环境一直都是直接放在当前项目目录下的，所以有些项目我都放在公共的单独目录，以前一直用`ubuntu`，系统没变过，所以重装系统了虚拟环境都没啥问题,项目都可以直接运行。直到这次把系统从`fedora`切回`ubuntu`终于出现了问题，具体为:明明安装并且识别了`MySQLdb-python`，但是运行项目就报错，开始是由于没有安装`mysql`报错:
```
ImportError: libmysqlclient.so.18: cannot open shared object file: No such file or directory
```
然后安装mysql,装好了之后再运行还是报错,网上
