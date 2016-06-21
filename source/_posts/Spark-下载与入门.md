title: Spark 下载与入门
date: 2016-06-21 23:03:39
tags: [大数据,Spark, 开发环境]
categories: 开发环境
---
开发环境:
OS: Ubuntu 16.04 64bit

### 安装步骤
#### 下载解压
```
sudo wget http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.6.tgz
sudo tar -xvf spark-1.6.1-bin-hadoop2.6.tgz -C /usr/dev 
sudo chown -R anonymous:anonymous spark-1.6.1-bin-hadoop2.6	# anonymous为当前用户名
```

#### 运行Spark
Spark支持多种语言,我们可以通过Python,Scala进入Spark交互式环境。
* Scala Shell

```
cd /usr/dev/spark-1.6.1-bin-hadoop2.6
./bin/spark-shell	# 启动Scala Shell
```
**注意:**如果报错:
```
Tue Jun 21 23:30:04 CST 2016 Thread[main,5,main] java.io.FileNotFoundException: derby.log (权限不够)
16/06/21 23:30:04 WARN Connection: BoneCP specified but not present in CLASSPATH (or one of dependencies)
Tue Jun 21 23:30:04 CST 2016 Thread[main,5,main] Cleanup action starting
ERROR XBM0H: Directory /usr/dev/spark-1.6.1-bin-hadoop2.6/bin/metastore_db cannot be created.
```
出现这个问题一般就是没有创建文件的权限,安装的最后一个命令就是起这个作用的
```
sudo chown -R anonymous:anonymous spark-1.6.1-bin-hadoop2.6	# anonymous为当前用户名
```

* Python Shell

```
./bin/pyspark
```
为了在Shell里面写Python有补全提示，强烈建议安装`ipython`,Ubuntu安装也很简单
```
sudo apt install -y ipython
```
然后启动的时候可以这样:
```
IPYTHON=1 ./bin/pyspark
```
进入Shell大概是这样的:
```
➜  spark-1.6.1-bin-hadoop2.6 IPYTHON=1 bin/pyspark
Python 2.7.11+ (default, Apr 17 2016, 14:00:29) 
Type "copyright", "credits" or "license" for more information.

IPython 2.4.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
16/06/21 23:57:06 INFO SparkContext: Running Spark version 1.6.1
16/06/21 23:57:06 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 1.6.1
      /_/

Using Python version 2.7.11+ (default, Apr 17 2016 14:00:29)
SparkContext available as sc, HiveContext available as sqlContext.

In [1]: lines = sc.textFile("README.md")
```

#### 简单样例
Scala也不熟，就以Python为例吧,注意我的当前目录是在`/usr/dev/spark-1.6.1-bin-hadoop2.6`:
```
In [1]: lines = sc.textFile("README.md")
 n [2]: lines.count()
16/06/21 23:57:41 INFO FileInputFormat: Total input paths to process : 1
16/06/21 23:57:41 INFO SparkContext: Starting job: count at <ipython-input-2-44aeefde846d>:1
16/06/21 23:57:41 INFO DAGScheduler: Got job 0 (count at <ipython-input-2-44aeefde846d>:1) with 2 output partitions
16/06/21 23:57:41 INFO DAGScheduler: Final stage: ResultStage 0 (count at <ipython-input-2-44aeefde846d>:1)
16/06/21 23:57:41 INFO DAGScheduler: Parents of final stage: List()
16/06/21 23:57:41 INFO DAGScheduler: Missing parents: List()
16/06/21 23:57:41 INFO DAGScheduler: Submitting ResultStage 0 (PythonRDD[2] at count at <ipython-input-2-44aeefde846d>:1), which has no missing parents
16/06/21 23:57:41 INFO MemoryStore: Block broadcast_1 stored as values in memory (estimated size 5.6 KB, free 173.1 KB)
16/06/21 23:57:41 INFO MemoryStore: Block broadcast_1_piece0 stored as bytes in memory (estimated size 3.4 KB, free 176.6 KB)
16/06/21 23:57:41 INFO BlockManagerInfo: Added broadcast_1_piece0 in memory on localhost:36609 (size: 3.4 KB, free: 511.5 MB)
16/06/21 23:57:41 INFO SparkContext: Created broadcast 1 from broadcast at DAGScheduler.scala:1006
16/06/21 23:57:41 INFO DAGScheduler: Submitting 2 missing tasks from ResultStage 0 (PythonRDD[2] at count at <ipython-input-2-44aeefde846d>:1)
16/06/21 23:57:41 INFO TaskSchedulerImpl: Adding task set 0.0 with 2 tasks
16/06/21 23:57:41 INFO TaskSetManager: Starting task 0.0 in stage 0.0 (TID 0, localhost, partition 0,PROCESS_LOCAL, 2151 bytes)
16/06/21 23:57:41 INFO TaskSetManager: Starting task 1.0 in stage 0.0 (TID 1, localhost, partition 1,PROCESS_LOCAL, 2151 bytes)
16/06/21 23:57:41 INFO Executor: Running task 0.0 in stage 0.0 (TID 0)
16/06/21 23:57:41 INFO Executor: Running task 1.0 in stage 0.0 (TID 1)
16/06/21 23:57:41 INFO HadoopRDD: Input split: file:/usr/dev/spark-1.6.1-bin-hadoop2.6/README.md:0+1679
16/06/21 23:57:41 INFO HadoopRDD: Input split: file:/usr/dev/spark-1.6.1-bin-hadoop2.6/README.md:1679+1680
16/06/21 23:57:41 INFO deprecation: mapred.tip.id is deprecated. Instead, use mapreduce.task.id
16/06/21 23:57:41 INFO deprecation: mapred.task.id is deprecated. Instead, use mapreduce.task.attempt.id
16/06/21 23:57:41 INFO deprecation: mapred.task.is.map is deprecated. Instead, use mapreduce.task.ismap
16/06/21 23:57:41 INFO deprecation: mapred.task.partition is deprecated. Instead, use mapreduce.task.partition
16/06/21 23:57:41 INFO deprecation: mapred.job.id is deprecated. Instead, use mapreduce.job.id
16/06/21 23:57:42 INFO PythonRunner: Times: total = 283, boot = 268, init = 14, finish = 1
16/06/21 23:57:42 INFO PythonRunner: Times: total = 291, boot = 272, init = 19, finish = 0
16/06/21 23:57:42 INFO Executor: Finished task 0.0 in stage 0.0 (TID 0). 2124 bytes result sent to driver
16/06/21 23:57:42 INFO Executor: Finished task 1.0 in stage 0.0 (TID 1). 2124 bytes result sent to driver
16/06/21 23:57:42 INFO TaskSetManager: Finished task 0.0 in stage 0.0 (TID 0) in 407 ms on localhost (1/2)
16/06/21 23:57:42 INFO TaskSetManager: Finished task 1.0 in stage 0.0 (TID 1) in 393 ms on localhost (2/2)
16/06/21 23:57:42 INFO DAGScheduler: ResultStage 0 (count at <ipython-input-2-44aeefde846d>:1) finished in 0.423 s
16/06/21 23:57:42 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool 
16/06/21 23:57:42 INFO DAGScheduler: Job 0 finished: count at <ipython-input-2-44aeefde846d>:1, took 0.489031 s
Out[2]: 95

In [3]: lines.first()
16/06/21 23:58:15 INFO SparkContext: Starting job: runJob at PythonRDD.scala:393
16/06/21 23:58:15 INFO DAGScheduler: Got job 1 (runJob at PythonRDD.scala:393) with 1 output partitions
16/06/21 23:58:15 INFO DAGScheduler: Final stage: ResultStage 1 (runJob at PythonRDD.scala:393)
16/06/21 23:58:15 INFO DAGScheduler: Parents of final stage: List()
16/06/21 23:58:15 INFO DAGScheduler: Missing parents: List()
16/06/21 23:58:15 INFO DAGScheduler: Submitting ResultStage 1 (PythonRDD[3] at RDD at PythonRDD.scala:43), which has no missing parents
16/06/21 23:58:15 INFO MemoryStore: Block broadcast_2 stored as values in memory (estimated size 4.8 KB, free 181.3 KB)
16/06/21 23:58:15 INFO MemoryStore: Block broadcast_2_piece0 stored as bytes in memory (estimated size 3.0 KB, free 184.3 KB)
16/06/21 23:58:15 INFO BlockManagerInfo: Added broadcast_2_piece0 in memory on localhost:36609 (size: 3.0 KB, free: 511.5 MB)
16/06/21 23:58:15 INFO SparkContext: Created broadcast 2 from broadcast at DAGScheduler.scala:1006
16/06/21 23:58:15 INFO DAGScheduler: Submitting 1 missing tasks from ResultStage 1 (PythonRDD[3] at RDD at PythonRDD.scala:43)
16/06/21 23:58:15 INFO TaskSchedulerImpl: Adding task set 1.0 with 1 tasks
16/06/21 23:58:15 INFO TaskSetManager: Starting task 0.0 in stage 1.0 (TID 2, localhost, partition 0,PROCESS_LOCAL, 2151 bytes)
16/06/21 23:58:15 INFO Executor: Running task 0.0 in stage 1.0 (TID 2)
16/06/21 23:58:15 INFO HadoopRDD: Input split: file:/usr/dev/spark-1.6.1-bin-hadoop2.6/README.md:0+1679
16/06/21 23:58:15 INFO PythonRunner: Times: total = 41, boot = -33244, init = 33285, finish = 0
16/06/21 23:58:15 INFO Executor: Finished task 0.0 in stage 1.0 (TID 2). 2143 bytes result sent to driver
16/06/21 23:58:15 INFO TaskSetManager: Finished task 0.0 in stage 1.0 (TID 2) in 67 ms on localhost (1/1)
16/06/21 23:58:15 INFO DAGScheduler: ResultStage 1 (runJob at PythonRDD.scala:393) finished in 0.068 s
16/06/21 23:58:15 INFO TaskSchedulerImpl: Removed TaskSet 1.0, whose tasks have all completed, from pool 
16/06/21 23:58:15 INFO DAGScheduler: Job 1 finished: runJob at PythonRDD.scala:393, took 0.082887 s
Out[3]: u'# Apache Spark'
```
看看，其实很简单的，但是有个问题，每次输入一个操作，结果搞出这么一大堆的日志，看着确实影响输入，我们可以控制一下Spark的输出日志级别,编辑`spark-1.6.1-bin-hadoop2.6/conf/log4j.properties.template`文件,为了安全起见，我们直接拷贝一份模板作为我们的日志配置文件:
```
cd conf/
sudo cp log4j.properties.template log4j.properties
```
定位到这一行:
```
log4j.rootCategory=INFO, console
```
改为:
```
log4j.rootCategory=WARN, console
```
看看之前的操作:
```
Using Python version 2.7.11+ (default, Apr 17 2016 14:00:29)
SparkContext available as sc, HiveContext available as sqlContext.

In [1]: lines = sc.textFile("README.md")

In [2]: lines.count()
Out[2]: 95

In [3]: lines.first()
Out[3]: u'# Apache Spark'

In [4]: 
```
这样就只会输出警告及以上的日志了。
