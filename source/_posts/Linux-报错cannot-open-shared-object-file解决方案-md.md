title: Linux 报错cannot open shared object file解决方案.md
date: 2016-07-08 15:47:34
tags: Linux
categories: Linux使用
---
在使用Ubuntu安装软件之后，启动软件有时候会出现软件一直在闪，但是最后无法进入软件启动界面，这种时候，需要在终端中运行，这个时候一般会报错：
```
error while loading shared libraries: libgstreamer-0.10.so.0: cannot open shared object file: No such file or directory
```
借助`apt-file`命令可以查找缺少包所依赖的linux文件, 然后用`apt-get install`安装所对应的文件

我的系统是`Ubuntu 16.04`，下面的操作都是在此系统上进行的操作:

* 安装`apt-file`:

```
sudo apt install apt-file -y
sudo apt-file update
```

* 查找缺失库

最开始的报错信息里面有关键字`libgstreamer-0.10.so.0`，这个时候就需要查找`libgstreamer-0.10.so.0`所对应的库:
```
➜  Blog git:(master) ✗ apt-file search libgstreamer-0.10.so.0
libgstreamer0.10-0: /usr/lib/x86_64-linux-gnu/libgstreamer-0.10.so.0
libgstreamer0.10-0: /usr/lib/x86_64-linux-gnu/libgstreamer-0.10.so.0.30.0
```
通过上面的命令查到对应的库是`libgstreamer0.10-0`,所以直接安装缺失的库就行:
```
sudo apt install libgstreamer0.10-0 -y
```
然后再重新运行程序，正常情况下应该没啥问题了，如果提示缺失其他的文件，同样的方式，直到不再报错为止.
