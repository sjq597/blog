title: Ubuntu 14.04 升级gcc
date: 2015-11-02 17:46:38
tags: [Linux]
categories: 开发环境
---
前段时间安装mysqldb时碰到gcc的编译错误问题,后来看到了网上有人说需要更新gcc编译器,在Ubuntu 14.04下更新gcc也很简单

### Ubuntu 更新gcc编译器

* 首先更新源

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-4.9 g++-4.9
```

* 保留旧的gcc

```bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 10  
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 20
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.9 20
```

* 查看配置

```bash
sudo update-alternatives --config gcc
```
 如下结果:
```
  选择       路径        优先级  状态
------------------------------------------------------------
* 0     /usr/bin/gcc-4.9   20   自动模式
  1     /usr/bin/gcc-4.8   10   手动模式
  2     /usr/bin/gcc-4.9   20   手动模式
```
 可以手动选择使用哪个版本,由于我们把`gcc-4.9`设置的优先级最高,所以默认就是使用的高版本的,对于`g++`同样.
