title: Python yield剖析
date: 2015-12-20 15:21:58
tags: [Python]
categories: Python笔记
---
首先,这个东西在其他的面向对象的语言里好像没有,所以第一次碰到难免会有些不好理解,网上看到的教程,做个笔记整理以后不懂再来看看.
要讲`yield`就必须先讲讲Python中的迭代器(iterator)和生成器(generator)

### 迭代器(iterator)
在Python中,for循环比其他的语言中的都要强大,可以用来遍历Python中的任何类型,包括列表,元祖等,其实根本的判断原则就是:
> for 循环可以用于任何 **可迭代对象**

这个含义就是迭代器,迭代器是一个实现了迭代器协议的对象,Python中的迭代器协议规定了必须有`next()`方法,调用这个方法会前进到下一个结果,当迭代器到达末尾的时候,会触发`StopIteration`.具有这些特性的对象在Python中都可以用for循环或其他遍历工具迭代,迭代的时候,每次都会调用`next()`方法,并且一旦捕捉到`StopIteration`异常就会停止迭代.
使用迭代器除了在写代码的时候看上去更简洁,优雅,更大的一个好处就是:每次只从对象中读取一条数据,不会造成过大的开销.
举个读文件的例子,一般我们会经常用到逐行读取一个文件的内容,利用`readlines()`方法,可以这么写:
```python
for line in open('test.txt','r').readlines():
	print line
```
看上去代码确实很优雅,简洁,但其实这并不是最好的方法,因为这其实也是一次把文件加载到了内存,然后再逐行遍历打印的,内存卡销很大,碰上一个很大的文件,程序有可能会崩溃.
这个时候我们可以利用`file`迭代器,优化之后的代码:
```python
for line in open('test.txt','r'):
	print line
```
虽然代码改动很小,甚至还去掉了一些代码,但是这个是运行速度最快的读文件的一种方法,没有显示的去读文件,而是利用迭代器每次读取下一行.

### 生成器(generator)
生成器函数在Python中与迭代器协议的概念联系在一起.简而言之,包含`yield`语句的函数会被特地编译成生成器.于是当我们调用这类函数的时候,会返回一个生成器对象,这个对象支持迭代器接口.而且就算函数有个`return`,它的作用也是用来`yield`产生值的.也就是不像一般的函数会生成值后退出,生成器函数在生成值后会自动挂起并暂停它们的执行状态,本地变量会保存状态信息,当函数恢复时将再度有效:
```python
def g(n):
    for i in range(n):
        yield i **2

for i in g(5):
    print i, ':',

0:1:4:9:16:
```
这么看好像也不太容易懂,我们用迭代器的`next()`方法来看看具体的运行原理:
```python
>>> t = g(5)
>>> t.next()
0
>>> t.next()
1
>>> t.next()
4
>>> t.next()
9
>>> t.next()
16
>>> t.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  StopIteration
```
在运行完5次`next()`之后，生成器抛出了一个`StopIteration`异常，迭代终止。再来看一个`yield`的例子，用生成器生成一个`Fibonacci`数列：
```python
>>> def fab(max):
...     n, a, b = 0, 0, 1
...     while n <= max:
...             yield b
...             a, b = b, a + b
...             n += 1
... 
>>> for n in fab(5):
...     print n,
... 
1 1 2 3 5 8
```
 **解释:**在`for`循环执行时，每次循环都会执行`fab`函数内部的代码，执行到`yield b`时，`fab`函数就返回一个迭代值，下次迭代时，代码从`yield b`的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到`yield`.
