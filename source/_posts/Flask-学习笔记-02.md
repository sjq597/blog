title: Flask 学习笔记 02
date: 2016-04-30 14:03:37
tags: [Python,Flask,Web]
categories: Flask
---
从简单的例子学习Flask,因为是简单的例子，就用子单的`sqlite3`数据库了，如果项目大了，数据量大可以考虑使用`MySQL`。

### 项目步骤
一步步来演示一个项目是如何创建的

* 先创建文件夹
```
➜  flasker  tree
.
├── static
└── templates
```
#### 数据库模式，将下面的内容保存在`schema.sql`，放在`flasker`目录下就行:
```
drop table if exists entries;
create table entries (
  id integer primary key autoincrement,
  title text not null,
  text text not null
);
```
#### 应用构建
创建应用模块，吧模块命名为`flaskr.py`,就放在flaskr文件夹中，为了方便学习，把库的导入和相关配置放在了一起，但是一个清晰的方案应该是放在一个独立的`__init__.py`文件中，然后在模块里导入配置，不过这个也有一个不好的地方，就是你在主文件里不知道的包到底是从哪导入的,flaskr.py内容如下:
```
# all the imports
import sqlite3
from flask import Flask, request, session, g, redirect, url_for, \
     abort, render_template, flash

# configuration
DATABASE = '/tmp/r.db'
DEBUG = True
SECRET_KEY = 'development key'
USERNAME = 'admin'
PASSWORD = 'default'
```
然后在同一个文件中创建真正的应用，使用配置来初始化：
```
# create our little application :)
app = Flask(__name__)
app.config.from_object(__name__)
```
`from_object`的传入参数如果是字符串则直接导入，它会搜索加载多有大写的变量名，就是我们写在最前面的，你也可以把这个写在`__init__.py`文件中。一般都是总配置文件导入配置的，建议使用`from_envvar()`来导入配置，上面的第二行可以替换为:
```
app.config.from_envvar('FLASKR_SETTINGS', silent=True)
```
这个可以设置一个`FLASKR_SETTINGS`变量来指定一个配置文件，并根据该文件来重载缺省配置，`silent`意思是如果没有，则不报错。
然后添加一个用于连接数据库的方法。
```
def connect_db():
    return sqlite3.connect(app.config['DATABASE'])
```
然后在最后以单机模式启动的代码:
```
if __name__ == '__main__':
    app.run()
```
这样虽然可以启动服务器，但是无法访问界面，因为没有构建任何视图。

#### 创建数据库
每次去执行命令导入不是很方便，受限于系统，所以添加一个数据库初始化函数，首先要导入`contextlib.closing()`函数,即:
```
from contextlib import closing
```
然后创建一个初始数据库的函数`init_db()`:
```
def init_db():
    with closing(connect_db()) as db:
        with app.open_resource('schema.sql', mode='r') as f:
            db.cursor().executescript(f.read())
        db.commit()
```
`closing()`函数允许我们在with代码块中保持数据库打开，然后`open_resouce()`也支持这个功能，直接在with代码块中使用，`sqlite3`里面的sql都必须显示的提交才会生效。

#### 请求数据库连接
当然你可以生成一个全局的数据库连接句柄，这样在每个函数里面就可以使用数据库连接了，但是并不推荐这样，会带来很多问题，也不够优雅。Flask里面利用装饰器能够做到优雅的访问数据库，`before_request()`,`after_request()`,`teardown_request()`这三个装饰器就可以满足需求：
```
@app.before_request
def before_request():
    g.db = connect_db()

@app.teardown_request
def teardown_request(exception):
    db = getattr(g, 'db', None)
    if db is not None:
        db.close()
    g.db.close()
```
来看看这段代码是如何的优雅，使用`before_request()`装饰函数会在请求之前调用，不用传递任何参数，这样在每个视图函数里面都可以通过全局`g`对象获取数据库连接句柄。使用`after_request()`会在请求之后调用，并且会传递相应对象给客户端，所以出错了就不会执行。因此需要用到第三个装饰器`teardown_request()`装饰器，这个装饰器会在响应对象构建完之后才调用被装饰的函数，不允许修改请求，而且返回值也会被忽略，如果出错了，这个错误会传递给每个函数。这里的`g`对象简单理解就是一个神奇的全局对象，并且多线程也可以正常工作。

#### 视图函数
在数据库连接处理完之后，就可以来构造视图了，下面简单介绍一个例子：
##### 显示条目
这个视图将会显示所有数据库中的连接，模板为`show_entries.html`,并返回渲染结果:
```
@app.route('/')
def show_entries():
    cur = g.db.execute('select title, text from entries order by id desc')
    entries = [dict(title=row[0], text=row[1]) for row in cur.fetchall()]
    return render_template('show_entries.html', entries=entries)
```
#### 添加条目
添加一条新记录，添加完之后并不会显示，结果显示在`show_entries`页面中，如果成功，则会flash()一个消息给下一个请求并重定向到`show_entries`页面：
```
@app.route('/add', methods=['POST'])
def add_entry():
    if not session.get('logged_in'):
        abort(401)
    g.db.execute('insert into entries (title, text) values (?, ?)',
                 [request.form['title'], request.form['text']])
    g.db.commit()
    flash('New entry was successfully posted')
    return redirect(url_for('show_entries'))
```
这里还有个检查是否登陆的，就是`logged_in`是否为`True`。另外一个，为了防止SQL注入，劲量不要拼sql,用`?`代替。

#### 登陆和注销
用于登陆和注销，根据配置中的用户名和密码验证用户会话中设置`logged_in`的键值。如果通过验证，则设置`logged_in`为True,然后重定向到`show_entries`页面。另外闪现一个信息，告诉用户已登陆成功，如果出错，则提示错误信息，并重新登陆:
```
@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != app.config['USERNAME']:
            error = 'Invalid username'
        elif request.form['password'] != app.config['PASSWORD']:
            error = 'Invalid password'
        else:
            session['logged_in'] = True
            flash('You were logged in')
            return redirect(url_for('show_entries'))
    return render_template('login.html', error=error)
```
注销视图则会相反，移除键值：
```
@app.route('/logout')
def logout():
    session.pop('logged_in', None)
    flash('You were logged out')
    return redirect(url_for('show_entries'))
```
使用`pop()`函数如果传递了第二个参数(键的缺省值),如果有的话就会删掉，如果没有，就啥也不做。

### 模板
光有视图还不够，上面用到的模板还没有写好，访问也会报错，然后还用到了模板继承保存所有页面布局统一，这些文件都保存在`templates`文件夹中:

#### layout.html
这个模板包含了HTML的骨架，头部和一个登陆链接(如果用户已登陆则变为一个注销连接)。如果有闪现消息，则也显示出来:
```
{% block body %}
```
上面的块儿会被子模块中同名的body替换。而且session在模板中也可以使用，如果键值(属性)不存在也可以正常运行：
```html
<!doctype html>
<title>Flaskr</title>
<link rel=stylesheet type=text/css href="{{ url_for('static', filename='style.css') }}">
<div class=page>
  <h1>Flaskr</h1>
  <div class=metanav>
  {% if not session.logged_in %}
    <a href="{{ url_for('login') }}">log in</a>
  {% else %}
    <a href="{{ url_for('logout') }}">log out</a>
  {% endif %}
  </div>
  {% for message in get_flashed_messages() %}
    <div class=flash>{{ message }}</div>
  {% endfor %}
  {% block body %}{% endblock %}
</div>
```
#### show_entries.html
```
{% extends "layout.html" %}
{% block body %}
  {% if session.logged_in %}
    <form action="{{ url_for('add_entry') }}" method=post class=add-entry>
      <dl>
        <dt>Title:
        <dd><input type=text size=30 name=title>
        <dt>Text:
        <dd><textarea name=text rows=5 cols=40></textarea>
        <dd><input type=submit value=Share>
      </dl>
    </form>
  {% endif %}
  <ul class=entries>
  {% for entry in entries %}
    <li><h2>{{ entry.title }}</h2>{{ entry.text|safe }}
  {% else %}
    <li><em>Unbelievable.  No entries here so far</em>
  {% endfor %}
  </ul>
{% endblock %}
```
我们在第一行使用了`layout.html`模板，扩展了基础模板，用于显示信息。for遍历了我们通过`render_template()`函数所有传递信息。模板也指明了method为post提交数据。

#### login.html
```
{% extends "layout.html" %}
{% block body %}
  <h2>Login</h2>
  {% if error %}<p class=error><strong>Error:</strong> {{ error }}{% endif %}
  <form action="{{ url_for('login') }}" method=post>
    <dl>
      <dt>Username:
      <dd><input type=text name=username>
      <dt>Password:
      <dd><input type=password name=password>
      <dd><input type=submit value=Login>
    </dl>
  </form>
{% endblock %}
```

### 添加样式
现在数据库连接有了，视图也有了，然后模板也有了，最后就差样式了，我们来添加一下样式，保存为`style.css`保存在`static`文件夹中
```
body            { font-family: sans-serif; background: #eee; }
a, h1, h2       { color: #377ba8; }
h1, h2          { font-family: 'Georgia', serif; margin: 0; }
h1              { border-bottom: 2px solid #eee; }
h2              { font-size: 1.2em; }

.page           { margin: 2em auto; width: 35em; border: 5px solid #ccc;
                  padding: 0.8em; background: white; }
.entries        { list-style: none; margin: 0; padding: 0; }
.entries li     { margin: 0.8em 1.2em; }
.entries li h2  { margin-left: -1em; }
.add-entry      { font-size: 0.9em; border-bottom: 1px solid #ccc; }
.add-entry dl   { font-weight: bold; }
.metanav        { text-align: right; font-size: 0.8em; padding: 0.3em;
                  margin-bottom: 1em; background: #fafafa; }
.flash          { background: #cee5F5; padding: 0.5em;
                  border: 1px solid #aacbe2; }
.error          { background: #f0d6d6; padding: 0.5em; }
```

### 测试Flask
这个必须介绍一下，因为写代码的过程中免不了要调试和测试，现在讲一下怎么用`unittest`包来测试flask应用

#### 测试骨架
为了测试我们的应用，我们添加一个新的模块`flaskr_tests.py`:
```
import os
import flaskr
import unittest
import tempfile

class FlaskrTestCase(unittest.TestCase):

    def setUp(self):
        self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        flaskr.app.config['TESTING'] = True
        self.app = flaskr.app.test_client()
        flaskr.init_db()

    def tearDown(self):
        os.close(self.db_fd)
        os.unlink(flaskr.app.config['DATABASE'])

if __name__ == '__main__':
    unittest.main()
```
稍微介绍一下这个测试是个啥意思:
1. `setUp()`方法中会创建一个新的测试客户端并出书画了一个连接，每一个独立的测试函数运行之前都要调用这个函数
2. `tearDown()`功能是在测试结束以后关闭文件，并在文件中删除数据库库文件，另外`TESTING=True`,这意味着在请求时关闭错误捕捉，这样可以真实的捕捉到错误，得到更好的错误报告。

测试客户端会提供一个简单的应用接口，我们通过这个接口向应用发送测试请求，还可以追踪cookies。
因为`sqlite3`是一个文件系统数据库，所以可以使用临时文件来创建一个临时数据库并初始化它。`mkstemp()`函数返回两个东西:一个低级别的文件句柄和一个随机文件名,这个文件名将会作为我们的数据库名称。必须把句柄保存到`self.db_fd`种，这样在整个测试类里面才能在其他方法中来关闭文件。
可以在终端中运行测试程序，如果没有报错，才说明没有语法错误：
```
(env)➜  flasker  python flaskr_tests.py 

----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
```

#### 第一个测试
Web应用就是测试一些URL访问是否正常，添加一个访问URL(/)的方法：
```
class FlaskrTestCase(unittest.TestCase):

    def setUp(self):
        self.db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
        self.app = flaskr.app.test_client()
        flaskr.init_db()

    def tearDown(self):
        os.close(self.db_fd)
        os.unlink(flaskr.app.config['DATABASE'])

    def test_empty_db(self):
        rv = self.app.get('/')
        assert 'No entries here so far' in rv.data
```
**注意:**测试的函数都是以`test`开始的，这样`unittest`就会自动识别这些用于测试的函数并运行它们。通过使用`self.app.get()`可以给制定的URL发送**HTTP GET**请求，其返回的是一个`～flask.Flask.reponse_class`对象，通过检查其data属性来检测其返回值

