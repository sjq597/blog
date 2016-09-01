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

