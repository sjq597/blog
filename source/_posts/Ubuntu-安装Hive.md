title: Ubuntu 安装Hive
date: 2016-02-27 12:30:27
tags: [Linux,Hive]
categories: 开发环境
---
在Ubuntu上搭建Hive环境
### 准备工作
OS：Ubuntu 14.04 64bit
Jdk: 1.7.40
Hive: 2.0.0
去Apache官网下载:[apache-hive-2.0.0-bin.tar.gz](http://hive.apache.org/downloads.html)

### 安装
```
sudo tar -zxvf apache-hive-2.0.0-bin.tar.gz -C /usr/qunar
```
然后需要设置环境变量，修改`~/.bashrc`文件，添加如下内容：
```bash
export HIVE_HOME=/usr/qunar/apache-hive-2.0.0-bin
export PATH=${HIVE_HOME}/bin:$PATH
```

### 元数据
所有的Hive客户端都需要一个metastoreservice(元数据服务),Hive中使用的数据是集群上的，数据对应的表需要一个地方存放，也就是Hive里面的表结构和数据是分离的，需要一个关系型数据库来存储这些信息。如果你没有装数据库也没有关系，Hive默认内置了一个`Derby SQL`服务器。不过有个限制：这个内置的服务器是单进程的，也就意味着最多只能有一个`Hive CLI`实例，我主要是为了在本地单机调试，所以这个问题不大，集群的话，建议使用`MySQL`来作为元数据信息数据库。

