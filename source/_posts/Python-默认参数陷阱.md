title: Python 默认参数陷阱
date: 2015-10-10 18:08:53
tags: [Python]
categories: [Python笔记]
---
Python的函数定义提供了默认参数这个选择，使得函数的定义和使用更加的灵活，但是也会带来一些坑，例如之前的一个例子：
函数定义：
```python
def add_end(L=[]):
    L.append('END')
    return L
```
调用函数的结果：
```bash
>>> add_end([1, 2, 3])
[1, 2, 3, 'END']
>>> add_end(['x', 'y', 'z'])
['x', 'y', 'z', 'END']
>>> add_end()
['END']
>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
```
很明显这个与函数的定义初衷不符，用一句话解释就是：
>Default values are computed once, then re-used.

为了深入研究这个问题，我们来看看另一个例子：
```python
# coding=utf-8

def a():
    print "a executed"
    return []

def b(x=a()):
    print "id(x):", id(x)
    x.append(5)
    print "x:", x

for i in range(2):
    print "不带参数调用，使用默认参数"
    b()
    print b.__defaults__
    print "id(b.__defaults__[0]):", id(b.__defaults__[0])

for i in range(2):
    print "带参数调用，传入一个list"
    b(list())
    print b.__defaults__
    print "id(b.__defaults__[0]):", id(b.__defaults__[0])
```
**NOTE:**稍微解释一下，所有默认值都存储在函数对象的`__defaults__`属性中，这是一个列表，每一个元素均为一个默认参数值。
来看看输出结果：
```bash
a executed
不带参数调用，使用默认参数
id(x): 140038854650552
x: [5]
([5],)
id(b.__defaults__[0]): 140038854650552
不带参数调用，使用默认参数
id(x): 140038854650552
x: [5, 5]
([5, 5],)
id(b.__defaults__[0]): 140038854650552
带参数调用，传入一个list
id(x): 140038854732400
x: [5]
([5, 5],)
id(b.__defaults__[0]): 140038854650552
带参数调用，传入一个list
id(x): 140038854732472
x: [5]
([5, 5],)
id(b.__defaults__[0]): 140038854650552
```
简单分析一下输出结果：
* 第1行
在定义函数`b()`，即执行`def`语句，代码第7行`def b(x=a()):`的时候，这句话使用了默认参数，所以在定义的时候会计算默认参数`x`的值，这个时候会调用`a()`，所以打印出了`a executed`。

* 第2~6行
第一次执行循环，代码第14行调用`b()`没有传递参数，使用默认参数，此时`x=[]`，所以调用一次之后
```python
print b.__defaults__
```
 输出结果为
```bash
([5],)
```

* 第7~11行
第二次循环，代码第14行调用`b()`没有传递参数，使用默认参数。
**注意：**`默认参数只会计算一次，也就是说那个内存区域就固定了，但是这个地址所指向的是一个list，内容可以改变`，此时由于上一次调用`x: [5]`，所以
```python
print b.__defaults__
```
 输出结果为
```bash
([5, 5],)
```

* 第12~16行
第二个循环语句，第一次循环，代码第20行传入一个空的`list`,所以不使用默认参数，此时`x=[]`，所以
```python
print b.__defaults__
```
 输出结果为
```bash
([5],)
```

* 第18~21行
第二个循环语句，第二次循环，代码第20行传入一个空的`list`,所以也不使用默认参数，此时仍然是`x=[]`，所以
```python
print b.__defaults__
```
 输出结果依然为
```bash
([5],)
```

函数也是对象，因此定义的时候就被执行，默认参数是函数的属性，它的值可能会随着函数被调用而改变。其他对象不都是如此吗？
**牢记:** 默认参数必须指向不变对象！代码改一下如下：
```python
# coding=utf-8

def a():
    print "a executed"
    return None

def b(x=a()):
    print "id(x):", id(x)
    if x is None:
        x = []
    x.append(5)
    print "x:", x

for i in range(2):
    print "不带参数调用，使用默认参数"
    b()
    print b.__defaults__
    print "id(b.__defaults__[0]):", id(b.__defaults__[0])

for i in range(2):
    print "带参数调用，传入一个list"
    b(list())
    print b.__defaults__
    print "id(b.__defaults__[0]):", id(b.__defaults__[0])
```
 此时的输出结果看看是什么：
```bash
a executed
不带参数调用，使用默认参数
id(x): 9568656
x: [5]
(None,)
id(b.__defaults__[0]): 9568656
不带参数调用，使用默认参数
id(x): 9568656
x: [5]
(None,)
id(b.__defaults__[0]): 9568656
带参数调用，传入一个list
id(x): 140725126699632
x: [5]
(None,)
id(b.__defaults__[0]): 9568656
带参数调用，传入一个list
id(x): 140725126699704
x: [5]
(None,)
id(b.__defaults__[0]): 9568656
```
