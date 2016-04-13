title: 高级特性:切片,迭代,生成式 
date: 2015-10-12 14:26:14
tags: [Python]
categories: [Python笔记]
---
Python崇尚的是代码越少越好，借鉴了其他各类语言的特性，另外考虑到日常编程中的一些非常繁琐的操作，Python对有一些非常常用的操作提供了简单的实现。

### Python高级特性

#### 切片
取一个`list`或`tuple`中的部分元素，当然其他语言，例如`java`也可以使用截取函数，传入区间进行截取，但是Python提供了一个更简单的操作
```bash
>>> L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
>>> L[0:3]
['Michael', 'Sarah', 'Tracy']
```
注意，3表示的不是截取的元素个数，而是索引结束位置，即不包括索引为3的元素，如果开始索引为0，还可以省略
```bash
>>> L[1:3]
['Sarah', 'Tracy']
>>> L[:3]
['Michael', 'Sarah', 'Tracy']
```
前面也提到过，Python取元素还支持`L[-1]`这种取倒数第一个元素的操作
```bash
>>> L[-2:]
['Bob', 'Jack']
>>> L[-2:-1]
['Bob']
```
`L[:]`，这个表示复制一个`list`，其实就是默认把整个`list`切片。

#### 迭代
这个和Java也差不多，在Java中也有迭代器以及`foreach(element: elements)`这种循环语句,在Python中，使用`for ... in`。
```bash
>>> for ch in 'abc':
...     print ch
... 
a
b
c
```
默认情况下，`dict`通过`key`迭代。也可以通过`value`来迭代：`for value in d.itervalues()`。也可以同时迭代`key`和`value`：`for k, v in d.iteritems()`。
所以，只要判断一个对象是可迭代对象就可以使用`for ... in`这种循环，通过`collections`模块的`Iterable`类型判断：
```bash
>>> from collections import Iterable
>>> isinstance('abc', Iterable) # str是否可迭代
True
>>> isinstance([1,2,3], Iterable) # list是否可迭代
True
>>> isinstance(123, Iterable) # 整数是否可迭代
False
```
有时候也需要里列表里的下标，这点Python也提供了一个内置的`enumerate`函数，可以把一个`list`变成`索引-元素对`，这样就可以做到在`for`循环中迭代索引和元素本身。
```bash
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print i, value
... 
0 A
1 B
2 C
```
还可以同时引用两个变量
```bash
>>> for x, y in [(1, 1), (2, 4), (3, 9)]:
...     print x, y
...
1 1
2 4
3 9
```

#### 列表生成式
即创建列表的方式，最笨的方法就是写循环逐个生成，前面也介绍过可以使用`range()`函数来生成，不过只能生成线性列表，下面看看更为高级的生成方式：
```bash
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```
写列表生成式时，把要生成的元素`x * x`放到前面，后面跟`for`循环，就可以把`list`创建出来，十分有用，多写几次，很快就可以熟悉这种语法。
你甚至可以在后面加上`if`判断：
```bash
>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```
循环嵌套，全排列：
```bash
>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```
看一个简单应用，列出当前目录下所有文件和目录：
```bash
>>> import os
>>> [d for d in os.listdir('.')]
['README.md', '.git', 'image', 'os', 'lib', 'sublime-imfix', 'src']
```

前面也说过Python里循环中可以同时引用两个变量，所以生成变量也可以：
```bash
>>> d = {'x': 'A', 'y': 'B', 'z': 'C' }
>>> [k + '=' + v for k, v in d.iteritems()]
['y=B', 'x=A', 'z=C']
```
也可以通过一个`list`生成另一个`list`，例如把一个list中所有字符串变为小写：
```bash
>>> L = ['Hello', 'World', 'IBM', 'Apple']
>>> [s.lower() for s in L]
['hello', 'world', 'ibm', 'apple']
```
但是这里有个问题，`list`中如果有其他非字符串类型，那么`lower()`会报错，解决办法：
```bash
>>> L = ['Hello', 'World', 'IBM', 'Apple', 12, 34]
>>> [s.lower() if isinstance(s,str) else s for s in L]
['hello', 'world', 'ibm', 'apple', 12, 34]
```

#### 生成器
列表生成式虽然强大，但是也会有一个问题，当我们想生成一个很大的列表时，会非常耗时，并且占用很大的存储空间，关键是这里面的元素可能你只需要用到前面很少的一部分，大部分的空间和时间都浪费了。Python提供了一种边计算边使用的机制，称为生成器(Generator)，创建一个Generator最简单的方法就是把`[]`改为`()`：
```bash
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x7fe73eb85cd0>
```

如果要一个一个打印出来，可以通过generator的next()方法：
```bash
>>> g.next()
0
>>> g.next()
1
>>> g.next()
4
>>> g.next()
9
>>> g.next()
16
>>> g.next()
25
>>> g.next()
36
>>> g.next()
49
>>> g.next()
64
>>> g.next()
81
>>> g.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```
其实`generator object`也是可迭代的，所以可以用循环打印，还不会报错。
```bash
>>> g = (x * x for x in range(10))
>>> for n in g:
...     print n
...
0
1
4
9
16
25
36
49
64
81
```
这是简单的推算算法，但是如果算法比较复杂，写在`()`里就不太合适了，我们可以换一种方式，使用函数来实现。
比如，著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：
> 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

斐波拉契数列用列表生成式写不出来，但是，用函数把它打印出来却很容易：
```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print b
        a, b = b, a + b
        n = n + 1
```
上面的函数可以输出斐波那契数列的前N个数,这个也是通过前面的数推算出后面的，所以可以把函数变成`generator object`，只需要把`print b`改为`yield b`即可。
```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
```
如果一个函数定义中包含了`yield`关键字，这个函数就不在是普通函数，而是一个`generator object`。
```bash
>>> fib(6)
<generator object fib at 0x7fa1c3fcdaf0>
>>> fib(6).next()
1
```
所以要想调用这个函数，需要使用`next()`函数，并且遇到`yield`语句返回(可以把yield理解为return):
```python
def odd():
    print 'step 1'
    yield 1
    print 'step 2'
    yield 3
    print 'step 3'
    yield 5
```
看看调用输出结果：
```bash
>>> o = odd()
>>> o.next()
step 1
1
>>> o.next()
step 2
3
>>> o.next()
step 3
5
>>> o.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```
同样也可以改为`for`循环语句输出。例如：
```python
def odd():
    print 'step 1'
    yield 1
    print 'step 2'
    yield 2
    print 'step 3'
    yield 3

if __name__ == '__main__':
    o = odd()
    while True:
        try:
            print o.next()
        except:
            break
```
