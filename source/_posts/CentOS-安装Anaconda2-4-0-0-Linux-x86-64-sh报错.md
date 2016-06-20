title: CentOS 安装Anaconda2-4.0.0-Linux-x86_64.sh报错
date: 2016-06-19 19:21:47
tags: [Linux, Python, 开发工具]
categories: 开发环境
---
前阵子想给服务器都装上Python的全家桶，结果怎么也装不上,开始安装
```
bash Anaconda2-4.0.0-Linux-x86_64.sh
```
但是装不了，一直报错:
```
mkdir: cannot create directory `/home/q/anaconda2': Permission denied
ERROR: Could not create directory: /home/q/anaconda2
```
网上查了之后发现很多人都碰到这个问题,后来大家都建议这么安装:
```
sudo bash Anaconda2-4.0.0-Linux-x86_64.sh
```
结果报错：
```
Sorry, user xxx is not allowed to execute '/bin/bash Anaconda2-4.0.0-Linux-x86_64.sh' as root on xxxx.
```
我去，简直无解了，没权限执行这个命令，普通的命令又装不了，结果折腾了好久也没办法，后来通过求助之后才发现，原来给这个包附个权限就可以执行了，原来如此,说干就干:
```
sudo chmod 777 Anaconda2-4.0.0-Linux-x86_64.sh
sudo ./Anaconda2-4.0.0-Linux-x86_64.sh
```
后面一路畅通，搞定。

**Tips:**如果想用这个代替默认的环境变量，有些包这里又没有，假设你的安装目录为`/home/usr/anaconda2`,那么你可以这么装:
```
sudo /home/usr/anaconda2/bin/pip install <module_name>
```
