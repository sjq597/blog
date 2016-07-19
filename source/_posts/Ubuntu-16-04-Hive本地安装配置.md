title: Ubuntu 16.04 Hive本地安装配置
date: 2016-07-19 13:18:47
tags: [Linux,Hive,Hadoop,大数据]
categories: 开发环境
---
线上集群测试太慢，有时候需要在本地跑一些计算或者测试某个逻辑，主要做调试用，所以在本地也装一个Hive测试用

### 准备工作
开发环境为:
```
OS: Ubuntu 16.04 LTS 64bit
JDK: 1.7.0_40
Mysql: 5.7.12
Hadoop:hadoop-2.6.4.tar.gz
Hive:apache-hive-2.1.0-bin.tar.gz
```

简单的介绍一下安装之前需要做的一些准备工作,Jdk不用介绍了，具体的安装过程可以自己去查阅，安装MySQL也比较简单:
```
sudo apt install mysql-server -y
```

### 安装步骤
具体的安装步骤可能有些多，具体过程如下:

#### 创建Hadoop用户
为了使用方便，和日常使用分开来，我们创建一个专属用户:
```
sudo useradd -m hadoop -s /bin/bash
sudo passwd hadoop
sudo adduser hadoop sudo
```
上面的命令就是创建了一个新用户，并且设置新用户的密码，连输两次，简单起见密码就设置为`hadoop`,然后把`hadoop`用户加到管理员组。
如果想删除这个用户可以这样:
```
sudo userdel hadoop	# 删除用户
sudo rm -rf /home/hadoop	# 删除用户目录
```
