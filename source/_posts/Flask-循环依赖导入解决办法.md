title: Flask 循环依赖导入解决办法
date: 2016-08-23 20:27:45
tags: [Python,Flask]
categories: Python笔记
---
今天在是使用Flask的`Flask-login`搭建一个系统的时候，在登陆`views.py`视图文件里面引用`models.py`实体类的时候出现了一个错误:
```
ImportError: cannot import name db
```
看看项目的文件结构:
```
➜  financial_bi git:(dev) ✗ tree -L 3
.
├── config.py
├── instances
│   └── dev_init.sql
├── README.md
├── requirements.txt
├── run.py
└── web
    ├── common
    │   └── __init__.py
    ├── __init__.py
    ├── security
    │   ├── __init__.py
    │   ├── models.py
    │   ├── templates
    │   └── views.py
    ├── static
    │   ├── assets
    │   └── favicon.ico
    └── templates
        └── layout

9 directories, 15 files
```
代码如下:
```
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from web.security.views import security

app = Flask('web', static_folder='static')
app.config.from_pyfile('../config.py')

db = SQLAlchemy(app)


def create_app():
    # 注册模块
    app.register_blueprint(security, url_prefix='')

    configure_flask_login(app)

    return app
```
看到这里确实有问题，我们来梳理一下这个引用关系:
![循环依赖图](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Flask%20%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E5%AF%BC%E5%85%A5%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%9501.png)
我们对照着这个图分析一下怎么产生循环依赖的:
```
1. __init__.py 先import web/security/views.py,然后声明变量db
2. web/security/views.py依赖web/security/models.py的User实体类
3. web/security/models.py依赖__init__.py中声明的变量db
```
结果就导致了这个问题，所以我们要想解决这个问题，可以在`__init__.py`中把变量`db`的声明顺序放到`import web/security/views.py`之后，如此一来`__init__.py`的`db`变量就不会依赖其他模块了.
所以我们得出一个技巧，在用蓝图注册模块的时候，把引用放到工厂函数里面去，像下面这样:
```
def create_app():
    # 注册模块
    from web.security.views import security

    app.register_blueprint(security, url_prefix='')

``` 
这样就不会有循环依赖的问题了。
