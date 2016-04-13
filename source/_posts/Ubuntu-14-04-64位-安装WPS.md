title: Ubuntu 14.04 64位 安装WPS
date: 2015-12-29 13:25:18
tags: [Linux]
categories: Linux使用
---
经常要处理数据,自带的那个`LibreOffice`看看数据还行,但是要自己统计,我还是不太会用,所以就装了个`WPS for Linux`.之前也装过一次,但是装完了启动不了,今天要做统计,所以就再装了一下.

首先要下载安装包[wps-office_10.1.0.5444~a20_amd64.deb](http://kdl.cc.ksosoft.com/wps-community/download/a20/wps-office_10.1.0.5444~a20_amd64.deb)

直接点开就可以安装,但是正常情况下会安装失败,缺少依赖啥的,所以要装依赖库`libc6-i386`:
```bash
$ sudo apt-get update
$ sudo apt-get install libc6-i386
```

安装成功之后再重新安装`WPS`的安装包,装好了就可以使用了,记住,刚开始启动可能会有个警告报错,缺少字体,下载对应的字体就可以了,当然不下载也是可以使用的,没有什么多大的区别.
