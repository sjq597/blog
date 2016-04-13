title: CentOS pip 无法安装python包
date: 2016-02-25 20:56:44
tags: [Linux, Python]
categories: Python笔记
---
最初源于一个错误，在使用python包`sqlalchemy`执行sql统计的时候，报错：
> This result object is closed sqlalchemy

网上查阅了一下，貌似是多个线程共同使用一个会话或者连接导致，由于之前在其他机器上使用的1.0.8版本同样的sql都没有报过错，我直接用yum安装的0.7.x版本，可能是这个版本比较老，于是我想重新使用pip安装最新版，由于之前升级python，pip是我编译安装的最新版，CentOS自带的是1.4.x版本，我安装的是7.1.2，所以才会有下面的各种问题。
```
sudo pip install sqlalchemy
```
报错：
> 
Requirement already satisfied (use --upgrade to upgrade): sqlalchemy in /home/q/python27/lib/python2.7/site-packages
You are using pip version 7.1.2, however version 8.0.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.

于是加上建议重新安装：
```bash
sudo pip install --upgrade sqlalchemy
```
仍然报错：
> 
The repository located at pypi.corp.qunar.com is not a trusted or secure host and is being ignored. If this repository is available via HTTPS it is recommended to use HTTPS instead, otherwise you may silence this warning and allow it anyways with '--trusted-host pypi.abc.def.com'.

依然按照建议加上参数：
```bash
sudo pip install --trusted-host pypi.abc.def.com --upgrade sqlalchemy
```
然后就安装成功了。
