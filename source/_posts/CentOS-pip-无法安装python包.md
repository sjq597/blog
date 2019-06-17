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
The repository located at pypi.xxx.xxx.com is not a trusted or secure host and is being ignored. If this repository is available via HTTPS it is recommended to use HTTPS instead, otherwise you may silence this warning and allow it anyways with '--trusted-host pypi.abc.def.com'.

依然按照建议加上参数：
```bash
sudo pip install --trusted-host pypi.abc.def.com --upgrade sqlalchemy
```
然后就安装成功了。

有时候升级pip会报这个错:
```
The pip==7.1.0' distribution was not found and is required by the application
```
详细解决方案如下:
```
[root@xxx ~]# easy_install pip
Searching for pip
Best match: pip 9.0.1
Adding pip 9.0.1 to easy-install.pth file
Installing pip script to /usr/local/python2.7/bin
Installing pip3.5 script to /usr/local/python2.7/bin
Installing pip3 script to /usr/local/python2.7/bin

Using /usr/local/python2.7/lib/python2.7/site-packages
Processing dependencies for pip
Finished processing dependencies for pip
```
然后看看系统的pip是哪个地方:
```
[root@xxx ~]# which pip
/usr/bin/pip
```
然后我们需要编辑一下这个文件`vi /usr/bin/pip`:
```
#!/usr/bin/python
# EASY-INSTALL-ENTRY-SCRIPT: 'pip==7.1.0','console_scripts','pip'
__requires__ = 'pip==9.0.1'
import sys
from pkg_resources import load_entry_point

if __name__ == '__main__':
    sys.exit(
        load_entry_point('pip==9.0.1', 'console_scripts', 'pip')()
    )

```
就是把pip改成你系统安装的pip,然后就可以安装包了．
