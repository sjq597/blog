title: 'Python file() argument 1 must be encoded string without NULL bytes, not str'
date: 2015-12-17 23:17:57
tags: Python
categories: Python笔记
---
心血来潮想写个`tcp`的服务端和客户端来向服务器传文件，其中为了让服务器在我正式传文件之前知道我所要传文件的详细信息，我用了`struct.pack()`函数，给文件名留了256个字节的内容。
这个时候问题就来了，一般我传的文件名称并没有那么长，但是这样名字是可以传过去，但是每次一调用`fp = open(file_path, 'wb')`就报错：
```
Python 2.7 TypeError: file() argument 1 must be encoded string without NULL bytes, not str
```
打印出来确实没有问题，是我的文件名，那到底是哪里出问题了？
仔细看报错信息,大概明白了，原来这里有个坑，Python是用C写的，所以Python里的字符串和C里面的`char[]`数组一样，是以`\0`结尾的，所以当你的路径里含有这些空值的时候，打开会报错，可以验证一下：
```
repr(file_path)
Python/socket/udp/Django-1.8.5.tar.gz\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
```
看到没有，果然有很多这种值，原来在调用`struct.pack()`函数的时候，不足的地方会填上这些空的东西，`print`是不会显示的，但是在打开文件的时候就悲剧了。
解决办法：
直接把这些空的去掉
```
file_path.strip('\0')
```