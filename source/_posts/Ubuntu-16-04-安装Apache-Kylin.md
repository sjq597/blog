title: Ubuntu 16.04 安装Apache Kylin
date: 2016-10-23 15:57:27
tags: [Hadoop,大数据,Linux]
categories: 开发环境
---
最近貌似用Kylin来搭建OLAP数据分析已经成为了大多数公司采取的做法了，这个东西搭建起来还有些=麻烦,因为它需要你先配置好Hadoop,Hive,Hbase这三个东西，任何一个配置的不对的话可能会出问题，下面介绍一下我的安装和配置过程

### 环境准备
主要是依赖的环境的一些版本
```
OS: Ubuntu 16.04 64bit
Java: 1.7.0_40
Hive: 2.1.0
Hadoop: 2.6.4
Hbase: 0.98.22-hadoop2
Kylin: 1.5.4.1
```

**备注:**这里Hbase并没有用官方的1.x版本，而是采用了Kylin官方建议的0.98版本，因为我第一次安装的时候由于hbase的问题而失败了，所以就换了一个低版本的，你在安装的时候也可以使用最新版，但是如果安装不成功，可以考虑采纳官方建议的组合版本

### 安装步骤
Hadoop,Hive,Hbase的安装这里就不讲了，可以参考我以前的文章，我采用的都是本地为分布式运行模式,去[Kylin官方下载](http://kylin.apache.org/download/)最新版本:
```
sudo tar -xvf apache-kylin-1.5.4.1-bin.tar.gz -C /usr/dev
```
解压好了，然后就是配置环境变量了，注意我都配置在`/etc/profile`这个文件里面了，主要是有些时候环境变量读取不到，所以干脆来个最狠的，直接全局配置了:
```
### 大数据相关
## Hbase
export HBASE_HOME=/usr/dev/hbase-1.2.3
export HBASE_CONF_DIR=${HBASE_HOME}/conf

## Hadoop
export HADOOP_HOME=/usr/dev/hadoop-2.6.4
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop

## Hive
export HIVE_HOME=/usr/dev/apache-hive-2.1.0-bin
export HCAT_HOME=${HIVE_HOME}/hcatalog
export HIVE_CONF=${HIVE_HOME}/conf

## Kylin
export KYLIN_HOME=/usr/dev/apache-kylin-1.5.4.1-bin
```
上面的目录如果不一致，改成你自己对应的目录,然后`source /etc/profile`使配置生效。

### 配置Kylin
配置Kylin使用Hive的数据库名
```
cd ${KYLIN_HOME}/conf
vim kylin.properties
kylin.job.hive.database.for.intermediatetable=kylin_middle
```
然后记得要手动在Hive中创建数据库`kylin_middle`.

然后创建一个目录并赋给当前用户权限,假设当前用户为`anonymous`:
```
hadoop fs -mkdir /kylin
hadoop fs -chown -R anonymous:anonymous /kylin
```
然后检测一下Kylin的环境变量:
```
cd ${KYLIN_HOME}/bin
./check-env.sh
```
如果你的系统也报错:
```
KYLIN_HOME is set to /usr/dev/apache-kylin-1.5.4.1-bin
cat：无效选项 -- 1
Try 'cat --help' for more information.
-mkdir: Not enough arguments: expected 1 but got 0
Usage: hadoop fs [generic options] -mkdir [-p] <path> ...
failed to create , Please make sure the user has right to access
```
那可能也是由于sh调用的问题，需要把所有sh脚本里面的`sh`替换成`bash`，我这里用`sed`命令来替换:
```
sed -i 's/`sh /`bash /g' *.sh
```
这样就把所有的替换了，并且替换结果也直接写回文件了.然后运行环境检测脚本
```
./check-env.sh
```
如果没报什么错误，就可以正式运行了，记得先把hadoop hbase服务启动
然后再启动kylin
```
./kylin.sh start
```
然后访问[127.0.0.1:7070/kylin](127.0.0.1:7070/kylin)
第一次会有点儿慢，需要在hbase中创建表，稍微等大概10s就会弹出登陆界面
ADMIN/KYLIN登陆就可以了

### 测试运行
官方也提供了官方测试例子，如果你可以顺利登入系统，那么在`${KYLIN_HOME}/bin`下面有个测试脚本`sample.sh`,运行这个脚本
```
cd ${KYLIN_HOME}/bin
./sample.sh
./kylin.sh stop
./kylin.sh start
```
一次执行上面三个命令，运行成功之后重新进入系统或者刷新即可，会有一个`learn_kylin`工程，你只需要在`Model`里面`Actions`里面`Build`这个Cube即可，然后在Monitor里面可以看编译的进度，只有到100%才说明你编译成功，双击可以查看每一步的编译具体步骤，可以查看具体的日志，如果你编译失败了，可以查看具体的日志，直到编译成功为止，然后在Insight里面可以写sql查询:
```
select part_dt,sum(price) from kylin_sales group by part_dt;
```
可以看看这句，这个放到hive里面起码要30s,但是这个在Kylin里面基本就秒出结果了,Kylin提供的是sql接口,下一节讲一讲怎么集成上层的多维分析工具。
