title: Python 正则表达式笔记
date: 2016-07-10 12:34:44
tags: [Python,正则]
categories: Python笔记
---
总结一下在工作中常用到的关于Python正则的一些用法,主要无非就是匹配提取置顶信息，或者替换指定信息,不过都是`re`模块的用法

### 查找匹配
查找和匹配主要就是`re.search()`以及`re.match()`
* match:从字符串的开始处匹配,匹配成功会返回`match object`,如果匹配不上返回None
* search:只要又字串符合就匹配成功，返回`match object`,如果没有一个子串满足匹配则返回None
* findall: 如果能匹配，返回所有的匹配结果list

下面会以代码介绍一下如何使用以及提取出匹配到的内容:

#### match,search
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re

__author__ = 'anonymous'

if __name__ == '__main__':
    str1 = 'hello world, test test Test'
    if not re.match('world', str1, re.I):
        print '1:not match'
    print '2:', re.match('hello', str1, re.I).group()
    print '3:', re.match('.*test', str1).group()
    print '4:', re.match('.*test', str1, re.I).group()
    print '5:', re.search('test', str1, re.I).group()
    pass
```
输出结果如下：
```
1:not match
2: hello
3: hello world, test test
4: hello world, test test Test
5: test
```
可以看到`match()`函数会从开头开始匹配，如果要匹配的内容不是在字符串的开头，那么需要加`.*`或者类似的通配符，否则返回None，另外`re.I`表示的是忽略大小写，从3,4的输出可以看出差别。而`search()`函数则是在整个字符串里面去查找给定的模式，没有位置限制,但是只会匹配第一个，就算后面还有也不会输出来。

#### findall
如果想获取指定内容应该怎么做呢？这里介绍一下正则里面的`()`,用括号匹配出来的内容可以通过`groups()`提取，看下面的代码:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re

__author__ = 'anonymous'

if __name__ == '__main__':
    str1 = 'sfasfasfsfaname=hello phone=234123423 mail=23423@qq.com,hslfas7'

    result = re.match(r'.*name=(\w*).*phone=(\d*) mail=(\d*@\w*.com).*', str1, re.I)

    if result is not None:
        print result.groups()
    pass
```
看一下输出结果：
```
('hello', '234123423', '23423@qq.com')
```
可以看到，匹配结果被保存到了一个tuple里面，只要遍历这个tuple或者访问指定顺序的下标就可以获取相应的匹配结果了,前面也说了，用`search()`匹配只会输出第一个匹配结果，而且上面的`match()`如果换成`search()`，调用`groups()`只会返回一个空的tuple,如果想返回字符串中所有出现的**as**应该怎么做呢？这个时候就需要用`findall()`:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re

__author__ = 'anonymous'

if __name__ == '__main__':
    str1 = 'sfasfasfsfaname=hello phone=234123423 mail=23423@qq.com,hslfas7'

    result1 = re.findall(r'name=(\w*).*phone=(\d*) mail=(\d*@\w*.com)', str1, re.I)
    result2 = re.findall(r'as', str1, re.I)
    print result1
    print result2
    pass
```
程序的输出结果为
```
[('hello', '234123423', '23423@qq.com')]
['as', 'as', 'as']
```
可以看到`findall()`函数返回的是一个list，返回所有匹配的结果，如果正则里面有`()`提取内容，这写内容也会出现在里面，单独保存在一个tuple中.


### 正则分割
在Python中，如果想对一个字符串进行分割的话，只需要调用str的split方法就可以实现，但是这个split只能根据某个字符来进行分割的操作，如果要同时指定多个字符来进行分割的话，它就无法实现了。
好在re模块也提供了split这个方法来对字符串进行分割，而且这个方法更加强大，可以同时根据多个字符进行分割的操作，下面来看分别看一下str的split和re的split有什么不同的地方：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re

__author__ = 'anonymous'

if __name__ == '__main__':
    str1 = 'helloword,i;am\nalex'
    str2 = str1.split(',')
    print str2

    str3 = re.split('[,|;|\n]', str1)
    print str3
```
看看输出结果:
```
['helloword', 'i;am\nalex']
['helloword', 'i', 'am', 'alex']
```
这里就可以看到差别是啥了，正则分割确实功能更加强大也更加灵活，可以用正则模式分割一个字符串

### 正则替换
讲到正则分割，当然少不了正则`replace()`，同理字符串也有替换方法，但是只能替换指定的内容，如果想达到模糊替换，只能用正则，不过正则里面的替换方法不叫`replace`，叫`sub()`.
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re

__author__ = 'anonymous'

if __name__ == '__main__':
    str1 = 'Hello 111 is 222'
    str2 = re.sub(r'\d+', '333', str1)
    print str2
```
输出结果为:
```
Hello 333 is 333
```
