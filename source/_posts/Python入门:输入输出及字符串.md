title: 输入输出及字符串 
date: 2015-10-09 16:14:18
tags: [Python]
categories: [Python笔记]
---
### 开发环境
```bash
os: Ubuntu 14.04 (Linux 3.13.0-32-generic linux x64)
python: Python 2.7.6 (default, Mar 22 2014, 22:59:56)
IDE: pyCharm 4.5.4
```

#### 准备工作
* 安装python
`Windows`用户需要手动安装`python`环境，`Ubuntu`自带`python`，所以这步跳过，不用安装，如果需要安装指定版本，则需要去官网下载手动安装，考虑到很多`python`的库都是基于`python 2.7.x`编写的，所以没啥大问题我就默认使用`python 2.7.6`了。

* 安装IDE
这个看个人喜好，我安装的是`PyCharm 4.5.4`，首先需要去官网下载，根据你的系统下载不同的包，下载页面是[Download PyCharm](http://www.jetbrains.com/pycharm/download/)。网速不是很好，我是开代理下载的，如果下载太慢，可以去我的百度网盘下载[PyCharm 4.5.4 百度云]()

* 解压到指定目录：
```bash
tar -xvf pycharm-professional-4.5.4.tar.gz -C /usr/devlop
```

* 运行`pycharm`
可以在`/usr/devlop/pycharm-4.5.4/bin`下找到一个脚本`pycharm.sh`。运行脚本即可,具体方法是:
```bash
cd /usr/devlop/pycharm-4.5.4/bin
./pycharm.sh
```

#### python基本知识
* 输出
```bash
>>> print 'hello, world'
hello, world

>>> print 'The quick brown fox', 'jumps over', 'the lazy dog'
The quick brown fox jumps over the lazy dog

>>> print '100 + 200 =', 100 + 200
100 + 200 = 300

```

* 输入
```bash
>>> name = raw_input()
Michael

>>> name
'Michael'

>>> print name
Michael

>>> name = raw_input('please enter your name: ')
please enter your name:
```
 **注意：** `raw_input`返回的永远是字符串，也就是说你输入一个`int`型，返回的是一个数字字符串，你需要进行转换：
```bash
>>> number = raw_input("输入一个整数：")
输入一个整数：123
>>> number
'123'
>>> number = int(raw_input("输入一个整数："))
输入一个整数：123
>>> number
123
```

* 字符串
字符串用`''`或者`""`括起来，如果字符串内部有`‘`或者`"`，需要使用`\`进行转义
```bash
>>> print 'I\'m ok.'
I'm ok.
```
 转义字符`\`可以转义很多字符，比如`\n`表示换行，`\t`表示制表符，字符`\`本身也要转义，所以`\\`表示的字符就是`\`。当然如果不需要转义，可以使用`r''`：
```bash
>>> print '\\\t\\'
\       \
>>> print r'\\\t\\'
\\\t\\
```
 如果字符串内部有很多换行，用\n写在一行里不好阅读，为了简化，Python允许用'''...'''的格式表示多行内容:
```bash
>>> print '''line1
... line2
... line3'''
line1
line2
line3
```
 如果写成程序，就是：
```python
print '''line1
line2
line3'''
```

* 空值
`python`里空值用`None`表示，相当于`Java/C++`里的`null`。

* 变量
`python`是动态语言，里面的变量不像`Java/C++`里，脚本语言都比较宽松，所以代码在声明变量的时候都不需要显示的声明类型：
```python
a = 123 # a是整数
print a
a = 'ABC' # a变为字符串
print a
```

* 常量
所谓常量就是不能变的变量，比如常用的数学常数π就是一个常量。在Python中，通常用全部大写的变量名表示常量：
```python
PI = 3.14159265359
```
 **注意：**`python`中即使这样定义了，也无法阻止你修改常量，可以随便改的，所以这只是个约定，但是最好不要改动。这里有个地方需要注意一下，`PI`是变量，但是其内容对象是不可变对象，看下面的例子：
```bash
>>> a = 'abc'
>>> b = a.replace('a', 'A')
>>> b
'Abc'
>>> a
'abc'
```

* 注释
 - 单行注释
 `#` 后面的内容为注释内容

 - 特殊注释
 ```bash
 #! /usr/bin/python
 ```
 告诉`Linux/Unix`去找到`python`的翻译器，这样有个好处，脚本可以直接运行，而不用`python hello.py`这样。

 - 编码注释
 ```bash
 # coding=utf-8
 ```
 **注意：**这个只能放在第二行，多一个空格都不行。

 - doc String注释
这个东西主要是用于模块、函数和类的描述，类似于下面：
```python
#!/usr/bin/env python
# coding=utf-8

def testDocString():
    """
    测试文档注释
    :return:
    """
    pass
 
print testDocString.__doc__

# 然后在终端中运行一下脚本，看看最后输出内容
➜  python  ./Comments.py      
     测试文档注释
    :return:
   
```

#### 可能出现的问题
* 中文编码问题
 
>\# coding = utf-8

 结果报错：
>SyntaxError: Non-ASCII character '/xe6'

 所以最后改成了
>\# coding=utf-8

* Unicode编码问题
```bash
➜  ~  python
Python 2.7.6 (default, Mar 22 2014, 22:59:56) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> len('中文')
6
>>> len(u'中文')
2
>>> 
```
 **注意：** 这个问题是由`python`编码导致的，详细的编码问题详见[字符串和编码](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386819196283586a37629844456ca7e5a7faa9b94ee8000)，但是在`python 3.x`中这个编码问题就不存在了：
```bash
➜  ~  python3 
Python 3.4.0 (default, Jun 19 2015, 14:20:21) 
[GCC 4.8.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> len('中文')
2
>>> len(u'中文')
2
>>> 
```

最常见的还不是编码带来的长度问题不一致，编码主要会导致一些系统异常，常见的有两种:
* encode(编码)异常

常见的就是`UnicodeEncodeError: 'ascii' codec can't encode character u'xxxx' in position xx: ordinal not in range(128)`,
意思就是把一个字符串编码为str的时候，系统默认是编码成`ascii`编码格式，但是中文里面的编码超出了`ascii`编码的最大范围，直接
报错，常见的就是直接掉`str()`函数的时候没有指定编码。建议少用`str()`函数，采用`xxx.encode('utf8')`这种显示的声明

* decode(解码)异常

道理是一样的，默认也是用`ascii`解码，如果是中文的话，需要手动指定`uft-8`解码方式.

