title: 函数及函数参数 
date: 2015-10-10 14:39:07
tags: [Python]
categories: [Python笔记]
---
方法或者说函数的使用，这个和其他编程语言`Java/C++`没什么区别，使用正确的方法名加参数就可以，`python`还有一个比较人性化的地方：在交互命令行下通过`help(functionName)`可以查看指定函数的使用帮助。例如：
```bash
>>> help(range)
```
会出现如下内容，按`q`退出：
```
Help on built-in function range in module __builtin__:

range(...)
    range(stop) -> list of integers
    range(start, stop[, step]) -> list of integers
    
    Return a list containing an arithmetic progression of integers.
    range(i, j) returns [i, i+1, i+2, ..., j-1]; start (!) defaults to 0.
    When step is given, it specifies the increment (or decrement).
    For example, range(4) returns [0, 1, 2, 3].  The end point is omitted!
    These are exactly the valid indices for a list of 4 elements.
(END)
```

### 函数
* 定义函数
在Python中，定义一个函数要使用`def`语句，依次写出函数名、括号、括号中的参数和冒号`:`，然后，在缩进块中编写函数体，函数的返回值用`return`语句返回。
我们以自定义一个求绝对值的`my_abs`函数为例：
```python
def my_abs(x):
    if x >= 0:
        return x
    else:
        return -x
```

* 空函数
```python
def nop():
    pass
```
 `pass`语句什么都不做，那有什么用？实际上`pass`可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。
 `pass`还可以用在其他语句里，比如：
```python
if age >= 18:
    pass
```
 缺少了`pass`，代码运行就会有语法错误。

* 参数检查
调用函数时，如果参数个数不对，Python解释器会自动检查出来，并抛出`TypeError`：
```bash
>>> my_abs(1, 2)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: my_abs() takes exactly 1 argument (2 given)
```

 但是如果参数类型不对，Python解释器就无法帮我们检查。试试`my_abs`和内置函数`abs`的差别：
```bash
>>> my_abs('A')
'A'
>>> abs('A')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: bad operand type for abs(): 'str'
```
 当传入了不恰当的参数时，内置函数`abs`会检查出参数错误，而我们定义的`my_abs`没有参数检查，所以，这个函数定义不够完善。

 让我们修改一下`my_abs`的定义，对参数类型做检查，只允许整数和浮点数类型的参数。数据类型检查可以用内置函数`isinstance`实现：
```python
def my_abs(x):
    if not isinstance(x, (int, float)):
        raise TypeError('bad operand type')
    if x >= 0:
        return x
    else:
        return -x
```
 添加了参数检查后，如果传入错误的参数类型，函数就可以抛出一个错误：
```bash
>>> my_abs('A')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in my_abs
TypeError: bad operand type
```
 错误和异常处理将在后续讲到，此处只需要大概知道这个可以抛出一个异常就行了。

* 返回多个值
一般像`Java/C++`这样的语言，要想函数返回多个值，通常有两种方式：第一种，如果返回值的类型一样，那么一般就是返回一个列表或者数组对象，然后要用就去列表里取；第二种，返回值类型不一样，这种情况通常是定义一个结构体或者类来接收。但是对于Python，就不用这么麻烦了，看下面的代码：
```python
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny
```
 在命令式交互行里看看返回值，`x`和`y`也可以直接单独使用：
```bash
>>> x, y = move(100, 100, 60, math.pi / 6)
>>> print x, y
151.961524227 70.0
```
 **注意：**虽然可以这么用，但是具体中间的过程还必须了解，其实返回值还是单一值，看下面：
```bash
>>> r = move(100, 100, 60, math.pi / 6)
>>> print r
(151.96152422706632, 70.0)
```
 可以看到，返回值是一个`tuple`！，但是`python`的语法上返回一个`tuple`可以省略括号，同时多个变量又可以同事接收一个`tuple`，即相当于
```python
x,y = r
```

* 函数参数
`C++`里函数可以设置缺省参数，`Java`不可以，只能通过重载的方式来实现，`python`里也可以设置默认参数,最大的好处就是降低函数难度，函数的定义只有一个，`并且python是动态语言，在同一名称空间里不能有想多名称的函数，如果出现了，那么后出现的会覆盖前面的函数`。
```python
def power(x, n=2):
    s = 1
    while n > 0:
        n = n - 1
        s = s * x
    return s
```
 看看结果：
```bash
>>> power(5)
25
>>> power(5,3)
125
```
 **注意：** 必选参数在前，默认参数在后，否则Python的解释器会报错。
 **建议：***当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面。变化小的参数就可以作为默认参数。

 默认参数也有坑，看看下面的代码，先定义一个`list`，添加一个`end`再返回：
```python
def add_end(L=[]):
    L.append('END')
    return L
```
 看看调用结果：
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
 这里需要解释一下，`Python`函数在定义的时候，默认参数`L`的值就被计算出来了，即`[]`。此时`L`指向`[]`。所以如果`L`中的内容改变了，下次调用引用的内容也就不再是`[]`了。所以要牢记一点`定义默认参数必须指向不可变对象！`。具体想深入了解可以看看[Python 默认参数陷阱](../Python-默认参数陷阱/index.html)

* 可变参数
第一种方法，传入的参数为一个`list`或者`tuple`。
```python
def calc(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
 调用方式：
```bash
>>> calc([1, 2, 3])
14
>>> calc((1, 3, 5, 7))
84
```
 第二种方式，直接传入多个参数，函数内部会自动用一个`tuple`接收。
```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
 调用方式：
```bash
>>> calc(1, 2)
5
>>> calc()
0
```
 这个时候如果还想把一个`list`或者`tuple`里的数据传进去，可以这样：
```bash
>>> nums = [1, 2, 3]
>>> calc(*nums)
14
```

* 关键字参数
关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个`dict`。
```python
def person(name, age, **kw):
    print 'name:', name, 'age:', age, 'other:', kw
```
 调用示例：
```bash
>>> person('Michael', 30)
name: Michael age: 30 other: {}
>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```

* 参数组合
在Python中定义函数，可以用必选参数、默认参数、可变参数和关键字参数，这4种参数都可以一起使用，或者只用其中某些，但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数和关键字参数。

* 递归函数
基本的也没什么可讲的，和`Java/C++`里一样，就是调用本身的一种。这里重点介绍一下`尾递归`优化。事实上尾递归和循环效果是一样的，很显然的一个优点那就是可以防止递归调用栈溢出。
**定义：**在函数返回的时候调用自身，并且，`return语句不能包含表达式`。编译器或者解释器可以对其做优化，无论调用多少次，只占用一个栈帧，不会出现溢出的情况。
举个简单的例子，以阶乘函数为例：
```python
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)
```
 如果传入的n很大，就可能会溢出，这是由于`return n * fact(n - 1)`引入了乘法表达式，就不是尾递归了。把代码改一下：
```python
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)
```
 但是一般不要用，有两点原因：1.python不支持尾递归优化；2.这样优化的代码可读性和逻辑性没有之前单纯的递归清晰。

