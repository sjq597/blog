title: Flask Blueprint笔记
date: 2016-08-07 22:48:19
tags: [Python,Flask,Web]
categories: Flask
---
最近开始用Flask做一个报表系统，为了方便组织代码，网上了解了一下`Flask Blueprint`这个东西，算是一个入门，新手容易碰到的问题。
关于蓝图的介绍网上也很多了，我也不多讲，主要是上代码，然后有常见的问题

### 项目结构
```
➜  script git:(flask-blueprint) ✗ tree -L 2
.
├── config.py
├── env
│   ├── bin
│   ├── include
│   ├── lib
│   ├── local
│   ├── pip-selfcheck.json
│   └── share
├── my_site
│   ├── app1
│   ├── app2
│   ├── app3
│   ├── __init__.py
│   ├── __init__.pyc
│   └── local.db
├── README.md
├── requirements.txt
└── run.py
```
项目的目录结构如上面所示，项目根目录为`scirpt`,真正的项目为`my_site`目录,`app1`,`app2`,`app3`为三个不同的应用，对应为3个不同模块.
`env`文件夹为`virtualenv`虚拟`python`环境安装目录,项目地址为()[]:

### 模块
一般自己写着玩或者写个小网站，就几个访问URL，都写在一个`views.py`里面当然没啥问题，但是项目大了，越来越复杂的话，肯定不可能都写在一个文件里面，极难维护，所以一般会按功能分成不同模块，下面以`app1`模块为例讲解一下:

* my_site/app1/views.py

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from flask import Blueprint, render_template

__author__ = 'anonymous'

app1 = Blueprint('app1', __name__, template_folder='templates')


@app1.route('/index')
def index():
    print '访问app1'
    return render_template('index.html')
```
这就相当于完成了一个模块了，模块里访问的`index.html`模板文件为:

* my_site/app1/templates/index.html

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>应用1</title>
</head>
<body>
    测试1
</body>
</html>
```
完成了模块之后还需要注册模块.

* my_site/__init__.py

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from flask import Flask

__author__ = 'anonymous'

app = Flask(__name__, instance_relative_config=True)

app.config.from_object('config')

from my_site.app1.views import app1
from my_site.app2.views import app2
from my_site.app3.views import app3

app.register_blueprint(app1, url_prefix='/app1')
app.register_blueprint(app2, url_prefix='/app2')
app.register_blueprint(app3, url_prefix='/app3')

```
这里注册了三个模块，为了演示，有三个模块，内容就不多虽说了，`views.py`的内容都是一样的，但是`index.html`稍作区分，以标志我们访问的是哪一个页面.


### 运行
然后在项目的根目录下创建一个`run.py`文件作为主启动文件:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from my_site import app

__author__ = 'anonymous'

if __name__ == '__main__':
    app.run(host='localhost', debug=True)

    pass
```
然后使用虚拟python环境就可以运行了。
