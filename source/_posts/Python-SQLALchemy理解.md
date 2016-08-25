title: Python SQLALchemy理解
date: 2016-08-20 12:19:18
tags: [Python,Flask,Web,SQL]
categories: Flask
---
要不是为了用Flask开发系统，要不是涉及到各种表，估计也没人愿意用`ORM`框架吧，简单的查询估计直接写sql,用`MySQLdb`了。但是当你的系统大了，功能复杂了，表多了之后，这个东西还是很有必要的。如果你做了一个还算不太复杂的系统，里面各种表，你还手写sql,那我敬你是一条汉子，毕竟我们都喜欢踏实肯干的小伙子啊，请务必把简历发到我们公司HR的邮箱。好了，前面都是扯淡的，现在我要一本正经的说重点了，Flask的ORM框架也不止一个，为啥多说人都推荐用`SQLALChemy`这个ORM框架呢？其实原因很简单:因为它非常的灵活，这直接导致了它的功能非常的强大，当然，间接导致了这个框架学习成本很高。其实主要是参数很多，很多人不知道什么场景用什么参数，所以才有这篇笔记，我也不知道这些参数都是干啥的，但是作者既然留了这么多参数，必然是有场景需要，不然人家吃饱了撑得？


### SQLALChemy用法
虽然是因为Flask而去用这个框架，但是我这里主要讲`SQLALChemy`怎么用，并没有涉及到`Flask`.前面说了一堆，无非就是想告诉你，这个框架真的很牛逼，用的人很多，出了问题你也不是第一个踩坑的，毕竟每个坑下面躺着不少你的前辈，有了问题也好解决。直接说有些空洞，先上个简单的代码:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from sqlalchemy import Integer, ForeignKey, String, Column
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///:memory:', echo=False)
Base = declarative_base()


class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)

    def __init__(self, name=None):
        self.name = name

    def __repr__(self):
        return '<User:id={0},name={1}>'.format(self.id, self.name)

    addresses = relationship('Address')


class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))

    def __init__(self, email=None, user_id=None):
        self.email = email
        self.user_id = user_id

    def __repr__(self):
        return '<Address:id={0},email={1},user_id={2}>'.format(self.id, self.email, self.user_id)


if __name__ == '__main__':

    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()

    session.add_all([
        User(name='san.zhang'),
        User(name='si.li'),
        User(name='wu.wang')
    ])

    session.add_all([
        Address(email='123@qq.com', user_id=1),
        Address(email='456@qq.com', user_id=2),
        Address(email='789@qq.com', user_id=2)
    ])
    session.commit()

    print '查询1:%s' % session.query(User).all()
    print '查询2:%s' % session.query(User).filter_by(id=1).first().addresses
    print '查询3:%s' % session.query(User).filter_by(id=2).first().addresses
    print '查询4:%s' % session.query(User).filter_by(id=3).first().addresses
```
解释一下:
* 第10行我直接创建了一个内存数据库，这东西用法就是一个数据库链接，用过jdbc的都应该知道我在说什么，不知道的就当你知道了。具体的链接不同数据库不同，我就举个MySQL的例子:

```
SQLALCHEMY_DATABASE_URI = 'mysql://<user_name>:<passward>@<host>:<port>/<database>'
```

* `echo`:这个参数就是是否打印出执行的sql,其实orm框架到最后也是执行sql(好像是废话哈),所以你如果平时调试的时候想看这个底层执行的sql是不是符合你的逻辑，你可以把这个参数设置成`True`,这样在控制台就会打印出你每次执行的代码对应的sql

* 实体类
这里创建了两个实体类`User`,`Address`,分别对应数据库里面的两个表，这就完成了简单的`ORM`关系映射，把实体类和表对应起来了,注意这里有个映射关系，一个用户可能对应多个地址,这里还有个很重要的东西`relationship('Address')`.这个东西很复杂，其实`SQLALChemy`的复杂主要就是体现在这个地方，关系映射这个东西不太好理解，后面会细讲

* 数据库操作
这个其实比较简单了，需要一个操作句柄，`session`,修改操作只有在`session.commit()`之后才会真正提交到数据库中，类似与Spring里面的事务管理.

具体使用上就比较简单了，先创建了几个对象，然后调用`add_all()`把数据写到数据库里面，后面就可以查询了

### 关系映射relationship
前面也说了，这个东西复杂主要就是复杂在`relationship`这个类的使用上，总的来说，关系型数据库当然少不了关系这个桥梁，而ORM框架里面的这个R就显得很重要了，网上的一些教程吧，说的很含糊，我自己学的时候看了很多，那些人以为自己理解了，讲清楚了，但是说出来的话模棱两可，对于新手来说具有很大的迷惑性，我只能自己验证一下:

#### 表关联
首先强调一点，在实体类里面用`relationship()`申明的属性并不是这个类对应的数据库里面的真实字段，也就是说上面的`User`类里面有`addresses`这个属性，但是数据库表`user`里面并没有`addresses`字段，这个只是ORM框架替你做了一个映射，你可以通过`User`里面的属性拿到和它关联的地址，但是你没办法通过`Address`类来获取它对应的用户信息，这就是关系映射,看看运行结果:
```
查询1:[<User:id=1,name=san.zhang>, <User:id=2,name=si.li>, <User:id=3,name=wu.wang>]
查询2:[<Address:id=1,email=123@qq.com,user_id=1>]
查询3:[<Address:id=2,email=456@qq.com,user_id=2>, <Address:id=3,email=789@qq.com,user_id=2>]
查询4:[]
```
我这里把输出sql的开关设置成`Flase`了，所以比较整洁，只有输出结果，如果打开的话，就会有对应的查询语句.

#### 延迟加载
这个东西比较有意思,我们把`User`类里面的东西该成下面这样:
```
addresses = relationship('Address', lazy='dynamic')
```
我们不仅加了关系映射，还加了一个属性`lazy='dynamic'`,这个是个啥意思呢？其实就是延迟加载，想象一个场景，如果一个人对应了1w条地址信息，那么每当我们查一个用户的地址，访问`addresses`字段时，就会返回那1w条记录，但是正常情况下这1w条记录里面并不都是你要的，如果加上了上面这个属性，意思就是你访问`addresses`属性不再返回数据了，而是返回一个查询对象，你可以在这个查询对象上应用过滤啊，条件啥的，找到你真正要的那部分数据，这样性能上也会更好，不用每次把所有的数据都返回，具体可以看下面的例子,上面的查询语句输出结果如下:
```
查询1:[<User:id=1,name=san.zhang>, <User:id=2,name=si.li>, <User:id=3,name=wu.wang>]
查询2:SELECT address.id AS address_id, address.email AS address_email, address.user_id AS address_user_id 
FROM address 
WHERE :param_1 = address.user_id
查询3:SELECT address.id AS address_id, address.email AS address_email, address.user_id AS address_user_id 
FROM address 
WHERE :param_1 = address.user_id
查询4:SELECT address.id AS address_id, address.email AS address_email, address.user_id AS address_user_id 
FROM address 
WHERE :param_1 = address.user_id AND address.email = :email_1
```
可以看到，并没有输出我们查询到的结果，而是返回了查处对象，那么如果要获取查询结果怎么办呢？其实也很简单，既然是查询对象，那像上面一样调用`first(),all()`就可以了，把查询语句改一下:
```
    print '查询1:%s' % session.query(User).all()
    print '查询2:%s' % session.query(User).filter_by(id=1).first().addresses.all()
    print '查询3:%s' % session.query(User).filter_by(id=2).first().addresses.all()
    print '查询4:%s' % session.query(User).filter_by(id=2).first().addresses.filter_by(email='456@qq.com').all()
```
输出结果为:
```
查询1:[<User:id=1,name=san.zhang>, <User:id=2,name=si.li>, <User:id=3,name=wu.wang>]
查询2:[<Address:id=1,email=123@qq.com,user_id=1>]
查询3:[<Address:id=2,email=456@qq.com,user_id=2>, <Address:id=3,email=789@qq.com,user_id=2>]
查询4:[<Address:id=2,email=456@qq.com,user_id=2>]
```
注意第四个查询语句，我们在获取`addresses`属性之后调用了过滤函数，最后的结果确实只返回了一个，所以说这个延迟加载的确很方便，可以在查询结果返回前过滤掉不用的数据

#### 反向查询
我们知道，通过`User`类可以访问到`addresses`属性，其实返回的就是`Address`对象，那么问题来了，如果想通过`Address`来反查某个地址下面的用户怎么办，当然你可以查询到这些`user_id`再去`user`表里面查对应的`user`信息，但是这样很麻烦，`sqlalchemy`也提供了一种反查机制,`backref`参数,类似于在`Address`中添加了`user`属性，同样的把`User`实体类改一下:
```
addresses = relationship('Address', backref=db.backref('user', lazy='joined'), lazy='dynamic')
```
注意到最外层的`lazy='dynamic'`是针对`User`查询多个地址时延迟加载策略,`db.backref('user', lazy='joined')`这个虽然定义在了`User`类里面，但是此处的作用是定义一个反向引用关系，这样就可通过`Address`来访问到对应的`User`类了，并且里面也定了一个策略`lazy='joined'`，这是告诉`Address`查找对应的`User`时采用连接查询，如果不加，那就会采用子查询的方式来查询。可以测试一下，需要打开`echo=True`设置,这里就查一个吧，因为要输出查询语句，所以输出信息有点儿多,假设查询语句为:
```
print '查询1:%s' % db.session.query(Address).filter_by(id=2).first().user
```
看看相信的输出结果:
```
2016-08-20 15:45:29,194 INFO sqlalchemy.engine.base.Engine SELECT address.id AS address_id, address.email AS address_email, address.user_id AS address_user_id, user_1.id AS user_1_id, user_1.name AS user_1_name 
FROM address LEFT OUTER JOIN user AS user_1 ON user_1.id = address.user_id 
WHERE address.id = ?
 LIMIT ? OFFSET ?
2016-08-20 15:45:29,194 INFO sqlalchemy.engine.base.Engine (2, 1, 0)
查询1:<User:id=2,name=si.li>
```
看到没有，用的是`LEFT OUTER JOIN`来查询的，那如果去掉`lazy='joined'`呢？看看输出结果:
```
2016-08-20 15:47:33,246 INFO sqlalchemy.engine.base.Engine SELECT address.id AS address_id, address.email AS address_email, address.user_id AS address_user_id 
FROM address 
WHERE address.id = ?
 LIMIT ? OFFSET ?
2016-08-20 15:47:33,246 INFO sqlalchemy.engine.base.Engine (2, 1, 0)
2016-08-20 15:47:33,247 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.name AS user_name 
FROM user 
WHERE user.id = ?
2016-08-20 15:47:33,247 INFO sqlalchemy.engine.base.Engine (2,)
查询1:<User:id=2,name=si.li>
```
看到没有，这个里面有两个sql,先查`address`表，可以获取到`user_id`字段，然后再用这些去`user`表里面查,所以这里的`lazy='joined'`相当于反向查询的惰性加载。
还有个奇特的用法，就是自引用，就是类似于无限目录这种机制:

##### 多对对映射
很多时候用ORM框架就是因为业务结构太复杂，复杂性一般体现在表与表之间很多都是多对多映射，如果两个对象是多对多的话怎么映射这个关系呢？难道是在两个实体类里面都用`relationship`互相声明吗？当然不用，`SQLALChecmy`里面提供了一个更简单的方法,中间表关联法,还是以`User`和`Address`来说明:
新来增加一张中间表
```
user_address = db.Table(
    'user_address',
    db.Column('User', db.Integer, db.ForeignKey('user.id')),
    db.Column('Address', db.Integer, db.ForeignKey('address.id'))
)
```
这个就不用实体类了，直接用中间表了,然后修改`User`表的定义:
```
    addresses = relationship('Address', secondary=user_address, backref=db.backref('user', lazy='joined'),
                             lazy='dynamic')
```
然后修改`Address`实体类定义:
```
class Address(db.Model):
    __tablename__ = 'address'
    id = Column(Integer, primary_key=True)
    email = Column(String)

    def __init__(self, email=None):
        self.email = email

    def __repr__(self):
        return '<Address:id={0},email={1}>'.format(self.id, self.email)
```
去掉了之前在这里面定义的外键`user_id`，然后我们来测试一下:
```
if __name__ == '__main__':
    db.create_all()

    add_list = []
    for item in ['123@qq.com', '456@qq.com', '789@qq.com']:
        address = Address()
        address.email = item

        add_list.append(address)
        db.session.add(address)

    user = User()
    user.addresses = add_list[:2]
    db.session.add(user)

    user = User()
    user.addresses = add_list[1:]

    db.session.commit()

    print '查询1:%s' % User.query.first().addresses.all()
    print '查询2:%s' % Address.query.filter_by(email='456@qq.com').first().user
```
创建了3个地址，每个用户拥有2个地址，并且这两个用户都有中间的地址，看看查询结果:
```
查询1:[<Address:id=1,email=123@qq.com>, <Address:id=2,email=456@qq.com>]
查询2:[<User:id=1,name=None>, <User:id=2,name=None>]
```
可以看到反查，正查都可以差多多对多的结果。
