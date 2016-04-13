title: Hive 参数设置
date: 2016-02-20 14:14:10
tags: [Hive]
categories: Hive笔记
---
## Hive参数设置方式
Hive提供了很多可设置参数，可以通过设置不同的参数来满足不同场景的各种需求，改变Hive有三种方式：
1. 修改Hive安装路径下的配置文件，即修改${HIVE_HOME}/conf/hive-site.xml文件里的配置；
2. 在启动Hive时，在命令行里面设置参数；
3. 在进入Hive的CLI客户端里面进行参数设置。

### 修改配置文件设置参数
在安装好Hive之后，默认的配置会在${hive_home}/conf/hive-default.xml文件里面，一般都会对默认的配置做一定的修改，如果要修改默认配置，可以先在相同目录下创建一个`hive-site.xml`,格式如下：
```xml
<configuration>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
        <description>
	    location of default database for the warehouse
	</description>
    </property>
</configuration>
```
所有的配置都是以`<configuration></configuration>`标签里面，里面可以有多个`<property></property>`标签，每个`<property></property>`标签里面可以设定我们需要设定的属性值：
1. `<name></name>:`要设定的属性的属性名
2. `<value></value>:`设定属性的值
3. `<description></description>:`描述属性的一些介绍性语句，可以不写
大部分共同的配置都是写到这里的，如果有必要的话，因为这里面的配置对全局用户都有效，并且一旦设置就永久有效。并且`hive-site.xml`里的配置会覆盖`hive-default.xml`里的配置，并且由于Hive是作为Hadoop的客户端启动的，所以Hive同时也会读取Hadoop的配置，同理，Hive的配置会覆盖Hadoop的配置。

### 命令行设置参数
在命令行启动Hive进入CLI的时候，可以在命令行里面添加`--hiveconf param=value`来设定参数，例如，在终端里输入：
```bash
$ hive --hiveconf mapreduce.job.queuename=queue1
```
这个是设置MapReduce任务的队列，注意这个设置是临时的，一旦你退出了Hive客户端，这个配置就失效了，下次如果想使用这个配置，你必须重新配置。

### 进入Hive的CLI客户端设置参数
当我们已经进入了CLI里面，可以使用`set`关键字来设置参数，例如：
```sql
>hive set mapreduce.job.queuename=queue1;
```
这个和上一种方式差不多，效果也是一样，一旦退出CLI客户端，也会失效，但是和方法二有一个不同的地方是，如果set后面只跟参数名而不带参数值，就可以查看这个参数目前的值，像下面这样：
```sql
>hive set mapreduce.job.queuename;
mapreduce.job.queuename=queue1
```
如果set后面连参数名都不跟，那么就可以查看整个Hive的所有配置
```sql
>hive set;
datanucleus.autoCreateSchema=true
datanucleus.autoStartMechanismMode=checked
datanucleus.cache.level2=false
datanucleus.cache.level2.type=none
datanucleus.connectionPoolingType=DBCP
datanucleus.identifierFactory=datanucleus
datanucleus.plugin.pluginRegistryBundleCheck=LOG
datanucleus.storeManagerType=rdbms
datanucleus.transactionIsolation=read-committed
```
上面介绍的这三种参数设置方式的优先级类似于编程语言里的变量声明，即本地变量会覆盖全局变量，优先级是依此递增的

## 常用Hive配置
注意，这些设置里面没有双引号
```sql
# 设置job名
set mapred.job.name=这是一个测试;
# 设置job执行队列名
set mapred.job.queue.name=root.tcop;
# 设定reduce内存大小,单位M
set mapreduce.reduce.memory.mb=8048;
set mapred.child.reduce.java.opts=-Xmx8048M;
set mapreduce.reduce.java.opts=-Xmx8048M;
```

