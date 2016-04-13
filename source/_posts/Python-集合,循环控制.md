title: 集合,循环控制 
date: 2015-10-09 20:17:47
tags: [Python]
categories: [Python笔记]
---
### python常用结构

`python`常用结构有集合以及循环，另外集合包括列表和集合两大类。

#### 集合类

* 有序列表list
```bash
>>> listTest = ['ha','test','yes']
>>> listTest
['ha', 'test', 'yes']
```

 `len()`获取`list`元素个数。
```bash
>>> len(listTest)
3
```
 可以用索引来访问每一个元素，`0`表示第一个，`-1`还可以表示最后一个，即倒数第一个，依此类推`-2`表示倒数第二个，超过了也会报越界错误。
```bash
>>> listTest[0]
'ha'
>>> listTest[1]
'test'
>>> listTest[3]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: list index out of range
>>> listTest[-1]
'yes'
>>> listTest[-2]
'test'
```

 也可以把元素插入到指定的位置，比如索引号为`1`的位置：
```bash
>>> listTest.insert(1,'jack')
>>> listTest
['ha', 'jack', 'test', 'yes']
```

 删除末尾元素，用`pop()`方法，添加到末尾用`append()`：
```bash
>>> list
['ha', 'jack', 'test', 'yes']
>>> listTest.pop()
'yes'
>>> listTest
['ha', 'jack', 'test']
```

 删除指定位置的元素，用`pop(i)`方法,其中`i`是索引位置:
```bash
>>> listTest
['ha', 'jack', 'test']
>>> listTest.pop(1)
'jack'
>>> listTest
['ha', 'test']
```

 把某个元素替换，直接赋值即可,并且类型也可以不同：
```bash
>>> listTest
['ha', 'test']
>>> listTest[1] = 'debug'
>>> listTest
['ha', 'debug']
>>> listTest[1] = 123
>>> listTest
['ha', 123]
```

 `list`也可以嵌套:
```bash
>>> s = ['python', 'java', ['asp', 'php'], 'scheme']
>>> len(s)
4
>>> s[1]
'java'
>>> s[2]
['asp', 'php']
>>> s[2][1]
'php'
```

 空的`list`:
```bash
>>> L = []
>>> len(L)
0
```

* 不可变列表tuple
另一种有序列表叫元组：tuple。tuple和list非常类似，但是tuple一旦初始化就不能修改。
```bash
>>> classmates = ('Michael', 'Bob', 'Tracy')
>>> classmates
('Michael', 'Bob', 'Tracy')
>>> classmates[1]
'Bob'
```
 **注意：**由于`tuple`不可变，所以代码更安全，如果可能，能用`tuple`代替`list`就尽量用`tuple`。
 `tuple的陷阱：`当你定义一个`tuple`时，在定义的时候，`tuple`的元素就必须被确定下来，比如：
```bash
>>> t = (1, 2)
>>> t
(1, 2)
```
 如果要定义一个空的`tuple`，可以写成()：
```bash
>>> t = ()
>>> t
()
```
 但是，要定义一个只有1个元素的`tuple`，如果你这么定义：
```bash
>>> t = (1)
>>> t
1
```
 定义的不是`tuple`，是`1`这个数！这是因为括号`()`既可以表示`tuple`，又可以表示数学公式中的小括号，这就产生了歧义，因此，`Python`规定，这种情况下，按小括号进行计算，计算结果自然是`1`。

 所以，只有1个元素的`tuple`定义时必须加一个逗号,，来消除歧义：
```bash
>>> t = (1,)
>>> t
(1,)
```
 `Python`在显示只有1个元素的`tuple`时，也会加一个逗号`,`，以免你误解成数学计算意义上的括号。
 最后来看一个`可变的`tuple：
```bash
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```
 **注意：**`tuple`所谓的`不变`是说，`tuple`的每个元素，指向永远不变。

* Map
`Python`内置了字典：`dict`的支持，`dict`全称`dictionary`，在`Java/C++`中也称为`Map`，使用键-值（`key-value`）存储，具有极快的查找速度。
```bash
>>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
>>> d['Michael']
95
```

 把数据放入`dict`的方法，除了初始化时指定外，还可以通过`key`放入：
```bash
>>> d['Adam'] = 67
>>> d['Adam']
67
```
 **注意：**如果key不存在，dict就会报错：
```bash
>>> d['Thomas']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'Thomas'
```
 要避免key不存在的错误，有两种办法，一是通过in判断key是否存在：
```bash
>>> 'Thomas' in d
False
```
 或者通过`dict`提供的`get`方法，如果`key`不存在，可以返回`None`，或者自己指定的value：
```bash
>>> d.get('Thomas')
>>> d.get('Thomas', -1)
-1
```

 删除`dict`里的元素，用`pop(key)`方法，对应的`value`也会从`dict`中删除：
```bash
>>> d.pop('Bob')
75
>>> d
{'Michael': 95, 'Tracy': 85}
```
 正确使用`dict`非常重要，需要牢记的第一条就是`dict`的`key`必须是不可变对象。

* set集合
`set`和`dict`类似，也是一组`key`的集合，但不存储`value`。由于`key`不能重复，所以，在`set`中，没有重复的`key`。
要创建一个`set`，需要提供一个`list`作为输入集合：
```bash
>>> s = set([1, 2, 3])
>>> s
set([1, 2, 3])
```
 `set`会自动过滤重复元素，`add(key)`添加元素，可以重复添加，但是没有效果。`remove(key)`删除元素。对于两个集合，还可以通过`&`取集合的交集，`|`取集合的并集。

#### 循环控制

* 条件判断
之前学过`Java/C++`可能会不太习惯这种方式，先看看`python`的条件判断代码：
```python
age = 20
if age >= 18:
    print 'your age is', age
    print 'adult'
```
 根据`Python`的缩进规则，缩进的代码块就相当于`Java/C++`里大括号的内容。即`if`为`True`,执行代码块内容，加`else`，`elif`同理，注意不要漏掉`:`。
```python
if <条件判断1>:
    <执行1>
elif <条件判断2>:
    <执行2>
elif <条件判断3>:
    <执行3>
else:
    <执行4>
```
 **注意：** `if`还可以简写，下面这个和`Java/C++`有很大的不同，`python`对类型的判断很宽松，只要`x`是非零数值、非空字符串、非空`list`等，就判断为`True`，否则为`False`。
```python
if x:
    print 'True'
```

* 循环控制
循环和`Java/C++`就很像了，也是有两种:`for`循环和`while`循环。
`for x in ...`循环就是把每个元素代入变量`x`，然后执行缩进代码块:
```python
names = ['Michael', 'Bob', 'Tracy']
for name in names:
    print name
```

 `while`循环,只要条件满足，就不断循环
```python
sum = 0
n = 99
while n > 0:
    sum = sum + n
    n = n - 2
print sum
```
