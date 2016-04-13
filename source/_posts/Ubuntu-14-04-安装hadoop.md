title: Ubuntu 14.04 安装Hadoop
date: 2015-11-05 19:53:43
tags: [hadoop,大数据,Linux]
categories: 开发环境
---
安装主要参考[Apache 官网教程](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

### 预备知识
1. Java必须安装,也就是必须有`Jre`
2. 必须装`ssh`.

**注意:**Ubuntu自带安装了ssh,但是是客户端,这里你需要安装`ssh server`,安装命令为:
```bash
sudo apt-get install ssh
sudo apt-get install rsync
```

### 安装步骤

* 下载hadoop,解压到指定目录:

[hadoop 2.7.0 官方下载地址](http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.7.0/)
```bash
tar -xvf hadoop-2.7.0.tar.gz -C /usr/qunar
```
 然后修改配置文件,设置java目录
```bash
cd /usr/qunar/hadoop-2.7.0/etc/hadoop
sudo vim hadoop-env.sh
```
 注意下面的内容:
```
# The java implementation to use.
export JAVA_HOME=${JAVA_HOME}
```
 所以如果你在环境变量里设置过`${JAVA_HOME}`环境变量,则这里不需要改,如果没有设置过,请自行修改`~/.bashrc`文件,设置环境变量:
```bash
export JAVA_HOME=/usr/qunar/jdk1.7.0_40
```

* 测试安装是否成功

```bash
cd /usr/qunar/hadoop-2.7.0/bin
./hadoop
```
 将会显示使用说明文档.

### Hadoop单机配置
Hadoop 默认模式为非分布式模式，无需进行其他配置即可运行。非分布式即单 Java 进程，方便进行调试。
现在我们可以执行例子来感受下 Hadoop 的运行。Hadoop 附带了丰富的例子:
```bash
cd /usr/qunar/hadoop-2.7.0
➜  hadoop-2.7.0  bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar
An example program must be given as the first argument.
Valid program names are:
  aggregatewordcount: An Aggregate based map/reduce program that counts the words in the input files.
  aggregatewordhist: An Aggregate based map/reduce program that computes the histogram of the words in the input files.
  bbp: A map/reduce program that uses Bailey-Borwein-Plouffe to compute exact digits of Pi.
  dbcount: An example job that count the pageview counts from a database.
  distbbp: A map/reduce program that uses a BBP-type formula to compute exact bits of Pi.
  grep: A map/reduce program that counts the matches of a regex in the input.
  join: A job that effects a join over sorted, equally partitioned datasets
  multifilewc: A job that counts words from several files.
  pentomino: A map/reduce tile laying program to find solutions to pentomino problems.
  pi: A map/reduce program that estimates Pi using a quasi-Monte Carlo method.
  randomtextwriter: A map/reduce program that writes 10GB of random textual data per node.
  randomwriter: A map/reduce program that writes 10GB of random data per node.
  secondarysort: An example defining a secondary sort to the reduce.
  sort: A map/reduce program that sorts the data written by the random writer.
  sudoku: A sudoku solver.
  teragen: Generate data for the terasort
  terasort: Run the terasort
  teravalidate: Checking results of terasort
  wordcount: A map/reduce program that counts the words in the input files.
  wordmean: A map/reduce program that counts the average length of the words in the input files.
  wordmedian: A map/reduce program that counts the median length of the words in the input files.
  wordstandarddeviation: A map/reduce program that counts the standard deviation of the length of the words in the input files.
```
直接选取`grep`例子看看如何使用,即将`input`文件夹中的所有文件作为输入,筛选出符合正则表达式`dfs[a-z.]+`的单词并且统计出现的次数,最后将结果输出到`output`文件夹中.
```bash
cd /usr/qunar/hadoop-2.7.0
mkdir input
cp ./etc/hadoop/*.xml input
./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep input output 'dfs[a-z.]+'
```
看看输出信息
```
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=123
	File Output Format Counters 
		Bytes Written=23
```
最后我们来看看结果:
```
cd output
➜  output  cat part-r-00000 
1 dfsadmin
```
所以可以看到统计出了一个单词,出现了一次.
**注意:**Hadoop默认不会覆盖结果文件,再次运行会出错,记得删掉`output`文件夹.

### Hadoop伪分布式配置
Hadoop 可以在单节点上以伪分布式的方式运行，Hadoop 进程以分离的 Java 进程来运行，节点既作为 NameNode 也作为 DataNode。
Hadoop 的配置文件位于`/usr/local/hadoop/etc/hadoop/`中，伪分布式需要修改2个配置文件`core-site.xml`和`hdfs-site.xml`。Hadoop的配置文件是`xml`格式，每个配置以声明`property`的`name`和`value`的方式来实现。
修改配置文件`core-site.xml`
```bash
sudo vim /usr/qunar/hadoop-2.7.0/etc/hadoop/core-site.xml
```
```xml
为下面内容:
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
修改存储配置文件`hdfs-site.xml`
```bash
sudo vim /usr/qunar/hadoop/etc/hadoop-2.7.0/hdfs-site.xml
```
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
以上配置来自官方教程,这样就可运行了,但是有个问题,默认使用`/tmp/hadoo-hadoop`作为临时目录,机器重启可能会被清理掉,然后就需要重新执行`format`.所以在配置里还需要再加上一些内容,最终配置如下:
* core-site.xml

```xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
	<value>file:/usr/local/hadoop/tmp</value>
	<description> 临时文件存放目录 </description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
* hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
	<description> 文件副本个数，单机测试一个就够了</description>
    </property>
    <!-- 下面是节点配置 -->
    <property>
        <name>dfs.namenode.name.dir</name>
	<value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
	<value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```
然后就可以初始化节点了,注意当前路径是在`hadoop-2.7.0/bin`目录下：
```bash
sudo bash hdfs namenode -format
```
如果报错：
> Error: JAVA_HOME is not set and could not be found.

则需要修改`hadoop-2.7.0/etc/hadoop`目录下的`hadoop-env.sh`文件：
将其中的
```shell
export JAVA_HOME=${JAVA_HOME}
```
改为你JDK的绝对路径，然后重新执行就可以了，成功初始化之后的返回结果是：
```
16/02/27 11:51:18 INFO util.ExitUtil: Exiting with status 0
16/02/27 11:51:18 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at test/127.0.0.1
************************************************************/
```
由于只是想在本地调试，所以我还是主要用的本地模式，主要是集群上查数太满了，所以把数据下载下来然后查询会非常节省时间。上面的伪分布式可以不用配置，用官网默认的就行。
