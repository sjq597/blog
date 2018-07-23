title: Linux运维常用命令备忘录
date: 2018-05-24 14:47:04
tags: [Linux,Shell]
categories: Linux使用
---
作为一个后端开发,尤其是数据开发,我们很多的服务都是以进程的方式运行在后台.例如Jstorm/Kafka/ElasticSearch等等,线上报警处理也是一个必备技能了,更多的可能是一些磁盘,内存,CPU指标之类的,有些命令不常用可能会忘记,做个记录方便查找.

### 磁盘
这个应该是最常见的,磁盘报警是家常便饭了.

#### df -lh

查看系统磁盘占用情况,一般磁盘报警了可以先用这个命令看下大概是哪个盘出问题了,找到占用比较大的磁盘有时候需要配合其他命令:
```
# 到具体的目录下执行
# 1.快速方法
du -sh
# 2.推荐方法(-x 可以过滤掉和一开始文件系统不一样的文件/文件系统)
du -h --max-depth=1
```
这里不推荐第一种方式是因为效率问题，如果碰上是根`/`目录满了,基本上统计不出来,如果为了快，想配合排序定位问题,还可以配合sort函数使用
```
# 倒序排(一般占用大基本上就是看G,很少会到T,毕竟磁盘没那么大,也有例外)
du -h --max-depth=1 | grep [TG] |sort -nr | head
```

#### find

这个命令紧跟在上一个命令后面,就是因为很多时候我们需要批量删除满足某些条件的文件,但这些文件可能并不是简单的在一个文件夹下面,类型一样.可能分散在某个目录下面的多级目录,并且类型也很多:
```
# 根据文件类型来删,比如日志文件 *.log
find . -name '*.xxx' -delete
# or
find . -name '*.xxx' -exec sudo rm -f {} \;
# 根据时间 -mtime:内容时间 -ctime:状态修改
find . -mtime +10 -a -ctime +10 -delete
# 根据文件大小 >100k <500M
find . -size +100k +size -500M -delete
```
**NOTE:**find命令`-a`:与,`-or`:或,`not`:否.另外使用删除请慎重,可以先用`-print`打印一下看看是否满足要求,不然删了不该删的后果可能很严重.

#### swap
有时候磁盘满了可能是swap设置的太大,占用的swap又无法释放(内存就算有很大空闲,swap已经使用的可能也不会释放),导致磁盘一直报警,处理这个要稍微麻烦一点儿.首先看下磁盘占用比较大的几个程序,或者说看下有没有比较重要的服务占用着缓存,有的话重启下,先把重要的程序占用的swap释放掉,然后关闭缓存,开启缓存:
看一次实际线上问题:
```
$ free -m
              total        used        free      shared  buff/cache   available
Mem:          11855        2366        9146         157         341        9144
Swap:          4095        2600        1495
```
可以看到内存有很多空闲,但是swap占用了2.6G无法释放，然后统计下是哪些进程在占用着swap:
```
$ for i in $(sudo ls /proc | grep "^[0-9]" | awk '$0>100'); do sudo awk '/Swap:/{a=a+$2}END{print '"$i"',a/1024"M"}' /proc/$i/smaps;done| sort -k2nr | head
awk: fatal: cannot open file `/proc/10612/smaps' for reading (No such file or directory)
awk: fatal: cannot open file `/proc/10613/smaps' for reading (No such file or directory)
awk: fatal: cannot open file `/proc/10614/smaps' for reading (No such file or directory)
awk: fatal: cannot open file `/proc/10615/smaps' for reading (No such file or directory)
awk: fatal: cannot open file `/proc/10616/smaps' for reading (No such file or directory)
20211 466.664M
23994 215.438M
9339 208.672M
9334 167.766M
9340 152.559M
9338 132.375M
20293 88.9883M
9342 86.918M
9335 84.4492M
4323 76.8984M
```
可以看到进程号和对应的缓存占用大小(如果不想看到错误信息,可以grep -v "No such"过滤掉),看下具体的有没有比较重要的,如果想看具体的程序,后面可以用awk切割出pid,然后配合`ps -p xxx`看下具体的进程是不是重要的，手动重启下.然后就是比较关键的两个操作:
```
# 关闭所有缓存
swapoff -a
# 开启所有缓存
swapon -a
```
这样swap就全部被清空了.

#### lsof
这个命令可能一般人用的比较少,这个主要是看文件句柄的,有时候一个文件很大，我们直接删了,但是会发现磁盘空间并没有释放.这个时候一般就是还有进程占用着这个文件句柄,所以你可能在目录里面看没有什么文件,但是磁盘就是被占用了.可以这么排查:
```
# 查看文件为删除状态的文件
lsof | grep deleted
```
这个命令里面就可以看到是哪个进程持有这个文件的句柄,比如像tomcat,有时候日志打太多了,但是你直接删掉,磁盘根本不会减少,这个时候你可以重启下tomcat,就会发现文件占用的空间又回来了.

### cpu/mem
这两个一般都是用top命令来看,所以放在一起说了.一般top之后可以看到进程的实时信息,top有一些常用参数命令
```
c top显示带具体进程信息
M 按mem占用排序
P 按cpu占用排序
# 具体线程信息
top -H -p {pid}
```
具体的进程问题，可能得用具体的方法,这里就不做展开了.
