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
运行程序会输出以下内容:
```
/
/login
/login?next=/
/user/John%20Doe
```
这里用到了一个方法`test_request_context()`,这个会告诉`Flask`我们正在处理一个请求，虽然我们并没有真正的去请一个URL.

#### HTTP方法
访问一个URL有好几个方法，常见的就是get，post方法,缺省情况下一个路由只回应GET请求，也可以像下面这样手动指定：
```
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```

#### 静态文件
一般就是指css,js文件，如果工程里面要用到这些静态的库，可以在应用的根目录下新建`static`文件夹,设用选定的`static`端点就可以生成对应的URL：
```
url_for('static', filename='style.css')
```
因此该文件对应的在文件系统中的路径应该是`static/style.css`

#### 渲染模板
这是个很强大的功能，基本上目前的框架都得支持这个，毕竟原生的写HTML很慢，还要考虑各种转义，乱码等，Flask内置Jinja2模板引擎。使用`render_template()`来渲染模板,只需要把模板的名字和对应的一些参数传进去就行了，例如:
```
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```
Flask默认会在`templates`文件夹内寻找模板，如果你的应用是一个模块,则`templates`文件夹应该和应用在一个目录下,即:
```
/application.py
/templates
    /hello.html
```
如果你的应用是一个包,则应该在包里面:
```
/application
    /__init__.py
    /templates
        /hello.html
```
这个就类似预JSP一类的东西，你也可以在模板内部访问**request, session, get_flashed_messages()`等，不过比JSP更加强大,因为模板可以实现继承，这样就能保持很多特定的元素重用，保持一致。
默认特殊的一些变量会被转义，例如HTML,如果信任某个变量，也可以使用`Markup`类来把它标记为安全的,具体的等学到了再详细研究。

### 操作请求数据
一个Web应用其实就是服务器相应客户端发来的消息，Flask中由全局对象`request`来提供请求信息，如果你熟悉Python你就会知道，全局的对象大家都可以访问，其实也并不是通常意义上的全局变量，这个request只是特定环境下的本地对象的一个代理。

#### 请求对象
首先必须从flask模块导入请求对象`from flask import request`,使用`method`属性可以操作当前请求的具体方法，`form`可以处理表单数据,例如:
```
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # 如果请求访求是 GET 或验证未通过就会执行下面的代码
    return render_template('login.html', error=error)
```
如果form属性中不存在这个键值，会像普通集合那样抛出一个`KeyError`的异常。如果不捕获的话，就会显示一个`HTTP 400 Bad Request`错误页面，虽然你可以不用处理这个异常，但是显然不是很友好，所以推荐捕获异常，然会一个预定义好的错误页面.
如果要处理URL中的参数，可以使用`request.args.get('key', '')`来获取.

#### 文件上传
使用Flask实现文件上传很简单，需要在HTML表单中设置`enctype="multipart/form-data`属性即可，上传的文件被存储在内存或者文件系统临时位置，通过请求对象files属性可以访问上传文件,看下面的例子：
```
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
```
要想知道文件上传之前在客户端系统的名字，可以使用`filename`属性，但是这个可以伪造，所以建议通过`Werkzeug`提供的`secure_filename()`函数：
```
from flask import request
from werkzeug import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/' + secure_filename(f.filename))
```

#### Cookies
对于Web应用来说，这个必不可少，要访问cookeis,可以使用`cookies`属性。通过请求对象的`set_cookies`方法设置`cookies`，请求对象的`cookies`包含了客户端的所有cookies字典，所以这很不安全，能使用会话就不要直接使用cookies.
* 读cookies:
```
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # 使用 cookies.get(key) 来代替 cookies[key] ，
    # 以避免当 cookie 不存在时引发 KeyError 。
```
* 存储cookies:
```
from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```
**注意:**cookies设置在响应对象上，通常只是视图函数返回字符串，Flask会把它们转化为响应对象,显示的转化可以使用`make_response()`函数,然后再修改对应的值.

### 重定向
`redirect()`函数可以重定向,`abort()`可以更早的退出请求，返回错误码:
```
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```
`401`表示一个无法访问的页面,缺省情况下每种错误代码都会对应显示一个黑白的出错页面。使用`errorhandler()`装饰器可以定制出错页面:
```
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

#### 关于响应
视图函数的返回值会自动转化为一个响应对象。如果你返回一个字符串，那么就会被转换为一个响应对象，其中包含这个字符串以及一些其他的必要信息，如果想在视图内部掌控响应对象的结果，可以使用一个`make_response()`函数进行强制转换，例如原始视图:
```
@app.errorhandler(404)
def not_found(error):
    return render_template('error.html'), 404
```
可以手动包装，修改某些特定的内容，例如头部信息：
```
@app.errorhandler(404)
def not_found(error):
    resp = make_response(render_template('error.html'), 404)
    resp.headers['X-Something'] = 'A value'
    return resp
```

#### 会话
除了请求对象和响应对象之外，还有个`session`这个对象，这个主要是用来在不同请求之间存储信息的。你可以简单理解为用密钥签名加密的cookie,cookie都可以查看，但是如果没有密钥就无法修改。所以使用会话之前必须设置一个密钥：
```
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # 如果会话中有用户名就删除它。
    session.pop('username', None)
    return redirect(url_for('index'))

# 设置密钥，复杂一点：
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
```
`escape()`可以用来转义，如果你使用模板就简单的多，不用管这些。
生成密钥要保证做够随机,例如根据操作系统来生成一个密钥:
```
import os
os.urandom(24)
```
基于`cookies`的会话，Flask其实是把会话对象里面的值都放在了cookies里面，如果你访问的会话里没有对应的属性。

#### 消息闪现
啥意思？简单来说就是在你请求结束的时候记录一个消息，然后你下次再请求的时候可以使用，然后用完就销毁了，所以叫闪现。`flash()`用于闪现一个消息，`get_flashed_messages()`来操作一个消息，具体的不介绍了，这个用的不是很多，用到再研究。

#### 日志
一个Web应用当然少不了日志，万一崩了，直接看日志定位问题，使用也很简单:
```
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```
