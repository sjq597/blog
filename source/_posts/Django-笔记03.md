title: Django 笔记03
date: 2016-01-03 18:59:05
tags: [Python,Web]
categories: Django
---
上一篇文章讲了数据库如何存储在数据库中，以及模块如何显示在网页中，利用管理模块添加管理模块。现在我们来做一个简单的投票网站，大概有以下几个页面：
### 整体框架
1. `index`Page:展示最近最新的问题
2. `detail`Page:展示一个问题的具体描述
3. `results`Page:展示具体的结果
4. `vote`Page:具体投票的界面

### 增加几个视图
编辑`polls.views.py`文件，内容如下：
```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the result of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
增加了视图，还要把视图的`url`写到`polls.urls`中：
```python
urlpatterns = [
    url(r'^$', views.index, name='index'),
    # ex: /polls/5/
    url(r'(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    # ex: /polls/5/results/
    url(r'(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    # ex:/polls/5/vote/
    url(r'(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```
**Tips**分别访问[/pools/3/](http://127.0.0.1:8000/polls/3/),[polls/3/results/](http://127.0.0.1:8000/polls/3/results/),[polls/3/vote/](http://127.0.0.1:8000/polls/3/vote/)会分别调用这几个模块视图,返回对应的结果.
稍微解释一下这个访问过程:当有个人访问[127.0.0.1:8000/polls/3/](http://127.0.0.1:8000/polls/3/)这个链接的时候,Django会加载`mysite.urls`这个模块,因为这个文件是项目的根配置链接模块,然后在这个模块里找到了对应的匹配项`^polls/`,然后把`3/`根据`include()`函数找到对应的模块的`urls`配置文件`polls.urls`,最终调用对应的视图`r'^P<question_id>[0-9]+)/$`.调用视图的参数传递像下面这样:
```python
detail(request=<HttpRequest object>, question_id='3')
```
这里的`question_id='34'`是从正则表达式`(?P<question_id>[0-9]+)`中来的,这个就是正则表达式里的一种用法,用括号括起来可以捕获正则匹配到的值,并且把他们作为参数传递给视图函数.其中`?P<question_id>`定义了这个匹配到的值的使用名,`[0-9]+`意思是匹配任意数字1次或者多次.

### 视图
上面的是一个简单的示例,其实这个视图根本没做什么事情,一般一个视图会做两件事:返回一个`HttpResponse`包含请求页面的内容,或者返回一个异常,例如`Http404`,这个取决于你.
为了让视图有内容,我们从数据库里读数据,将这些数据展示在网页上,例如显示最近的5条记录,修改`polls/views.py`
```python
# Create your views here.
from models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.quetion_text for q in latest_question_list])
    return HttpResponse(output)
```
这里又有一个问题了,一般的网页设计都是写死的,如果你想改变网页的展示方式,你就要取改对应的Python代码,所以Django采用了设计和数据分离的模板模式,下面讲一讲如何使用模板来展示数据.

### 网页模板
首先在`polls`目录下创建一个`templates`目录,Django会到这个目录里找模板.在`mysite/settings.py`文件里,有如下内容:
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
看到`'APP_DIRS': True`,这个会在我们刚创建的`templates`目录下扫描你已经安装的`APP`的目录里的模板.以我们这个例子为例,我们创建的模板的相对路径为:`polls/templates/polls/index.html`.当你使用的时候,直接就是`polls/index.html`.不要觉得这个麻烦,为什么要多创建一个`polls`的子目录而不是把模板文件直接放在`templates`下,这个就涉及到一个模板的查找个加载,Django默认从`templates`下查找,如果找到了就匹配第一个,所以不要省略子目录`polls`,你可以把这个看成是Java里的包的路径或者C++里的命名空间.
编辑`polls/templates/pools/index.html`
```html
{% if latest_question_list %}
    <ul>
        {% for question in latest_question_list %}
            <li><a href="/polls/{{ question.id}}">
                {{ question.question_text}}
                </a>
            </li>
        {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
然后更新视图文件`polls/views.py`:
```python
# Create your views here.
from django.template import loader
from models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
这段代码加载了一个模板`polls/index.html`,并且给模板传递了一个上下文,这是个字典.启动服务器,访问对应的网址`127.0.0.1:8000/polls/`默认会调用`index()`视图函数,返回一个列表,就是数据库里查询出来的几条记录.
`Django`对于这种常用的模板操作,也提供了一个简便的用法,我们来重写一下`index()`方法:
```python
# Create your views here.
from django.shortcuts import render
from models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {
        'latest_question_list': latest_question_list,
    }
    return render(request, 'polls/index.html', context)
```
**render()**函数接收三个参数,第一个是一个`request`对象,第二个是一个模板,第三个是一个可选参数,字典内容.它返回一个渲染过的模板过后的`HttpResponse`对象以及字典内容

### 返回404错误
现在我们来处理一下详情`detail()`函数,编辑`polls/views.py`:
```python
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoseNotExists:
        raise Http404("Question does not exist")

    return render(request, 'polls/detail.html', {'question': question})
```
然后在`polls/detail.html`中添加简单的内容
```html
<p>Hello</p>
{{ question }}
```
同理,调用对应的方法,如果数据有,则会调用模板,没有则会跑出异常.

### get_object_or_404()
由于我们经常使用这个用法,即有就获取内容,没有就抛出异常,所以Django也有一个简单的用法,重写`detail()`方法:
```python
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)

    return render(request, 'polls/detail.html', {'question': question})
```
这个会自动捕获`object doesn't exist`这个异常,还有个类似的方法`get_list_or_404()`,在列表是空的时候会抛异常.

### 使用模板系统
编辑模板文件夹下的`polls/detail.html`文件:
```html
<h1>{{ question.question_text }}</h1>
<ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }}</li>
    {% endfor %}
</ul>
```
移除模板里的硬编码,让模板可以动态的显示内容,编辑`polls/index.html`:
把如下的内容:
```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
改为:
```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
先说一下为什么要这么改,上面一种写法为什么不好,其实可以想象,如果连接都写死,一旦我们的模板多起来了,改链接会很麻烦.其实有个简单的方法,在前面`urls.py`里,我们在写正则匹配url的时候,还定义了模板的名字,所以你可以使用`{% raw %}{% url %}{% endraw %}`这个模板标签,这个是根据下面的定义来的:
```python
...
# the 'name' value as called by the {% url %} template tag
url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
...
```
这样当你改了外部访问链接,例如`127.0.0.1:8000/polls/specifics/12/`,你只用在`polls/url.py`里改对应的正则规则就行,模板里的代码不用改.

### URL命名空间
虽然上面的方法可以让我们少改代码,但是试想一下,一个真正的工程不可能只有一个`APP`模块,Django怎么知道在找到多个的时候用那个,答案是命名空间,是的,就是C++里的那个命名空间,我们来修改`polls/urls.py`文件:
```python
app_name = 'polls'
urlpatterns = [
	......
    ]
```
然后修改`polls/index.html`文件,将
```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
修改为:
```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```
如果这样直接运行,然后访问`127.0.0.1:8000/polls/`会报错:
```
u'polls' is not a registered namespace
```
解决办法是在project的`urls.py`中`include()`的时候加上`namespace`属性,编辑`mysite/urls.py`文件:
```python
urlpatterns = [
    url(r'^polls/', include('polls.urls', namespace='polls')),
    url(r'^admin/', admin.site.urls),
]
```
然后就可以访问了.
