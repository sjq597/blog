title: Python 多线程
date: 2016-06-18 19:23:09
tags: [Python, 多线程]
categories: Python笔记
---
### 全局解释器锁
谈到Python多线程,不得不先说一下全局解释器锁(GIL),Python代码的执行由Python虚拟机(也叫解释器主循环)来控制,虽然有多个进程,但是某一个时刻只会有一个线程在执行,对Python虚拟机的访问由全局解释器锁(global interpreter lock,GIL)来控制,正是由于有GIL,同一时刻只会有一个线程在运行,具体的执行步骤为:
```
1. 设置GIL
2. 切换到一个线程去运行
3. 运行
   a. 运行指定字节码的指令,或者
   b. 线程主动让出控制
4. 把线程设置为睡眠状态
5. 解锁GIL
6. 重复以上所有步骤
```
所以在调用外部代码的时候,GIL会被锁定,直到调用函数结束为止，看到这里,你可能会觉得Python程序的效率会非常低,毕竟我们的程序会去访问数据库,外部接口,加载本地文件,如果是这样,那我们的程序基本上无时无刻都是卡在那等着。其实你完全不用担心,所有面向I/O的(即调用内建的操作系统C代码)的程序,GIL会在这个I/O调用之前被释放,这样其他程序是可以在等待I/O的时候执行的。不过如果一个程序并没有很多I/O操作,那他只要运行,就一直占用CPU,多线程只对那些I/O密集的程序更有好处。


### threading模块
其实还有一个模块叫`thread`,但是这个模块使用特别麻烦,官方不建议使用,其中对于锁的操作特别麻烦,而且还有个很大的问题,一旦主进程退出,不管子线程运行完没有都会被强制结束。所以我也不介绍这个模块怎么用了，我们直接看`threading`模块的使用。
#### 传入可调用函数
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import threading
from time import sleep, ctime

__author__ = 'anonymous'

loops = [4, 2]


def loop(n_loop, n_sec):
    print '开始线程', n_loop, '于', ctime()
    sleep(n_sec)
    print 'loop函数', n_loop, '完成于', ctime()


def main():
    print '开始主线程：', ctime()
    threads = []        # 一个用于储存线程对象的列表
    n_loops = range(len(loops))
    for i in n_loops:
        t = threading.Thread(target=loop, args=(i, loops[i]))   # 每次循环创建一个Thread的实例
        threads.append(t)   # 将新创建的对象放到一个列表中

    for i in n_loops:
        threads[i].start()  # 每次循环运行一个线程

    or i in n_loops:
        threads[i].join()   # 等待子线程的完成

    print '主线程完成：', ctime()

if __name__ == '__main__':
    main()
```
输出结果为:
```
开始主线程： Sat Jun 18 19:47:40 2016
开始线程 开始线程0 于  Sat Jun 18 19:47:40 20161
 于 Sat Jun 18 19:47:40 2016
loop函数 1 完成于 Sat Jun 18 19:47:42 2016
loop函数 0 完成于 Sat Jun 18 19:47:44 2016
主线程完成： Sat Jun 18 19:47:44 2016

Process finished with exit code 0
```
一旦调用`start()`方法被调用的函数就开始执行了,如果你在主函数里面还要做其他操作，并且这个操作与子线程的执行无关，你完全不用调用`join()`，可以去做其他操作。调用`join()`会一直等待子线程执行完毕返回然后再执行后面的步骤。
**注意:**这个输出并不是连串的,因为我的CPU是多核的，所以日志看上去有点儿不太正常。`target`一个可调用对象，这里我们传入了一个函数;`args`是一个元祖，均以位置参数的方式传递给被调用对象。


#### 传入可调用类
我们也可以传入一个可调用类给`Thread`实例，不过类必须要实现`__call()__`方法，
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import threading
from time import sleep, ctime

__author__ = 'anonymous'

loops = [4, 2]


def loop(n_loop, n_sec):
    print '开始线程', n_loop, '于', ctime()
    sleep(n_sec)
    print 'loop函数', n_loop, '完成于', ctime()


class ThreadFunc(object):
    def __init__(self, func, args):
        self.func = func
        self.args = args

    def __call__(self):  # 关键是要实现这个方法
        apply(self.func, self.args)


def main():
    print '开始主线程：', ctime()
    threads = []  # 一个用于储存线程对象的列表
    n_loops = range(len(loops))
    for i in n_loops:
        t = threading.Thread(target=ThreadFunc(func=loop, args=(i, loops[i])), )  # 每次循环创建一个Thread的实例，目标是一个类
        threads.append(t)  # 将新创建的对象放到一个列表中

    for i in n_loops:
        threads[i].start()  # 每次循环运行一个线程

    for i in n_loops:
        threads[i].join()  # 等待子线程的完成

    print '主线程完成：', ctime()


if __name__ == '__main__':
    main()
```
函数的执行结果和上面是一样的，这里我就不再贴运行结果了。

#### 创建Thread子类
这种方式比较灵活，更加通用，只用在内部冲洗run方法即可
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import threading
from time import sleep, ctime

__author__ = 'anonymous'

loops = [4, 2]


def loop(n_loop, n_sec):
    print '开始线程', n_loop, '于', ctime()
    sleep(n_sec)
    print 'loop函数', n_loop, '完成于', ctime()


class MyThread(threading.Thread):
    def __init__(self, func, args):
        threading.Thread.__init__(self)  # 调用父类的构造函数
        self.func = func
        self.args = args

    def run(self):
        apply(self.func, self.args)


def main():
    print '开始主线程：', ctime()
    threads = []  # 一个用于储存线程对象的列表
    n_loops = range(len(loops))
    for i in n_loops:
        t = MyThread(func=loop, args=(i, loops[i]))  # 使用我们自己的类来新建对象
        threads.append(t)  # 将新创建的对象放到一个列表中
    for i in n_loops:
        threads[i].start()  # 每次循环运行一个线程

    for i in n_loops:
        threads[i].join()  # 等待子线程的完成

    print '主线程完成：', ctime()


if __name__ == '__main__':
    main()
```
调用子类`MyThread`的`start()`方法即可，同样`join()`用于等待自线程执行完。这种方式相比上面的两种方式好处在哪？想象一下，我们的函数执行如果返回的是一个二维数组，如果仅仅有`start(),join()`这样的方法，我们怎么获取最后我们执行的返回值呢？如果是子类，我们完全可以在调用`run()`方法里面把这个结果保存下来，然后最后调用子类的实例去获取这个变量就行了,像下面这样:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import threading
from time import sleep, ctime

__author__ = 'anonymous'

loops = [4, 2]


def loop(n_loop, n_sec):
    return n_loop + n_sec  # 返回两个参数的和


class MyThread(threading.Thread):
    def __init__(self, func, args):
        threading.Thread.__init__(self)  # 调用父类的构造函数
        self.func = func
        self.args = args
        self.result = None

    def run(self):
        self.result = apply(self.func, self.args)  # 调用函数的结果作为一个属性


def main():
    print '开始主线程：', ctime()
    threads = []  # 一个用于储存线程对象的列表
    n_loops = range(len(loops))
    for i in n_loops:
        t = MyThread(func=loop, args=(i, loops[i]))  # 使用我们自己的类来新建对象
        threads.append(t)  # 将新创建的对象放到一个列表中
    for i in n_loops:
        threads[i].start()  # 每次循环运行一个线程

    for i in n_loops:
        threads[i].join()  # 等待子线程的完成

    for i in n_loops:
        print '执行结果为：', threads[i].result  # 打印该对象的属性

    print '主线程完成：', ctime()


if __name__ == '__main__':
    main()
```
执行结果为:
```
开始主线程： Sat Jun 18 20:11:51 2016
执行结果为： 4
执行结果为： 3
主线程完成： Sat Jun 18 20:11:51 2016

Process finished with exit code 0
```

### Python进程池
实际在项目中，我们可能会碰到这样的问题，有一个MySQL的表分库分表了，总数大概有1000张，我们想看看每天的数据量有多大，由于这个是线上的库，我们不能开太多的连接，如果把数据库搞挂了不好，但是一个一个去算又太慢，所以需要控制连接的数量，太大太小都不好，这个时候可以使用进程池，控制并发的数量。

#### 进程池(非阻塞)
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'anonymous'

from multiprocessing import freeze_support, Pool
import time


def Foo(i):
    time.sleep(2)
    print 'time start:%s' % time.strftime('%Y-%m-%d %H:%M:%S')
    return i + 100


def Bar(arg):
    print 'time  done:%s %s' % (time.strftime('%Y-%m-%d %H:%M:%S'), arg)


if __name__ == '__main__':
    freeze_support()
    pool = Pool(3)  # 线程池中的同时执行的进程数为3

    for i in range(4):
        pool.apply_async(func=Foo, args=(i,), callback=Bar)  # 线程池中的同时执行的进程数为3，当一个进程执行完毕后，如果还有新进程等待执行，则会将其添加进去

    print('end')
    pool.close()
    pool.join()  # 调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
```
输出结果为:
```
end
time start:2016-06-19 18:08:12
time  done:2016-06-19 18:08:12 100
time start:2016-06-19 18:08:12
time  done:2016-06-19 18:08:12 101
time start:2016-06-19 18:08:12
time  done:2016-06-19 18:08:12 102
time start:2016-06-19 18:08:14
time  done:2016-06-19 18:08:14 103
```
进程池的大小为3，很明显可以看到，前三个函数的开始时间都是一样的，并且都是不等`Foo`函数执行完就掉用`Bar`函数
* `apply_async(func[, args[, kwds[, callback]]])`它是**非阻塞**，`apply(func[, args[, kwds]])`是**阻塞**的.
* `close()`关闭pool,使其不再接受新的任务
* `join()` 主进程阻塞，等待子进程退出，必须在`close()`和`terminate()`之后调用

#### 进程池(阻塞)
和上面的代码差不多，只是把对应部分改一下:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

__author__ = 'anonymous'

from multiprocessing import freeze_support, Pool
import time


def Foo(i):
    time.sleep(2)
    print 'time start:%s' % time.strftime('%Y-%m-%d %H:%M:%S')
    return i + 100


def Bar(arg):
    print 'time  done:%s %s' % (time.strftime('%Y-%m-%d %H:%M:%S'), arg)


if __name__ == '__main__':
    freeze_support()
    pool = Pool(3)  # 线程池中的同时执行的进程数为3

    for i in range(4):
        pool.apply(func=Foo, args=(i,))

    print('end')
    pool.close()
    pool.join()  # 调用join之前，先调用close函数，否则会出错。执行完close后不会有新的进程加入到pool,join函数等待所有子进程结束
```
执行结果如下:
```
time start:2016-06-19 18:14:00
time start:2016-06-19 18:14:02
time start:2016-06-19 18:14:04
time start:2016-06-19 18:14:06
end
```

#### 进程池(关注返回结果)
多半情况下我们还是要关注函数的执行返回结果，并不能完全想像上面那样阻塞运行或者非阻塞等函未执行完就调用回调函数:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import multiprocess

__author__ = 'anonymous'
import time


def func(msg):
    print 'time start:%s' % time.strftime('%Y-%m-%d %H:%M:%S')
    time.sleep(2)
    print 'time   end:%s' % time.strftime('%Y-%m-%d %H:%M:%S')
    return 'done' + msg

if __name__ == '__main__':
    pool = multiprocess.Pool(2)
    result = []

    for i in range(3):
        msg = 'hello %s' % i
        result.append(pool.apply_async(func=func, args=(msg,)))

    pool.close()
    pool.join()

    for res in result:
        print '***: %s' % res.get()

    print 'end'
```
执行看一下返回结果是啥
```
time start:2016-06-19 18:43:22
time start:2016-06-19 18:43:22
time   end:2016-06-19 18:43:24
time start:2016-06-19 18:43:24
time   end:2016-06-19 18:43:24
time   end:2016-06-19 18:43:26
***: done hello 0
***: done hello 1
***: done hello 2
end
```
可以看到，我们可以使用`get()`方法获取被调用函数的返回值


#### multiprocessing
还有一种方式，可以使用`pool.map()`，以我们最开始的例子为例，我们要查询1000张表，可以把参数放在一个可迭代对象里，使用pool.map()找依次多个处理
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import multiprocessing

__author__ = 'anonymous'


def m1(x):
    return x * x


if __name__ == '__main__':
    pool = multiprocessing.Pool(multiprocessing.cpu_count())
    i_list = range(8)
    result = pool.map(m1, i_list)

    print sum(result)
```
执行结果：
```
140
```
