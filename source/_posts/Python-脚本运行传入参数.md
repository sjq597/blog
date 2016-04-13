title: Python 脚本运行传入参数
date: 2015-11-06 19:25:41
tags: [Python]
categories: Python笔记
---
想让Python获取脚本执行时后面的参数,可以这么做:
先看看测试代码:
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
第一个就是python脚本的名字,后面四个就是四个参数了.
