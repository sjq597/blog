title: Python 参数解析
date: 2016-06-29 23:21:52
tags: [Python]
categories: Python笔记
---
有时需要用到外部传入的参数，虽然简单的使用`*arg`数组可以获取到每个参数的值，但是有一个局限性就是参数必须按顺序传入，不能多也不能少。
但是考虑到有时候我们需要制定特定的参数的值，参数的个数，顺序都是不定的，这个时候单纯的靠`*args`显然无法满足我们的需求,这个时候就轮到`argparse`模块上场了，基本上一般的参数解析都可以胜任，下面看看怎么用：

### 参数的几种解析方式
主要有四种方式，简单介绍一下,先介绍一下最简单的按顺序解析不带`key`指定参数形式:

#### 不带key按顺序解析
这种方式可以获取脚本运行时后面的所有参数，但是顺序由输入的顺序决定的，
先看看测试代码,假设文件名叫`args.py`:
```python
#! /usr/bin/python
# coding=utf-8

import sys

if __name__ == '__main__':
    for arg in sys.argv:
        print arg
    pass
```
来一段测试代码:
```bash
$ python args.py 12 34 56 78                                                                                                
args.py
12
34
56
78
```
第一个参数就是python脚本的名字,后面四个就是四个参数了.但这个有时候我们希望我们传入参数的顺序可以是随意的，只要我们指定每个参数的`key`,程序就能解析到对应的参数值，那应该怎么做呢？下面介绍一下带`key`的参数解析。

#### 不带`-`参数

新建一个脚本`arg_demo.py`,然后输入以下代码:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import argparse

__author__ = 'anonymous'

if __name__ == '__main__':
    parse = argparse.ArgumentParser()
    parse.add_argument('a', help='必填参数')
    args = parse.parse_args()
    
    print args.a  
    pass
```
终端运行:
```
➜  parameter git:(python-note) ✗ python arg_demo.py                    
usage: arg_demo.py [-h] a
arg_demo.py: error: too few arguments
➜  parameter git:(python-note) ✗ python arg_demo.py 我是参数           
我是参数
➜  parameter git:(python-note) ✗ python arg_demo.py 我是参数1 我是参数2
usage: arg_demo.py [-h] a
arg_demo.py: error: unrecognized arguments: 我是参数2
➜  parameter git:(python-note) ✗ 
```
可以看出不带`-`的参数在调用脚本时必须输入值，并且输入的顺序必须和程序定义的一直，而且个数也得一致

#### 带`-`的参数

同样的文件，改动以下代码
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import argparse

__author__ = 'anonymous'

if __name__ == '__main__':
    parse = argparse.ArgumentParser()
    parse.add_argument('-a')
    args = parse.parse_args()

    print args.a
```
同样，我们在终端中输入不同值运行一下:
```
➜  parameter git:(python-note) ✗ python arg_demo.py                          
None
➜  parameter git:(python-note) ✗ python arg_demo.py 我是参数                 
usage: arg_demo.py [-h] [-a A]
arg_demo.py: error: unrecognized arguments: 我是参数
➜  parameter git:(python-note) ✗ python arg_demo.py -a 我是参数              
我是参数
➜  parameter git:(python-note) ✗ python arg_demo.py -a我是参数 
我是参数
➜  parameter git:(python-note) ✗ python arg_demo.py -a 我是参数1 -a 我是参数2
我是参数2
➜  parameter git:(python-note) ✗ python arg_demo.py -a 我是参数2 -a 我是参数1 
我是参数1
```
可以看出，带`-`的参数,可以输入也可以不输入，但是不能输入的时候不指定key,并且输入的key可以和参数分开或者连在一起，多次对一个key输入值，后面的会覆盖前面的输入。

#### 带`--`参数

`-`参数还可以指定`shorname`,即`--shortname`这种格式，表示变量的别名，改动一下代码:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import argparse

__author__ = 'anonymous'

if __name__ == '__main__':
    parse = argparse.ArgumentParser()
    parse.add_argument('-a', '--another_name', default='我是默认参数')
    args = parse.parse_args()

    print args.another_name
```
同理在终端中运行:
```
➜  parameter git:(python-note) ✗ python arg_demo.py                          
我是默认参数
➜  parameter git:(python-note) ✗ python arg_demo.py -a输入参数
输入参数
```
可以看到如果不输入就输入默认值，输入了我们引用别名`another_name`也可以输出`-a`的值

### 解析参数的其他属性
`argparse`还有很多可选参数，用来设置我们解析参数的具体操作，例如:

* dest
这个参数表示绑定参数在程序中对应的变量名称
```
add_argument("a",dest='code_name')
```
表示`a`参数直接绑定到程序中的变量值`code_name`

* default
为参数提供默认值，如果没有输入这个参数，就用默认值代替,注意不带`-`的参数不能制定默认值，因为不带`-`必须输入参数值，也就没有不输入而采用默认值的场景

* help
参数的帮助文档，一般用来告诉用户这个参数是什么意思，起提示和指导作用

* type
为参数指定一个类型，一般不指定的时候，默认会把输入的参数解析成字符串类型
```
add_argument("c", type=int)
```

* action
指对参数的具体操作

参数 | 解释
-----|-----
store | 默认就是这个模式，存储值到制定变量
store_const | 存储值在参数的const部分指定，多用于实现非布尔的命令行flag
store_true/store_falst | 布尔开关，store_true默认为False,输入则为True,store_false则相反
append | 存储值到列表，该参数可以重复使用
append_const | 存储值到列表，存储值在参数的const部分指定
count | 统计参数简写输入的个数
version |输出版本信息然后退出

* const
配合`action＝"store_const|append_const"`使用,默认值

* choices
输入值的范围
```
add_argument("--gb", choices=['A', 'B', 'C', 0])
```

* required
默认为`False`，若为`True`，则必须输入该参数.
