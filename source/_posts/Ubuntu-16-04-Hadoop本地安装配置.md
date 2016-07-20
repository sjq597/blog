title: Ubuntu 16.04 Hadoop本地安装配置
date: 2016-07-19 13:18:47
tags: [Linux,Hadoop,大数据]
categories: 开发环境
---
线上集群测试太慢，有时候需要在本地跑一些计算或者测试某个逻辑，主要做调试用，所以在本地也装一个Hive测试用,但是装Hive需要先安装Hadoop.

### 准备工作
开发环境为:
```
OS: Ubuntu 16.04 LTS 64bit
JDK: 1.7.0_40
Hadoop:hadoop-2.6.4.tar.gz
```

### 安装步骤
具体的安装步骤可能有些多，具体过程如下:

#### 创建Hadoop用户
为了使用方便，和日常使用分开来，我们创建一个专属用户:
```
➜  sudo useradd -m hadoop -s /bin/bash
➜  sudo passwd hadoop
➜  sudo adduser hadoop sudo
```
上面的命令就是创建了一个新用户，并且设置新用户的密码，连输两次，简单起见密码就设置为`hadoop`,然后把`hadoop`用户加到管理员组。
如果想删除这个用户可以这样:
```
➜  sudo userdel hadoop	# 删除用户
➜  sudo rm -rf /home/hadoop	# 删除用户目录
```

#### ssh登陆配置
然后点右上角的切换用户登陆，以hadoop用户登陆之后，执行下面的语句:
```
➜  sudo apt update
➜  sudo apt install openssh-server -y
➜  ssh localhost
```
输入密码之后可以登陆则没有问题，然后使用`exit`命令注销当前用户,如果第一步报错:
```
忽略:1 http://dl.google.com/linux/chrome/deb stable InRelease
获取:2 http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease [3,192 B]
命中:3 http://cn.archive.ubuntu.com/ubuntu xenial InRelease
错误:2 http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease
  由于没有公钥，无法验证下列签名： NO_PUBKEY 8D5A09DC9B929006
命中:4 http://cn.archive.ubuntu.com/ubuntu xenial-updates InRelease
获取:5 http://cn.archive.ubuntu.com/ubuntu xenial-backports InRelease [92.2 kB]
命中:6 http://security.ubuntu.com/ubuntu xenial-security InRelease
命中:7 http://dl.google.com/linux/chrome/deb stable Release
正在读取软件包列表... 完成
W: GPG 错误：http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease: 由于没有公钥>，无法验证下列签名： NO_PUBKEY 8D5A09DC9B929006
E: 仓库 “http://archive.ubuntukylin.com:10006/ubuntukylin xenial InRelease” 没有数字签名。
N: 无法安全地用该源进行更新，所以默认禁用该源。
N: 参见 apt-secure(8) 手册以了解仓库创建和用户配置方面的细节。
```
需要按如下方案解决,手动添加上面报错的签名`8D5A09DC9B929006`:
```
➜  sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8D5A09DC9B929006
```

#### 生成密钥
每次登陆都需要输密码很不方便，设置免密码登陆,记得注销当前用户`exit`:
```
➜  cd ~/.ssh
➜  sh-keygen -t rsa
➜  cat id_rsa.pub >> authorized_keys
```
**注意:**生成密钥的时候一路回车，不要输入任何东西,如果进行到这一步，我们就可以切回原来的系统了，然后使用:
```
ssh hadoop@localhost
```
因为Linux本来就是一个支持多用户登陆的操作系统，配置好`ssh serveer`之后就可以完全在我们平时用的系统里通过`ssh`以`hadoop`用户登陆进行操作，下面的操作都是以`hadoop`用户登陆进行的操作。

#### 安装JDK
由于原来的用户已经装过了，这里只需要把之前用的的`JDK`路径设置一下就行了，编辑`~/.bashrc`文件:
```
export JAVA_HOME=/usr/dev/jdk1.7.0_40
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
然后更新一下文件:
```
➜  source ~/.bashrc
```
测试JDK是否配置正确:
```
➜  java -version
java version "1.7.0_40"
Java(TM) SE Runtime Environment (build 1.7.0_40-b43)
Java HotSpot(TM) 64-Bit Server VM (build 24.0-b56, mixed mode)
```

#### 安装Hadoop
确认JDK安装正确之后，下面就可以来安装Hadoop了,进入到压缩包所在目录:
```
➜  sudo tar -xvf hadoop-2.6.4.tar.gz -C /usr/dev/
➜  sudo chown -R hadoop /usr/dev/hadoop-2.6.4/
```
看看是否正确安装了:
```
➜  cd /usr/dev/hadoop-2.6.4
➜  ./bin/hadoop version
Hadoop 2.6.4
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 5082c73637530b0b7e115f9625ed7fac69f937e6
Compiled by jenkins on 2016-02-12T09:45Z
Compiled with protoc 2.5.0
From source with checksum 8dee2286ecdbbbc930a6c87b65cbc010
This command was run using /usr/dev/hadoop-2.6.4/share/hadoop/common/hadoop-common-2.6.4.jar
```
可以识别到版本就说明没啥问题了。
**注意:**后面的操作如果没有特殊指明，都是在hadoop的安装目录`/usr/dev/hadoop-2.6.4`下进行的操作

### 单机配置
单机配置比较简单，我们以一个简单的单词统计来看看怎么配置:
```
➜  cd /usr/dev/hadoop-2.6.4
➜  mkdir input
➜  cp etc/hadoop/*.xml input/
➜  ./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.4.jar grep input/ output 'dfs[a-z.]+'
➜  cat output/*
1       dfsadmin
```

### 伪分布式
伪分布式需要修改两个文件`etc/hadoop/core-site.xml`以及`etc/hadoop/hdfs-site.xml`,修改对应位置，改为如下内容:

* etc/hadoop/core-site.xml

```
<configuration>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/dev/hadoop-2.6.4/tmp</value>
    <description>Abase for other temporary directories.</description>
  </property>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

* etc/hadoop/hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/dev/hadoop-2.6.4/tmp/dfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/dev/hadoop-2.6.4/tmp/dfs/data</value>
  </property>
</configuration>
```

格式化节点:
```
➜  ./bin/hdfs namenode -format
............省略一堆调试信息............
16/07/19 16:23:54 INFO common.Storage: Storage directory /usr/dev/hadoop-2.6.4/tmp/dfs/name has been successfully formatted.
16/07/19 16:23:54 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
16/07/19 16:23:54 INFO util.ExitUtil: Exiting with status 0
16/07/19 16:23:54 INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at junqiang.shen/127.0.1.1
************************************************************/
```
**注意:**如果消息最后不是`been successfully formatted.`以及返回值不是`Exiting with status 0`则说明执行失败。

开启`NameNode`和`DataNode`守护进程:
```
➜  ./sbin/start-dfs.sh
Starting namenodes on [localhost]
localhost: Error: JAVA_HOME is not set and could not be found.
localhost: Error: JAVA_HOME is not set and could not be found.
Starting secondary namenodes [0.0.0.0]
0.0.0.0: Error: JAVA_HOME is not set and could not be found.
```
出现这个错误，并不是由于jdk没设置好，不然前面的也不能运行，只好手动修改文件`etc/hadoop/hadoop-env.sh`，原来的地方是`export JAVA_HOME=${JAVA_HOME}`改成`~/.bashrc`里面的内容:
```
export JAVA_HOME=/usr/dev/jdk1.7.0_40
```
然后再试一下:
```
➜  ./sbin/start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/dev/hadoop-2.6.4/logs/hadoop-hadoop-namenode-junqiang.out
localhost: starting datanode, logging to /usr/dev/hadoop-2.6.4/logs/hadoop-hadoop-datanode-junqiang.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/dev/hadoop-2.6.4/logs/hadoop-hadoop-secondarynamenode-junqiang.out
```
看看进程:
```
➜  jps
19407 SecondaryNameNode
19239 DataNode
19104 NameNode
19574 Jps
```
**注意:**没有NameNode或者`DataNode`则`NameNode`则不对，正常启动之后可以在Web页面[http://localhost:50070](http://localhost:50070)查看:

### 伪分布式实例
最开始演示了一个单机模式的单词统计，这里来一个统计hdfs文件统计的例子:
```
➜  ./bin/hdfs dfs -mkdir -p /user/hadoop
➜  ./bin/hdfs dfs -mkdir input
➜  ./bin/hdfs dfs -put etc/hadoop/*.xml input/
➜  ./bin/hdfs dfs -ls input/
Found 8 items
-rw-r--r--   1 hadoop supergroup       4436 2016-07-19 19:37 input/capacity-scheduler.xml
-rw-r--r--   1 hadoop supergroup       1051 2016-07-19 19:37 input/core-site.xml
-rw-r--r--   1 hadoop supergroup       9683 2016-07-19 19:37 input/hadoop-policy.xml
-rw-r--r--   1 hadoop supergroup       1105 2016-07-19 19:37 input/hdfs-site.xml
-rw-r--r--   1 hadoop supergroup        620 2016-07-19 19:37 input/httpfs-site.xml
-rw-r--r--   1 hadoop supergroup       3523 2016-07-19 19:37 input/kms-acls.xml
-rw-r--r--   1 hadoop supergroup       5511 2016-07-19 19:37 input/kms-site.xml
-rw-r--r--   1 hadoop supergroup        690 2016-07-19 19:37 input/yarn-site.xml

➜  ./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.4.jar grep input/ output 'dfs[a-z.]+'
```

查看hdfs文件的统计结果可以使用:
```
➜  ./bin/hdfs dfs -cat output/*
```
结果也可以取回本地，当然记得先把本地的文件夹删了:
```
➜  rm -r output
➜  ./bin/hdfs dfs -get output output
➜  cat output/*
```

### 配置Yarn
有时候我们还需要一个任务或资源的调度器，当然如果是本地的任务，这个功能其实没啥用，反而会是任务运行变慢，这里仅当学习使用:
```
➜  cp etc/hadoop/mapred-site.xml.template etc/hadoop/mapred-site.xml
```
编辑下面两个文件`etc/hadoop/mapred-site.xml`，`etc/hadoop/yarn-site.xml`:
* etc/hadoop/mapred-site.xml

```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

* etc/hadoop/yarn-site.xml

```
<configuration>
<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

启动资源管理器
```
➜  ./sbin/start-yarn.sh
➜  ./sbin/mr-jobhistory-daemon.sh start historyserver
```
集群资源管理器的访问地址为:[http://localhost:8088/cluster](http://localhost:8088/cluster),另外第二条命令是为了看历史任务的。

关闭资源你管理器
```
➜  ./sbin/stop-yarn.sh
➜  ./sbin/mr-jobhistory-daemon.sh stop historyserver
```
