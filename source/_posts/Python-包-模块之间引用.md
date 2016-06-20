title: Python 包 模块之间引用
date: 2016-02-20 17:32:06
tags: [Python]
categories: Python笔记
---
Python在涉及到跨包的模块引用的时候，并不像Java这种语言那样，可以做到自动查找包路径。在网上也看到各种说法，有些还是不行，所以打算亲自实验一番。
### 准备工作
实验环境：
> 
OS：Ubuntu
Python: Python 2.7.10+
IDE:PyCharm

实验之前还要说一下，Python里面，必须在每个文件夹下面建一个`__init__.py`，这个文件是空的，但是必须要有这个文件，Python才会把这个文件夹视为一个`Python Package`，包里面的文件就可以称为模块。
首先看看测试工程的目录结构：
```
➜  package_ref  tree
.
├── module1
│   ├── __init__.py
│   ├── m11.py
│   ├── m12.py
│   └── module11
│       ├── __init__.py
│       └── m1_m11.py
├── module2
│   ├── __init__.py
│   └── m21.py
├── module3
│   ├── __init__.py
│   └── m31.py
└── m.py
```
具体的引用关系为：  
1. m21.py引用了m11,m12中的函数  
2. m31引用了m1_11中的函数  
3. m.py引用了m11,m12中的函数

### 文件内容
* m11.py
```python
#!/usr/bin/env python
# coding=utf-8


def func1():
    print 'm11 func1'


def func2():
    print 'm11 func2'
```
* m12.py
```python
#!/usr/bin/env python
# coding=utf-8


def func1():
    print 'm12 func1'


def func2():
    print 'm12 func2'
```
* m1_m11.py
```python
#!/usr/bin/env python
# coding=utf-8


def func1():
    print 'm1_m11 func1'


def func2():
    print 'm1_m11 func2'
```
* m21.py
```python
#!/usr/bin/env python
# coding=utf-8
from module1 import m11, m12


def func1():
    m11.func1()
    print 'm21 func1'


def func2():
    m12.func2()
    print 'm21 func2'


if __name__ == '__main__':
    func1()
    func2()
```
* m31.py
```python
#!/usr/bin/env python
# coding=utf-8
from module1.module11 import m1_m11


def func1():
    m1_m11.func1()
    print 'm31 func1'


def func2():
    m1_m11.func2()
    print 'm31 func2'


if __name__ == '__main__':
    func1()
    func2()
```
* m.py
```python
#!/usr/bin/env python
# coding=utf-8
from module1 import m11, m12


def func1():
    m11.func1()
    print 'm func1'


def func2():
    m12.func2()
    print 'm func2'

if __name__ == '__main__':
    func1()
    func2()
```
### 执行脚本
在PyCharm里执行，全部都可以正确执行,进入`package_ref`根目录，终端中执行`python m.py`,也没有问题。
进入`module2`目录，执行`python m21.py`，报错：
> 
Traceback (most recent call last):
  File "module2/m21.py", line 3, in <module>
    from module1 import m11, m12
ImportError: No module named module1

进入module3也是同样的错误。

### 解决方案
在报错的文件中，引入本地包里面的模块之前，头部加上：
```python
import sys
sys.path.append('..')
```
其他地方不用改变，测试之后在pyCharm和终端中执行都没有问题。

**备注:**详细的工程在github上也有，git地址为[python-package_ref](https://github.com/sjq597/python_package_ref)
