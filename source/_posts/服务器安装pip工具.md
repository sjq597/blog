title: 服务器安装pip工具
date: 2016-01-14 15:03:17
tags: [Python,Linux]
categories: 开发环境
---

系统版本:CentOS release 6.4 (Final)
Python版本:2.6.6

### 安装步骤
1. 先下载安装setuptools
```bash
sudo wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py
sudo python ez_setup.py --insecure
```
2. 然后下载安装pip
```bash
sudo wget --no-check-certificate https://pypi.python.org/packages/source/p/pip/pip-7.1.2.tar.gz
sudo tar -xvf pip-7.1.2.tar.gz
cd pip-7.1.2
sudo python setup.py install
```

3. pip使用方法
```bash
sudo pip install module_name
```

### 安装python
有时候用pip安装,会提示:
```
SystemError: Cannot compile 'Python.h'. Perhaps you need to install python-dev|python-devel
```
这时候不能像Ubuntu上安装`python-dev`,在CentOS上,名字变了,像如下安装:
```
sudo yum install python-devel
```

还需要升级python2.6.6至2.7.x
查看python2.7安装位置，假设在`/usr/bin/q-python2.7`,建立软连接
```bash
# 加-b 参数是为了覆盖之前的软链接
sudo ln -sb /usr/bin/q-python2.7 /usr/bin/python
```
这会导致yum无法使用，修复办法,编辑`/usr/bin/yum`
将头部的python改为python2.6

有时候由于国内某些你懂的原因，安装包经常连不上，需要设置pip的国内镜像CentOS修改`~/.pip/pip.conf`文件，内容如下：
```
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple/
```
对应的如果是Windows,则需要在对应用户目录下修改，例如`C:\Users\zhangsan\pip\pip.ini`，内容如上。

指定url安装包可以这样, 以`Flask`为例:
```
pip install -i http://mirrors.aliyun.com/pypi/simple/ Flask
```

项目依赖打包以及环境快速恢复:
```
# 依赖导出
pip freeze > requirements.txt

# 安装文件中所有依赖
pip install -r requirements.txt
```
