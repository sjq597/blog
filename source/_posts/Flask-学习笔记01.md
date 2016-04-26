title: 'Flask 学习笔记01 '
date: 2016-04-24 18:56:39
tags: [Python,Flask,Web]
categories: Flask
---
由于平时经常使用Python,所以想使用Python的微框架开发一些简单的工具来提升工作效率，特此记录一下学习Flask的笔记。

### 开发环境
本机系统: Ubuntu 14.04 64bits
所有的学习第一步就是配置开发环境，还好Ubuntu自带了Python,这里采用的虚拟Python配置，
```
$ sudo apt-get install python-virtualenv
```
或者使用pip安装:
```
$ $ sudo pip install virtualenv
```
如果你是Windows用户，并且你已经安装了`pip`工具，则去掉`sudo`在cmd命令下也可以安装
然后创建一个包含`venv`的文件夹的项目文件夹，项目就建在这里面
```
$ sudo mkdir flask_project
$ cd flask_project
$ sudo virtualenv venv
```
现在每次要使用项目，就可以在`flask_project`文件夹中运行下面的命令
```
$ . venv/bin/activate
```
你现在进入你的virtualenv(注意查看你的shell提示符已经改变了)。每次需要安装包的时候就先激活虚拟环境，下面安装`flask`
```
pip install flask
```
**注意:**这里有个地方需要个别强调一下，安装包不能使用sudo权限来安装，如果不带`sudo`安装提示权限不够，请把`flask_project`文件夹的权限设置为当前用户,例如，当前用户为`zhangsan`,在`flask_project`的父目录执行:
```
sudo chown zhangsan:zhangsan -R flask_project
```
相应的退出使用:
```
$ deactivate
```

### 一个最简单的应用
在`flask_project`文件夹中创建一个文件`hello.py`
```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```
