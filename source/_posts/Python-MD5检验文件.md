title: Python MD5校验文件 
date: 2015-12-18 20:32:38
tags: Python
categories: Python笔记
---
今天看到了一个用Python校验文件md5的代码,但是本人Python不是很熟,所以里面有很多语法看的不是很懂,大概讲讲吧.
首先要想校验一个文件的md5值在Linux下很方便,命令行就可以校验:
```bash
➜  ~  md5sum 订单数据.xls 
933ed658af57a9aec4d1ba9800fe5460  订单数据.xls
```
虽然你也可以调用Python来校验文件的md5,但是这样很不方便,也不能保证windows下也可以运行吧,Python中也md5校验的函数,Python 2.5之后推荐使用`hashlib`来代替`md5`模块来做校验.
### 简单的MD5校验

```python
#!/usr/bin/env python
# coding=utf-8
import hashlib


def md5sum(file_name):
    fp = open(file_name, 'rb')
    content = fp.read()
    fp.close()
    m = hashlib.md5(content)
    file_md5 = m.hexdigest()

    return file_md5


if __name__ == '__main__':
    print md5sum('TCPServer.py')
```
 **注意:**`hashlib.md5()`返回的是一个对象,要想得到md5值,还需要调用一下`hexdigest()`方法.还有一个地方需要注意一下,这个校验方法有个很大的缺陷,由于是要校验文件里面的内容,所以每次是把文件读到内存的,试想一下,如果这个文件很大,先不说慢,更有可能程序就直接卡死了.

### 改进的MD5校验
下面的代码确实写的很牛逼,对于我这种刚学Python的渣渣来说,很多地方都看的不太明白,先把程序贴出来,后面有一些程序中不懂的地方的一些详尽的解释.优化之后的代码:
#### 加密字符串
```python
def md5hex(word):
    """
    MD5加密算法,返回32为小写16进制符号
    :param word: 需要加密的字符串
    :return: 返回32为小写16进制符号
    """
    if isinstance(word, unicode):
        word = word.encode('utf-8')
    elif not isinstance(word, str):
        word = str(word)

    m = hashlib.md5()
    m.update(word)

    return m.hexdigest()
```
**解释:**查了一下Python里关于`str`和`unicode`的区别,简单来说就是`str`类似于`byte[]`,即字节字符串.而`unicode`类似于`char[]`.说白了就是Python内部存储用`unicode`,而在和人交互的时候用`str`,str是字节串，由unicode经过编码(encode)后的字节组成的,所以它们的关系就是`unicode.encode()-->str`或者`str.decode()-->unicode()`.时刻记住:
> **unicode**才是真正的字符串.

#### 加密文件
```
def md5sum(file_name):
    """
    根据给定文件名计算文件MD5值
    :param file_name: 文件的路径
    :return: 返回文件MD5校验值
    """

    def read_chunks(fp):
        fp.seek(0)
        chunk = fp.read(8 * 1024)
        while chunk:
            yield chunk
            chunk = fp.read(8 * 1024)
        else:	# 最后要将游标放回文件开头
            fp.seek(0)

    m = hashlib.md5()
    if isinstance(file_name, basestring) \
            and os.path.exists(file_name):
        with open(file_name, 'rb') as fp:
            for chunk in read_chunks(fp):
                m.update(chunk)
    elif file_name.__class__.name__ in ["StringIO", "cStringIO"] \
            or isinstance(file_name, file):
        for chunk in read_chunks(file_name):
            m.update(chunk)
    else:
        return ""

    return m.hexdigest()
```
这段程序写的确实很牛逼,我稍微解释一下:
1. `8~15`行:的意思就是每次返回读取的文件的8k的内容,下次从上次读取的地方继续读取,如果读完了,就把游标重新放到文件开头.注意这里面用到了`yield`,对这个关键字不熟的可以简单把他理解为关键字`return`.但是它和`return`不同的地方在于,再次调用函数的时候,上次调用的变量还是有效的,也就是说它不是从文件的开头重新读,而是接着上回读的地方再读8k的内容
2. `18`行:`isinstance(file_name, basestring)`这个是判断文件名是不是一个`basestring`类型,`basestring`是`str`和`unicode`的父类,这个类是个抽象类,不能被实例化,不过可以被用来判断一个对象是不是字符串.`with`关键字,简单理解为`try/catch`就可以了,即使发生了异常,文件也可以正常关闭.
3. `23`行:`StringIO`和`cStringIO`与`file`对象非常像,指内存里的文件,例如上传的文件缓存或者已经打开的文件流,那么就不用打开文件,直接读就行

### 总结
这个程序其实就是每次读取8k的内容,然后更新已经读取的内容的`md5`值,核心代码:
```python
for chunk in read_chunks(fp):
	m.update(chunk)
```
如果不明白`update()`的用法,可以测试一下:
```bash
>>> import hashlib
>>> x = hashlib.md5()
>>> x.update('hello, ')
>>> x.update('python')
>>> x.hexdigest()
'fb42758282ecd4646426112d0cbab865'
>>> hashlib.md5('hello, python').hexdigest()
'fb42758282ecd4646426112d0cbab865'
>>> 
```
所以可以看到,`update()`方法其实并不会丢弃上一次校验的结果,就是在不停的累加,尤其适合处理大文件.
