title: Ubuntu 16.04 Hbase 本地安装配置
date: 2016-10-09 18:15:04
tags: [Linux,Hbase,大数据]
categories: 开发环境
---
单机安装OLAP Kylin多维数据分析工具,Kylin依赖`Hadoop`,`Hive`,`Hbase`这三个，之前安装过前面两个，所以只用装一个Hbase就可以了，也是单机/伪分布式这种模式

### 系统环境
```
OS: Ubuntu 16.04 LTS 64bit
JDK: 1.7.0_40
Hadoop:hadoop-2.6.4
➜  Blog git:(master) ✗ whoami 
anonymous
```

### 安装步骤
首先去官网[Kylin下载地址](http://apache.fayea.com/hbase/stable/)，可以下载编译好的`hbase-1.2.3-bin.tar.gz`,也可以下载源码自己编译，我下载源码版`hbase-1.2.3-src.tar.gz`自己编译的,其他的应该都一样,下载之后解压编译:
```
tar -xvf hbase-1.2.3-src.tar.gz -C /usr/dev
cd /user/dev/hbase-1.2.3
mvn package -Dmaven.test.skip=true	# 跳过单元测试，不然测试时间会非常长
```

配置hbase,当前目录默认是`/usr/dev/hbase-1.2.3`.

#### 单机版
先编辑`conf/hbase-site.xml`
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    //这里设置让HBase存储文件的地方
    <value>file:///tmp/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    //这里设置让HBase存储内建zookeeper文件的地方
    <value>/tmp/zookeeper</value>
  </property>
</configuration>
```
然后启动hbase:
```
./bin/start-hbase.sh
```
看看是否成功启动:
```
➜  hbase-1.2.3 jps
22228 Jps
2738 Main
21914 HMaster
```
如果有`HMaster`就说明启动成功了，然后可以访问web界面了:
```
http://127.0.0.1:16010/
```
**注:**不同的版本可能端口号不一致，具体的如果访问不了，网上查一下看看你的版本是那个端口号.

#### 伪分布式
先把hbase停掉`./bin/stop-hbase.sh`,然后修改`conf/hbase-env.sh`文件:
```
HBASE_MANAGE_XK=true
```
然后重新修改`conf/hbase-site.xml`文件:
```
<configuration>
  <property>
    <name>hbase.rootdir</name>
    //这里设置让HBase存储文件的地方
    <value>hdfs://localhost:9000/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    //这里设置让HBase存储内建zookeeper文件的地方
    <value>/tmp/zookeeper</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
</configuration>
```
**注:**上面的`hbase.rootdir`配置要和`/usr/dev/hadoop-2.6.4/etc/hadoop/core-site.xml`一致:
```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
</property>
```
