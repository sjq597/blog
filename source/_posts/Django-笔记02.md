title: Django 笔记02
date: 2016-01-03 11:29:44
tags: [Python,Web]
categories: Django
---
上一篇文章简单的搭建了一个`Django`的应用,互联网的应用，最后的数据都必须存储在数据库里，所以一个Web应用不可能不用到数据库，今天简单介绍一下`Django`中如何使用和配置数据库。

### 数据库设置
打开配置文件`mysite/settings.py`,默认这个配置文件是使用的`SQLite`数据库，如果只是简单的学些这个框架，做一些简单的应用，这个数据库很方便，是Python内置的，你不用再安装任何其他的驱动，包之类的。当然，如果你想使用更牛逼的数据库，例如`MySQL`，`PostgreSQL`，看配置文件的下面的内容：
```python
# Database
# https://docs.djangoproject.com/en/dev/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```
**ENGINE**:
```
'django.db.backends.sqlite3',
'django.db.backends.postgresql',
'django.db.backends.mysql',
'django.db.backends.oracle',
```
**NAME**:你所创建的数据库文件的地址，记得是绝对路径，默认的是`os.path.join(BASE_DIR,'db.sqlite3')`。
如果使用的是其他的数据库，还需要数据库的用户名和密码，记得一定要安装对应的驱动，例如，使用`MySQL`要安装类似于`python-mysqldb`这样的驱动，配置文件如下：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'test',    	#你的数据库名称
        'USER': 'root',   	#你的数据库用户名
        'PASSWORD': '', 	#你的数据库密码
        'HOST': '', 		#你的数据库主机，留空默认为localhost
        'PORT': '3306', 	#你的数据库端口
    }
}
```
由于只是练手，简单的就是用默认配置了，在使用数据库之前，需要创建表，在终端里执行以下命令：
```bash
python manage.py migrate
```
这个命令其实是会根据`mysite/settings.py`里安装的模块创建对应的表，如果某些模块需要使用数据库，它就会创建对应的表，这么看实在是强大又方便。
```python
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
看看执行命令的输出确实是这样的,有四个模块使用了数据库：
```
Operations to perform:
  Apply all migrations: admin, contenttypes, auth, sessions
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
```

### 实体类设计
在我们之前创建的一个应用里，我们来创建两个实体类：`Question`和`Choice`,每个`Choice`都关联着一个`Question`。编辑`polls/models.py`文件：
```python
from __future__ import unicode_literals

from django.db import models


# Create your models here.

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    quetion = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
**NOTE:**每个实体类有两个字段属性，注意`Choice`里的`question`不是字段，只是定义一个外键。

### 激活模块
现在我们已经创建了模块，剩下的我们要告诉我们的工程，`polls`这个应用要被安装，就是上面我们提到的配置文件，修改`mysite/settings.py`，改为以下内容：
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
然后在终端里运行下面的命令：
```bash
python manage.py makemigrations polls
```
将会看到如下输出：
```
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field quetion to choice
```
运行上面的命令的意思就是说，你已经修改了你的实体类，然后想把这些改动存储到数据库里。关于这个命令目前不需要知道太详细，知道这样用就可以了，但这样还不够，还需要运行下面的命令来把这些操作生效：
```bash
python manage.py migrate
```
看到如下输出就说明成功了：
```
Operations to perform:
  Apply all migrations: admin, contenttypes, polls, auth, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```
上面的命令是把改变写到数据库中，使之生效，如果不运行这个，会报错，找不到表。

### 使用API交互调试
上回忘了说，`manage.py`有很多命令，其中有一个是交互式调试工具`manage.py shell`:
```python
In [1]: import django

In [2]: django.setup()

In [3]: from polls.models import Question,Choice

In [4]: Question.objects.all()
Out[4]: <QuerySet []>

# 下面我们来插入一条记录，稍微多说一句，使用`timezone.now()`来替代
# datetime.datetime.now()
In [5]: from django.utils import timezone

In [6]: q = Question(question_text="What's new?", pub_date=timezone.now())
# 把这条记录存到数据库里
In [7]: q.save()

# 存到数据库之后这条记录就有id了
In [8]: q.id
Out[8]: 1

In [9]: q.question_text
Out[9]: "What's new?"

In [10]: q.pub_date
Out[10]: datetime.datetime(2016, 1, 3, 10, 4, 54, 39966, tzinfo=<UTC>)

In [11]: Question.objects.all()
Out[11]: <QuerySet [<Question: Question object>]>
```
看最后一行，是不是觉得根本看不出什么东西？类比于`Java`里，我们知道，有些类都会自带一个`toString()`方法，可以把一个类作为一个字符串输出来，这里我们也来定义一下类的`toString`方法。
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text


class Choice(models.Model):
    quetion = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```
然后你需要退出交互式Python环境，重新进一边，就可以看到变化了：
```python
In [4]: Question.objects.all()
Out[4]: <QuerySet [<Question: What's new?>]>
```
再给`Question`加一个比较常用的方法:
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text
    
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

### 使用admin模块
这个模块主要是管理员模块，管理网站的各种权限
```bash
python manage.py createsuperuser
```
后面照着填就行了。开启服务器：`python manage.py runserver`,访问`http://127.0.0.1:8000/admin/`。
登陆进去之后界面大概是这样的：
![Django 后台管理模块](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Django%20%E7%AC%94%E8%AE%B00201.png)

### 导入模块到后台管理
编辑`polls/admin.py`文件，内容如下：
```python
from django.contrib import admin

# Register your models here.
from models import Question

admin.site.register(Question)
```
把模块注册到后台管理模块中之后，就可以在管理模块中操作我们的数据了，再次刷新就可以看到我们的模块了：
![polls模块](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Django%20%E7%AC%94%E8%AE%B00202.png)
还可以添加管理数据：
![后台管理数据库数据](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Django%20%E7%AC%94%E8%AE%B00203.png)
这些数据会根据自己的类型选择自己的展示方式，日期的会有一个日历展示框。
