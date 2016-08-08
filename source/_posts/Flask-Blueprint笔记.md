title: Flask Blueprint笔记
date: 2016-08-07 22:48:19
tag`s: [Python,Flask,Web]
categories: Flask
---
最近开始用Flask做一个报表系统，为了方便组织代码，网上了解了一下`Flask Blueprint`这个东西，算是一个入门，新手容易碰到的问题。
关于蓝图的介绍网上也很多了，我也不多讲，主要是上代码，然后有常见的问题

### 项目结构
```
➜  script git:(3a429f0) tree -L 4
.
├── my_site
│   ├── app1
│   │   ├── __init__.py
│   │   ├── templates
│   │   │   └── index.html
│   │   └── views.py
│   ├── app2
│   │   ├── __init__.py
│   │   ├── templates
│   │   │   └── index.html
│   │   └── views.py
│   ├── app3
│   │   ├── __init__.py
│   │   ├── templates
│   │   │   └── index.html
│   │   └── views.py
│   └── __init__.py
├── README.md
└── run.py
```
项目的目录结构如上面所示，项目根目录为`scirpt`,真正的项目为`my_site`目录,`app1`,`app2`,`app3`为三个不同的应用，对应为3个不同模块.
`env`文件夹为`virtualenv`虚拟`python`环境安装目录.

### 准备工作
安装虚拟Python环境，这个不多说，网上教程一堆，因为我用的Pycharm IDE,所以在Pycharm里面设置成我的安装环境就行。具体方法:`File-->Settings-->Project-->Project Interpreter`,选你安装虚拟环境的地址即可。

### 模块
一般自己写着玩或者写个小网站，就几个访问URL，都写在一个`views.py`里面当然没啥问题，但是项目大了，越来越复杂的话，肯定不可能都写在一个文件里面，极难维护，所以一般会按功能分成不同模块，下面以`app1`模块为例讲解一下:

* `my_site/app1/views.py`

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

* `my_site/app1/templates/index.html`

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

* `my_site/__init__.py`

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
然后直接用IDE运行`run.py`文件就行。

### 访问
然后通过浏览器访问`http://localhost:5000/app1/index.html`，结果返回的值为:
```
测试3
```
问题有点儿奇怪，通过设置断点,请求确实进入了`app1/views.py`文件，但是返回的却是`app3/index.html`的内容，上网查了一下，发现这个问题好像还挺常见的，官方也没把这个问题定义成bug,这个就涉及到`Flask`的`render_template()`函数在查找模板的时候是如何处理模板的了，默认是在项目的目录下查找`template`文件夹,如果没找到，就去模块下找，最后会把所有模块下找到的模板文件的路径加到一个字典文件里面，因为字典是无序的，所以具体会返回哪个页面得看字典的`hash`算法了,核心模块代码如下:

* `python2.7/site-packages/flask/templating.py`

```
def get_source(self, environment, template):
    if self.app.config['EXPLAIN_TEMPLATE_LOADING']:
        return self._get_source_explained(environment, template)
    return self._get_source_fast(environment, template)

def render_template(template_name_or_list, **context):
    """Renders a template from the template folder with the given
    context.

    :param template_name_or_list: the name of the template to be
                                  rendered, or an iterable with template names
                                  the first one existing will be rendered
    :param context: the variables that should be available in the
                    context of the template.
    """
    ctx = _app_ctx_stack.top
    ctx.app.update_template_context(context)
    return _render(ctx.app.jinja_env.get_or_select_template(template_name_or_list),
                   context, ctx.app)
```

针对这个情况，有两种解决办法:

* 官方解决方案:

在每个模块的`template`文件夹下面的模板文件不要重名，怎么做呢？很简单，就是以模块名再建一个文件夹，把所有的模板文件放到这个文件夹下面，最后的结构可能就变成了:
```
├── config.py
└─── my_site
    ├── app1
    │   ├── __init__.py
    │   ├── templates
    │   │   └── app1
    │   │       └── index.html
    │   └── views.py
    └── app2
        ├── __init__.py
        ├── templates
        │   └── app2
        │       └── index.html
        └── views.py
```
所以对应的`views.py`文件中需要改成:
```
return render_template('app1/index.html')
```

* 第三方解决方案

既然已经知道问题出在处理模板的文件`templating.py`文件中，所以在对应额地方加上处理逻辑即可，github上的一位道友给出的解决方案,修改`python2.7/site-packages/flask/templating.py`:
```
def render_template(template_name_or_list, **context):
    """Renders a template from the template folder with the given
    context.
    :param template_name_or_list: the name of the template to be
                                  rendered, or an iterable with template names
                                  the first one existing will be rendered
    :param context: the variables that should be available in the
                    context of the template.
    """
    ctx = _app_ctx_stack.top
    ctx.app.update_template_context(context)

    template = None

    if _request_ctx_stack.top is not None and \
            _request_ctx_stack.top.request.blueprint is not None and \
            isinstance(template_name_or_list, string_types):
        bp = ctx.app.blueprints[_request_ctx_stack.top.request.blueprint]
        if bp.jinja_loader is not None:
            try:
                template = bp.jinja_loader.load(ctx.app.jinja_env,
                                                template_name_or_list,
                                                ctx.app.jinja_env.globals)
            except TemplateNotFound:
                pass

    if template is None:
        template = ctx.app.jinja_env\
            .get_or_select_template(template_name_or_list)

    return _render(template, context, ctx.app)
```
然后再访问就可以了。
**注意:**两种方式都可以，但是推荐使用第一种官方的方案，因为如果改源码，虽然可以正常运行，但是换个环境，或者项目重新给别人部署，如果不加说明，不知道的人不会去改`Flask`代码,那么这个项目则会出错。
