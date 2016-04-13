title: Django 笔记01
date: 2015-12-31 22:58:52
tags: [Python,Web]
categories: Django
---
记录一下关于学习`Django`的一些细节，主要还是网上的一些教程实在是太坑了，可能是早期的版本，所以各种报错，所以最后都是参考了官方的原版教程，以及结合着其他博客慢慢折腾出来的，这里我会尽量把作为一个新手的常用问题记录下来。

### 环境准备
> 
OS:Ubuntu 14.04 64bits
Python: 2.7.6
Django:1.10.dev20151230230702

### 安装Django
方法很多，只讲我的方法
```bash
cd /usr/dev
git clone git://github.com/django/django.git
sudo pip install -e django/
```
**注意:**如果没有装`pip`，可以直接在终端里装：
```bash
sudo apt-get install python-pip
```
最终目录结构看起来就是下面这样：
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```
关于这些文件的用途，大概介绍一下，开始不必过多纠结这个，后面用到了会详细讲：
1. 外层的`mysite/`是工程的根目录，这个名字对于`Django`来说无所谓，你可以随便改个名字。
2. `manage.py`:管家，启动服务器，数据库之类的都需要通过这个来完成。
3. `mysite/__init__.py`：这个空的文件只是为了标示这个文件夹是个`Python`包。
4. `mysite/settings.py`:整个工程的配置文件。
5. `mysite/urls.py`:整个`Django`工程`url`映射。
6. `mysite/wsgi.py`:一个供调试用的小型web服务器，类似Apache，tomcat。

### 启动服务
在外层的`mysite`目录下，在终端中执行下面的命令：
```bash
python manage.py runserver
```
此时就可以在控制台看到成功启动服务器的信息，浏览器访问[http://127.0.0.1:8000/](http://127.0.0.1:8000/)，能访问看到对应的欢迎信息则说明成功了。
再多说一点，如果不指定端口和IP，则默认的就是`127.0.0.1`和`8000`.着意味着你只能是在本地访问这个网址，如果要局域网里其他的机器可以访问你的这个网址，你需要像下面这样启动：
```bash
python manage.py runserver 0.0.0.0:8000
```
这样服务器会监听所有的公共IP地址，你可以通过局域网其他IP地址访问这台机器的ip地址加端口号来访问对应的网址。

### 创建一个应用
首先看看应用和工程有什么区别？我们可以这样理解，一个应用就是完成了一些功能的一个Web程序，一个工程就是由很多应用，或者说模块组成的，各个应用完成自己的功能，组成了功能强大的工程，下面我们来创建一个应用，还是到最外层的`mysite`下：
```bash
python manage.py startapp polls
```
刚刚创建的`polls`文件夹的目录就像下面这样：
```
polls|master⚡ ⇒ tree
.
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
├── urls.py
├── views.py
```
来写我们的第一个视图，编辑`polls/views.py`文件，代码如下：
```python
from django.http import HttpResponse


# Create your views here.
def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
这个是一个最简单的视图了，为了访问到这个视图，我们需要创建一个`URL`映射，编辑`polls/urls.py`文件，最终文件看上去像下面这样：
```python
#!/usr/bin/env python
# coding=utf-8

from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```
光这样还不行，这个只是把访问这个应用时，内部如何处理映射了，但是最外层如何访问到这个内层应用，还需要在`mysite`中配置一个`URL`映射,修改`mysite/urls.py`文件：
```python
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]
```
记得要导入`include`模块。
**注意:**什么时候需要使用`include`模块？当你的URL匹配中包含另外的URL匹配映射时，你需要用`include()`,对于我们的例子，当在处理`127.0.0.1:8000/polls/`这个链接时，需要用到`polls/urls.py`里面的映射，所以我们要用`include()`将其他的匹配规则包含进来。
一切都完成了之后，我们来启动工程：
```bash
python manage.py runserver
```
同样，浏览器访问[http://127.0.0.1:8000/polls/](http://127.0.0.1:8000/polls/),如果看到输出了：
> Hello, world. You're at the polls index.

则说明我们的第一个应用完成了。
