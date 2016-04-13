title: Django 笔记04
date: 2016-01-07 14:15:22
tags: [Python,Web]
categories: Django
---
通过第三篇笔记我们也基本知道怎么处理Web的视图和数据库及链接和模板,这次主要讲一讲表单,因为Web免不了要提交数据给服务器,表单作为一种非常常见和基本的提交数据的方式,在Web开发中是很重要的一种方法.

### 一个简单的表单
更新我们的模板文件,编辑`polls/detail.html`,改成如下内容:
```html
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p><% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
    {% csrf_token %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{forloop.counter}}" value="{{ choice.id}}"/>
        <label for="choice{{forloop.counter}}">{{ choice.choice_text }}</label><br/>
    <% endfor %}

    <input type="submit" value="Vote" />
</form>
```
* 上面的是一个简单的单选按钮的模板，每一个投票选项是一个单选按钮，然后这个单选按钮的值就是关联的问题选项的`id`。每个单选按钮的名字都一样，是`choice`，也就是如果你选了某个选项，那么POST的数据就是`choice=#id`。
```python
url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
```

* **forloop.counter**暗示了有多少个选项，其实就是外层循环次数。
* `{%raw%} {% csrf_token %} {% endraw %}`是个防止跨域攻击的标签，只需要知道这么用就行了。

这回我们可以来完善一下我们的`vote`视图，让它有一些功能，编辑`polls/views.py`:
```python
from django.core.urlresolvers import reverse
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import render, get_object_or_404
from models import Question, Choice
# 省略其他视图

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)

    try:
        select_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # 重新展示问题的投票按钮
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice."
        })
    else:
        select_choice.votes += 1
        select_choice.save()

        # 投票成功之后要重定向到一个结果网页
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))

```
有一些代码需要解释一下，好多前面并没有介绍过：
* **request.POST**就像一个字典一样，你可以通过key来访问表单提交的数据，`request.POST['choice']`返回的是选项的`id`，不过返回的是个字符串形式，因为它返回的始终是字符串形式。
* **request.POST['choice']**会抛出一个`KeyError`当这个值没有提供的时候，这里我们的处理是没提供就返回到提交投票的表单并且给出一个信息提示。
* 当投票成功之后，代码中返回了一个`HttpResponseRedirect`而不是普通的`HttpResponse`,这个函数还可以带参数，可以重定向到我们想访问的网址,建议当我们成功处理的一个POST请求的时候都可以这么做。
```
/polls/3/results/
```
这个3就是参数中带的`question.id`的值。
当某个人对这个问题投票的时候，`vote()`视图将会重定向到结果页面，还需要增加一个`results()`视图，编辑`polls/views.py`：
```python
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
这个和之前的`detail()`视图非常相似，用到了`results.html`模板，新建`template/polls/results.html`文件:
```html
<h1>{{ question.question_text }}</h1>

<ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }} -- {{ choice.votes }} vote {{ choice.votes | pluralize }}</li>
    <% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
此时如果你直接运行，访问`polls/1/`这个链接里面是空的，因为你的数据库里没有`Choice`,你可以直接在命令行中运行插入，或者一个简单的，直接按照前面讲的，把`choice`模块注册到`admin`模块中管理，就可以在网页上直接操作了，加入注册模块也很简单，修改`polls/admin.py`文件，内容如下：
```python
from django.contrib import admin

# Register your models here.
from models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```
然后就去管理员后台直接添加几个选项和问题吧。添加完毕然后再访问，就会出现一个问题和对应的投票选项，就像我们做选择题一样，一个题有几个选项。假设我们什么都不选，然后点提交。会得到一个错误信息，就显示在选项上面**You didn't select a choice.**
