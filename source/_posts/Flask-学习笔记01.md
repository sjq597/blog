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
对应的Windows启动命令为
```
$ . venv/Scripts/activate
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
在命令行或者终端里运行:
```
$ python hello.py
```
然后在浏览器打开`http://127.0.0.1:5000/`就可以看到一个最简单的hello world页面了。大概解释一下：
1. 首先我们导入了**Flask**类。这个类的实例将会成为我们的**WSGI**应用,即后面的**app**。
2. 接着我们创建了这个类的实例。第一个参数是应用模块或者包的名称。如果你使用一个 单一模块（就像本例），那么应当使用`__name__`,因为名称会根据这个模块是按 应用方式使用还是作为一个模块导入而发生变化（可能是`__main__`，也可能是 实际导入的名称）。这个参数是必需的，这样**Flask**就可以知道在哪里找到模板和 静态文件等东西。不明白也没关系，先这么写，后面慢慢会深入研究。
3. 然后我们使用`route()`装饰器来告诉**Flask**触发函数的**URL**。函数名称可用于生成相关联的**URL**，并返回需要在用户浏览器中显示的信息。这个和Java里的**Controller**非常的像，用过Spring的同学对此应该很熟悉。
4. 最后，使用`run()`函数来运行本地服务器和我们的应用。`if __name__ == '__main__':` 确保服务器只会在使用Python解释器运行代码的情况下运行，而不会在作为模块导入时运行。

本机访问没问题，但是局域网的其他机器如果也想访问，则需要这样启动:
```
app.run(host='0.0.0.0')
```
这行代码告诉你的操作系统监听一个公开的IP 。

### 调试模式
开发程序不可能这么简单，需要不断的调试，所以**Flask**也为我们想好了，专门有个调试模式，这样不用重启服务器，代码有变动，会自动重启，并且出错提示信息也很强大，打开调试器的有两种方式：
1. 通过设置实例的属性
```
app.debug = True
app.run()
```
2. 启动传参设置
```
app.run(debug=True)
```
**注意:**调试器很强大，可以执行任意代码，所以千万不要在项目上线之后打开调试模式。

### 路由route()
路由装饰器用于将一个函数和一个**URL**绑定，例如:
```
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello World'
```
这只是最简单，最基础的，实际开发中不可能都是这么简单的URL,大多数情况下URL里面有一部分是动态变化的，例如好多URL里面会带ID,表示网页的一个标号。

#### 变量规则
<variable_name>里面可以添加变量，<converter:variable_name>可以为变量加一个转换器,目前只支持`int,float,path`最后一个为路径，接受`/`，例如：
```
@app.route('/user/<username>')
def show_user_profile(username):
    # 通过获取URL的user_name作为函数的传入参数
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # 传入post_id，并且必须是一个整数
    return 'Post %d' % post_id
```

#### 唯一URL/重定向问题
看下面一个例子:
```
@app.route('/projects/')
def projects():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```
注意一个尾部有**斜杠**，一个没有。看上去带**斜杠**的URL很像一个文件夹，当我们访问一个没有带**斜杠**的URL的时候，为自动加一个**斜杠**。但是反过来，如果访问第二个URL你在尾部带了一个**斜杠**，则会报错。可能你有些疑惑，为啥要这么做，答案也很简单：这样在访问一个不带**斜杠**的URL时，不管真正的URL是否带**斜杠**我们都可以继续访问URL,并且URL是唯一的。

#### URL构建
设想一下，你想先设定好页面的访问URL,你需要测试你写的函数能不能被你指定的URL访问到，直接构建这些URL就可以了。`url_for`它把函数名称作为第一个参数，其余参数对应URL中的变量。未知变量将添加到URL中作为查询参数。 例如：
```
from flask import Flask, url_for
app = Flask(__name__)
@app.route('/')
def index(): pass

@app.route('/login')
def login(): pass

@app.route('/user/<username>')
def profile(username): pass

with app.test_request_context():
	print url_for('index')
	print url_for('login')
	print url_for('login', next='/')
	print url_for('profile', username='John Doe')
```
