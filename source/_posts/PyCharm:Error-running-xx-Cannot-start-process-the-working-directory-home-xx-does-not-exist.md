title: 'Error running xx: Cannot start process, the working directory /home/xx does not exist'
date: 2015-11-07 22:40:09
tags: [Python, 开发工具]
categories: Python笔记
---
用PyCharm 4.5.4写python代码的时候,改了文件夹的名字,再运行脚本,始终运行不起来,一直报错:
```bash
Error running xx: Cannot start process, the working directory /home/xx does not exist
```
看一下项目的配置,具体是工具栏:`Run-->Edit Configurations`
![项目详细配置](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/PyCharm%3AError%20running%20Cannot%20start%20process01.png)
看了一下`Working directory:`以及`Script:`的内容果然不对,改为对应的脚本以及脚本目录就好了.
