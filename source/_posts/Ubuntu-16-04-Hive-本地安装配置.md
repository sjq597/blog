title: Ubuntu 16.04 Hive 本地安装配置
date: 2016-07-20 15:24:23
tags: [Linux,Hive,大数据]
categories: 开发环境
---
很多时候需要测试一下我们的sql逻辑是否有问题，或者想在本地调试一个东西，线上的Hadoop跑的太慢，要视资源和进群调度情况而定，所以本地调试无疑是效率最高的方式，记录一下本地配置Hive的过程。

### 系统环境
```
OS: Ubuntu 16.04 LTS 64bit
JDK: 1.7.0_40
Hadoop:hadoop-2.6.4.tar.gz
MySQL: 5.7.13
Hive:
```

### 安装步骤
首先你需要安装配置好`Jdk`,`Hadoop`,`MySQL`这三个东西，前面两个可以参见上一篇笔记[Ubuntu 16 04 Hadoop本地安装配置](../../19/Ubuntu-16-04-Hadoop本地安装配置)
**注意:**本篇所有的安装软件都安装在`Hadoop`用户下，请务必注意.具体的可以使用``ssh hadoop@localhost`登陆，然后在终端中操作.

#### 安装MySQL
```
sudo apt install -y mysql-server
```
设置对应的密码就行，其他的默认就可以，比较简单就不细讲了.主要讲后面的。

#### 创建Hive用户
登陆MySQL，创建Hive用户:
```
mysql -uroot -proot
create user 'hive' identified by 'hive';
grant all privileges on *.* to 'hive'@'localhost' identified by 'hive';
```
**注意:**第一个`hive`是创建的用户名为`hive`,`identified by`后面的那个`hive`是密码。
然后用刚创建的用户登陆,并创建数据库:
```
mysql -uroot -proot
mysql> create database hive;
```

#### 安装Hive
去Apache官网下载就行[Hive官网地址](https://hive.apache.org/),我这里下载的是最新的版本`apache-hive-2.1.0-bin.tar.gz`:
```
sudo tar -xvf apache-hive-2.1.0-bin.tar.gz -C /usr/dev/
sudo chown -R hadoop /usr/dev/apache-hive-2.1.0-bin/
```
然后需要配置Hive,一下操作默认都是在`/usr/dev/apache-hive-2.1.0-bin`目录中操作的.
```
cp conf/hive-default.xml.template hive-site.xml
```
然后修改`hive-site.xml`文件中对应位置的内容，根据实际情况修改，比如数据库的名字，用户名，密码等，我的配置如下:
```
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/home/hadoop/hive/warehouse</value>
  <description>location of default database for the warehouse</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive?characterEncoding=UTF-8&createDatabaseIfNotExist=true</value>
  <description>
    JDBC connect string for a JDBC metastore.
    To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
    For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
  </description>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
  <description>Username to use against metastore database</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>hive</value>
  <description>password to use against metastore database</description>
</property>
```
MySQL主要用来存储元数据，即一些表的信息，默认的是`Derby`,现在我们改为用`MySQL`，所以还需要把`Jdbc`驱动包拷贝到Hive的库目录下,如果本地没有可以去[MySQL官网](https://dev.mysql.com/downloads/connector/j/)下载,不过如果你用过`JetBrain`系列的IDE，例如`IDEA`,`PyCharm`，可以在该用户目录下的`.IntelliJIdea14`目录下找到这个jar包，把这个jar包拷贝到Hive库目录下即可:
```
locate mysql-connector-java
```
根据查找内容找到jar包所在目录，可能会有很多地方有这个jar包，随便拷一个就行，拷贝该jar包即可,以我的为例:
```
cp /home/anonymous/.m2/repository/mysql/mysql-connector-java/5.1.16/mysql-connector-java-5.1.16.jar /usr/dev/apache-hive-2.1.0-bin/lib
```
到这里终于可以启动Hive试一试了:
```
./bin/hive
Logging initialized using configuration in jar:file:/usr/dev/apache-hive-2.1.0-bin/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Exception in thread "main" java.lang.RuntimeException: org.apache.hadoop.hive.ql.metadata.HiveException: org.apache.hadoop.hive.ql.metadata.HiveException: MetaException(message:Hive metastore database is not initialized. Please use schematool (e.g. ./schematool -initSchema -dbType ...) to create the schema. If needed, don't forget to include the option to auto-create the underlying database in your JDBC connection string (e.g. ?createDatabaseIfNotExist=true for mysql))
	at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:578)
	at org.apache.hadoop.hive.ql.session.SessionState.beginStart(SessionState.java:518)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:705)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:641)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:221)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:136)
Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: org.apache.hadoop.hive.ql.metadata.HiveException: MetaException(message:Hive metastore database is not initialized. Please use schematool (e.g. ./schematool -initSchema -dbType ...) to create the schema. If needed, don't forget to include the option to auto-create the underlying database in your JDBC connection string (e.g. ?createDatabaseIfNotExist=true for mysql))
	at org.apache.hadoop.hive.ql.metadata.Hive.registerAllFunctionsOnce(Hive.java:226)
```
启动不了,看错误信息里面有提示，好像是元数据库没有初始化，就照着搞一下。
```
./bin/schematool -initSchema -dbType mysql
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/dev/apache-hive-2.1.0-bin/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/dev/hadoop-2.6.4/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:	 jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true
Metastore Connection Driver :	 com.mysql.jdbc.Driver
Metastore connection User:	 hive
Starting metastore schema initialization to 2.1.0
Initialization script hive-schema-2.1.0.mysql.sql
Initialization script completed
schemaTool completed
```
好，再启动一下试试:
```
./bin/hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/dev/apache-hive-2.1.0-bin/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/dev/hadoop-2.6.4/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/usr/dev/apache-hive-2.1.0-bin/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Exception in thread "main" java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D
	at org.apache.hadoop.fs.Path.initialize(Path.java:206)
	at org.apache.hadoop.fs.Path.<init>(Path.java:172)
.......
```
尼玛，还是不行，继续排查,同样，看错误信息貌似是配置文件里面的`${system:java.io.tmpdir`这个出问题了，解决方法:替换掉`conf/hive-site.xml`里面的所有的`${system:java.io.tmpdir}`为固定值，比如我设置的就是`/home/hadoop/tmp`,只列出两个，有好几处，都得替换掉:
```
<property>
    <name>hive.exec.local.scratchdir</name>
    <value>/home/hadoop/tmp/${system:user.name}</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/home/hadoop/tmp/${hive.session.id}_resources</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
```
好，再启动试一下:
```
./bin/hive
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/dev/apache-hive-2.1.0-bin/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/dev/hadoop-2.6.4/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/usr/dev/apache-hive-2.1.0-bin/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. tez, spark) or using Hive 1.X releases.
hive> 
```
终于启动成功了,试一下基本的命令:
```
hive> show databases;
OK
Failed with exception java.io.IOException:java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:user.name%7D
Time taken: 0.99 seconds
```
尼玛，又不行，同样的，把`conf/hive-site.xml`里面的`${system:user.name}`全部替换成`${user.name}`，然后重新试一下:
```
hive> show tables;
OK
Time taken: 0.981 seconds
hive> show databases;
OK
default
Time taken: 0.024 seconds, Fetched: 1 row(s)
hive> 
```
到此终于搞定了。试下创建一张表：
```
hive> create database dw_subject;
OK
Time taken: 0.233 seconds
hive> show databases;
OK
default
dw_subject
Time taken: 0.017 seconds, Fetched: 2 row(s)
hive> use dw_subject;
OK
Time taken: 0.021 seconds
hive> create table table_test (
    >   id bigint comment '自增id',
    >   name string comment '姓名'
    > ) comment '测试表'
    > ROW FORMAT DELIMITED
    > FIELDS TERMINATED BY '\t'
    > LINES TERMINATED BY '\n';
OK
Time taken: 0.315 seconds
hive> show create table table_test;
OK
CREATE TABLE `table_test`(
  `id` bigint COMMENT '??id', 
  `name` string COMMENT '??')
COMMENT '???'
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='\t', 
  'line.delim'='\n', 
  'serialization.format'='\t') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://localhost:9000/home/hadoop/hive/warehouse/dw_subject.db/table_test'
TBLPROPERTIES (
  'COLUMN_STATS_ACCURATE'='{\"BASIC_STATS\":\"true\"}', 
  'numFiles'='0', 
  'numRows'='0', 
  'rawDataSize'='0', 
  'totalSize'='0', 
  'transient_lastDdlTime'='1469260899')
Time taken: 0.156 seconds, Fetched: 23 row(s)
```
这里可以看到有个问题，中文注释乱码了，怎么解决呢？网上有相关的教程，需要修改源码，重新编译`hive-exec-2.1.0.jar`这个包，暂时能用了，本机基本上是测试用，中文乱码也不影响使用，有空再研究怎么去乱码。
