title: python 连接hive
date: 2017-06-08 16:44:40
tags: [Hive, Python]
categories: [Python笔记]
---
首先不要参考官网的例子,pyhs2的作者已经不再更新了，所以你如果用官网的例子，多半会由于版本问题而搞出各种莫名奇妙的问题，这里推荐一下使用dropbox推出的PyHive,[PyHive Github地址](https://github.com/dropbox/PyHive)

官网配置貌似不大对，所以可以直接参考我的这个，起码我是本地测试通过的,环境:
```
OS: Ubuntu 16.04 64bit
JDK: 1.7.40
Hadoop: 2.7.3
Hive: 2.1.0
Python: 2.7.12
```

### 开启hiveserver2
这个地方官方说的不是很好，导致后面按官网配置可能依然导致无法连接hiveserver2，下面说下我的步骤,首先`hive-site.xml`里面的默认配置都不用改，除非你的端口被占用或者你想指定端口.
重点是需要改Hadoop的配置文件`${HADOOP_HOME}/etc/hadoop/core-site.xml`文件,我们假设你需要用`dev`这个用户来访问hiveServer2`服务,那么需要在`core-site.xml`文件里面加上下面的配置:
```
  <property>
    <name>hadoop.proxyuser.dev.groups</name>
  <value>*</value>
  </property>

  <property>
   <name>hadoop.proxyuser.dev.hosts</name>
   <value>*</value>
  </property>
```
如果你这里不写这个，那么很可能也连接的时候会报错，提示你没有权限,然后其他的默认就行，不过确保你的`hive-site.xml`文件里面的配置是下面这样:
```
<property>
 <name>hive.server2.enable.doAs</name>
 <value>true</value>
</property>
```
然后可以开启hiveServer2服务
```
bash /workspace/dev/apache-hive-2.1.0-bin/bin/hiveserver2
```

### 常见错误
当配置好hiveserver之后,执行简单的select可能没有啥问题,但是像有一些稍微复杂的sql可能会失败.

* hiveserver 执行简单的count报错

```
Exception in thread "LeaseRenewer:anonymous@localhost:9000"
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "LeaseRenewer:anonymous@localhost:9000"
Exception in thread "HiveServer2-Handler-Pool: Thread-49"
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "HiveServer2-Handler-Pool: Thread-49"
```
解决方案:修改hadoop配置文件`core-site.xml`

```
<property>
    <name>mapreduce.map.memory.mb</name>
    <value>1024</value>
  </property>

  <property>
    <name>mapred.child.map.java.opts</name>
    <value>-Xmx900M</value>
  </property>

  <property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx900M</value>
  </property>

  <property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>2048</value>
  </property>

  <property>
    <name>mapred.child.reduce.java.opts</name>
    <value>-Xmx1900M</value>
  </property>

  <property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx1900M</value>
  </property>
```
简单来说就是内存溢出,所以更改下默认的map,reduce阶段的一些内存设置.
