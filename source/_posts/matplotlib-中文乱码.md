title: matplotlib 中文乱码
date: 2016-02-25 20:37:40
tags: [matplotlib, Python, Linux]
categories: Python笔记
---
在使用matplotlib画图的时候，出现了以下错误：
> 
/home/q/python27/lib/python2.7/site-packages/matplotlib-1.5.1-py2.7-linux-x86_64.egg/matplotlib/font_manager.py:1288: UserWarning: findfont: Font family [u'sans-serif'] not found. Falling back to Bitstream Vera Sans
  (prop.get_family(), self.defaultFamily[fontext]))

这让我十分费解，应该有这个字体的，于是让其他人登陆跑同样的程序则可以运行，于是我好奇登陆了交互式环境，引入`matplotlib`包看了一下，结果出现以下错误：
```bash
Python 2.7.4 (default, Dec 24 2013, 16:30:42)
[GCC 4.4.6 20110731 (Red Hat 4.4.6-3)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import matplotlib
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/q/python27/lib/python2.7/site-packages/matplotlib-1.5.1-py2.7-linux-x86_64.egg/matplotlib/__init__.py", line 111, in <module>
    import inspect
  File "/home/q/python27/lib/python2.7/inspect.py", line 42, in <module>
    from collections import namedtuple
  File "collections.py", line 17, in <module>
    import numpy as np
  File "/home/q/python27/lib/python2.7/site-packages/numpy-1.10.4-py2.7-linux-x86_64.egg/numpy/__init__.py", line 180, in <module>
    from . import add_newdocs
  File "/home/q/python27/lib/python2.7/site-packages/numpy-1.10.4-py2.7-linux-x86_64.egg/numpy/add_newdocs.py", line 13, in <module>
    from numpy.lib import add_newdoc
  File "/home/q/python27/lib/python2.7/site-packages/numpy-1.10.4-py2.7-linux-x86_64.egg/numpy/lib/__init__.py", line 8, in <module>
    from .type_check import *
  File "/home/q/python27/lib/python2.7/site-packages/numpy-1.10.4-py2.7-linux-x86_64.egg/numpy/lib/type_check.py", line 11, in <module>
    import numpy.core.numeric as _nx
  File "/home/q/python27/lib/python2.7/site-packages/numpy-1.10.4-py2.7-linux-x86_64.egg/numpy/core/__init__.py", line 58, in <module>
    from numpy.testing import Tester
  File "/home/q/python27/lib/python2.7/site-packages/numpy-1.10.4-py2.7-linux-x86_64.egg/numpy/testing/__init__.py", line 10, in <module>
    from unittest import TestCase
  File "/home/q/python27/lib/python2.7/unittest/__init__.py", line 58, in <module>
    from .result import TestResult
  File "/home/q/python27/lib/python2.7/unittest/result.py", line 9, in <module>
```
这个时候就很郁闷了，服务器上的`matplotlib`都是我装的，结果反倒我不能用，但是后来在网上看到了解决方案:原来出现这个问题是由于用户目录下的缓存导致的，需要清空缓存:
```bash
sudo rm -r ~/.cache/
```
清空缓存之后就可以运行了。
