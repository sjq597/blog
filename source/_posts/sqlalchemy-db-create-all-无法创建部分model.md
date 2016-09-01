title: sqlalchemy db.create_all 无法创建部分model
date: 2016-09-01 20:34:57
tags: [Flask,SQLALChemy,Python]
categories: Flask
---
今天在做Flask模块的实体类ORM声明时，发现会报错,花了一上午查了好多资料才解决，记录一下解决方案，查资料的时候貌似碰到这个问题的人还挺多的.
报错信息为:
> 
InvalidRequestError: When initializing mapper Mapper|Parent|parent, expression 'childs' failed to locate a name ("name 'Child' is not defined"). If this is a class name, consider adding this relationship() to the Parent class after both dependent classes have been defined.

什么意思呢？就是在在`Parent`与`Child`一对多的映射时，映射找不到这个类，这两个类属于不同的注册模块，当然也就不在一个文件中了，结构如下:
```
➜  web 
├── parent
│   ├── __init__.py
│   ├── models.py
│   ├── templates
│   └── views.py
└── child
    ├── __init__.py
    ├── models.py
    ├── templates
    └── views.py
```
部分关键代码如下:
```
class Parent(db.Model):

    __tablename__ = 'parent'
    __bind_key__ = 'test'

    id = db.Column(db.Integer, primary_key=True)
    childs = db.relationship('Child', lazy='dynamic')

class Child(db.Model):

    __tablename__ = 'child'
    __bind_key__ = 'test'

    id = db.Column(db.Integer, primary_key=True)
    parent_id = db.Column(db.Integer, db.ForeignKey('parent.id'))

```
这是一个很简单的一对多映射，其他的类都可以调用`db.create_all()`创建，但是我调用这个命令确并不会创建`Child`表,实在想不明白这个类特殊在哪，于是网上查阅说可以强制导入model的定义:
```
from web.child.models import *
db.create_all()
db.session.commit()
```
然后去mysql里面看，确实可以创建表了，于是启动项目，结果登陆系统还是报一样的错，仍然是映射关系出问题，于是把谷歌翻了个变，终于找到了一个解决方法.
通常来说,`db.create_all()`无法创建所有表会有以下几个原因:
> 
1. model未继承`db.Model`
2. views中未引入该表的model
3. create_all()与models定义不在一个文件

对于一个多模块的系统来说，第3个原因不太可能，因为几乎不可能把models定义和`create_all()`放到一个文件里面。但是其他模块都没啥问题，所以问题不会出在这。
第一个显然不是，我的所有实体类都是继承了`db.Model`,所以当我把问题定位在2时，仔细一想好像真是这样，因为我虽然新增了一个模块，但是只是先把模块的`models`定义好了，`views.py`里面其实啥也没写，自然没有引用到新定义的`models`。
于是在`views.py`文件里面加了一句`web/child/views.py`:
```
from web.child.models import *
```
然后执行:
```
db.drop_all()
db.create_all()
db.session.commit()
```
然后登陆系统，果然没有出现映射问题了。
