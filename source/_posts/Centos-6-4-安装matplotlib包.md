title: Centos 6.4 安装matplotlib包
date: 2016-02-04 19:20:11
tags: [Linux,Python]
categories: 开发环境
---
官方推荐的安装方法：
* Debian / Ubuntu : sudo apt-get install python-matplotlib
* Fedora / Redhat : sudo yum install python-matplotlib

判断是否安装成功的方法，在python交互式环境下:
```python
import matplotlib
```
如果没有报错，则说明安装成功了，如果这样还不成功，可以通过pip来安装，如何安装pip,
参见文章[服务器安装pip工具](../../../2016/01/14/服务器安装pip工具),具体的命令就是：
```bash
$ sudo wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py
$ sudo python ez_setup.py --insecure
$ sudo wget --no-check-certificate https://pypi.python.org/packages/source/p/pip/pip-7.1.2.tar.gz
$ sudo tar -xvf pip-7.1.2.tar.gz
$ cd pip-7.1.2
$ sudo python setup.py install
```

# 使用方法
```
$ sudo pip install module_name
```

如果使用pip安装不成功，可以通过编译`matplotlib`源码来手动安装，
```bash
$ sudo wget --no-check-certificate https://pypi.python.org/packages/source/m/matplotlib/matplotlib-1.5.1.tar.gz 
$ sudo tar -xvf matplotlib-1.5.1.tar.gz
$ cd matplotlib-1.5.1
$ python setup.py build
$ python setup.py install
```
但是我在执行`python setup.py build`的时候，,提示缺少依赖，无法构建，缺少依赖,根据`build`打印出的信息，把必须的依赖里为`no`的包都装上，基本就是`libpng`,`freetype`这两个，具体如何安装，可以先执行：
```bash
$ yum list | grep libpng
```
从打印出来的结果里挑一个适合你系统的版本，记住，如果正统的版本安装了还不行，你还需要安装`libpng-devel`这种类似的名字的包，是情况而定。
然后所有的依赖安装好了之后，终于`build`成功了，但是我在执行`install`的时候，还是出错了，`gcc-plus`错误，出现这种错误是由于没有装`g++`的库，安装方法：
```bash
$ yum list | grep gcc
gcc.x86_64                             4.4.7-3.el6                 @base        
gcc-c++.x86_64                         4.4.7-3.el6                 @base        
libgcc.x86_64                          4.4.7-3.el6                 @anaconda-CentOS-201303020151.x86_64/6.4
compat-gcc-34.x86_64                   3.4.6-19.el6                base         
compat-gcc-34-c++.x86_64               3.4.6-19.el6                base         
compat-gcc-34-g77.x86_64               3.4.6-19.el6                base         
gcc-gfortran.x86_64                    4.4.7-3.el6                 base         
gcc-gnat.x86_64                        4.4.7-3.el6                 base         
gcc-java.x86_64                        4.4.7-3.el6                 base         
gcc-objc.x86_64                        4.4.7-3.el6                 base         
gcc-objc++.x86_64                      4.4.7-3.el6                 base         
libgcc.i686                            4.4.7-3.el6                 base         
mingw32-gcc.x86_64                     4.4.6-4.el6                 base         
mingw32-gcc-c++.x86_64                 4.4.6-4.el6                 base         
mingw32-gcc-gfortran.x86_64            4.4.6-4.el6                 base         
mingw32-gcc-objc.x86_64                4.4.6-4.el6                 base         
mingw32-gcc-objc++.x86_64              4.4.6-4.el6                 base     
```
可以看到有很多包，你可以都装上，但是此处我们只需要`gcc-c++.x86_64`，装完之后，然后重新执行`python setup.py install`安装即可。然后再进入`python`交互式环境就可以引用`matplotlib`包了。
