title: Linux 传输文件-nc+pv
date: 2018-05-13 11:01:18
tags: [Linux, Shell]
categories: Linux使用
---
做数据的写代码多了,会经常碰到传输文件的需求,之前还好，一般是下载文件,直接用python内置的server起个服务就搞定了.但是对于跨机房有防火墙存在的情况，一般数据是单向的，就是假设(A-->B)A作为HTTPServer,B可以下载文件.但是反过来就不好使了，因为防火墙策略没有开,(B-->A)用B作为HTTPServer,A无法访问到服务.所以这个时候,A仍然得作为服务端.这里涉及到两个概念，可以简单了解下.

正向代理和反向代理(稍后补充)

* nc传输文件

1.正向传输:A->B
简单来说就是接收端监听本机指定端口的数据,发送端发送到指定接收端端口.
即:B监听本机指定端口,A向B指定端口发送数据
```
B: nc -l 12345 | sudo tar -zxvf -
A: tar -zcvf - file/directory | nc -l {B_IP} 12345
```
**PS:**注意如果是想B-A传输,对调一下就行,注意服务的开启顺序

2.反向传输:A->B
这个和正向不大一样,就是发送端发送到本机指定端口,接收端监听发送端指定端口数据
B监听A指定端口数据,A数据发送到本机指定端口
```
A: nc -l 12345 < file/directory
B: nc {A_IP} 12345 > file/directory
```
**PS:**注意和上面的顺序不要搞混了

* 配合pv使用

pv我就不介绍是啥了,这个小工具也非常的好用,因为一般传输大文件的时候，我们希望看到传输进度啊,速度啊之类的,还有一个很重要的功能就是限速,尤其是专线跨机房问题.
A:pv -p -r -L 10m heap.bin | nc -l 9099
B:nc {A_IP} 9099 > heap.bin
常用参数:
```
-p, --progress           show progress bar
-r, --rate               show data transfer rate counter
-L, --rate-limit RATE    limit transfer to RATE bytes per second
```

