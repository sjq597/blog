title: IllegalAccessError HBaseZeroCopyByteString
date: 2019-02-22 19:32:09
tags: [HBase, Hadoop, Java]
categories: 数据架构
---
# 背景
最近有个Flink实时作业写HBase的任务发现丢数据了，Flink平台和HBase运维也无法定位到具体的问题，也没有任何异常日志。没办法只能通过把HBase数据导出到离线Hadoop集群来分析。
一开始怀疑MQ没有采集到日志，后来通过把Kafka日志拉取到HDFS查询发现数据是有的，那问题就只可能是在计算过程中丢失了。万幸实时采集的数据都有落HDFS，所以想离线分析一波，首先让运维
给HBase打了一个快照，然后给了个MR代码让我自己解析数据结构。

# 问题
其实代码很简单，就是解析Cell把值解析出来然后写到HDFS路径上,需要用到的包也不多,pom.xml文件如下:
``结构。

# 代码
其实代码很简单，就是解析Cell把值解析出来然后写到HDFS路径上,需要用到的包也不多,pom.xml文件如下:
```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.5</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.5</version>
</dependency>

<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>${hbase.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-protocol</artifactId>
    <version>${hbase.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-common</artifactId>
    <version>${hbase.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-server</artifactId>
    <version>${hbase.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-client-core</artifactId>
    <version>${hadoop.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>${hadoop.version}</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.54</version>
</dependency>
```

具体的版本根据你的集群而定，然后就是解析程序了:
```java
public class HBase2HDFSApp {
    static Logger LOG = LoggerFactory.getLogger(HBase2HDFSApp.class);
    /**
     *      * 需要传入的参数：快照名字，解析之后输出路径，快照输入路径，临时路径
     *      * @param args
     *      * @throws Exception
     *
     */
    public static void main(String[] args) throws Exception {
        if (args == null) {
            System.err.println("Parameter Errors ! Usage : <snapshot_name> <output_path> <input_path> <tmp_output_path>");
            System.exit(-1);
        }
        String snapShotName = args[0];
        Path outputPath = new Path(args[1]);
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.rootdir", "hdfs://" + args[2]);
        configuration.set("mapreduce.job.queuename", "xxx");
        configuration.set("hadoop.tmp.dir", "xxxx");
        String jobName = HBase2HDFSApp.class.getSimpleName();
        Job job = Job.getInstance(configuration, jobName);
        job.setJarByClass(HBase2HDFSApp.class);
        LOG.info("start to init");
        TableMapReduceUtil.initTableSnapshotMapperJob(snapShotName,
                new Scan(),
                HBase2HDFSMapper.class,
                Text.class,
                NullWritable.class,
                job, true, new Path(args[3]));
        LOG.info("init success");
        outputPath.getFileSystem(configuration).delete(outputPath, true);
        FileOutputFormat.setOutputPath(job, outputPath);
        job.setOutputFormatClass(TextOutputFormat.class);
        job.setNumReduceTasks(0);//没有reduce
        job.waitForCompletion(true);

    }
    public static class HBase2HDFSMapper extends TableMapper<Text, NullWritable> {
        @Override
        protected void map(ImmutableBytesWritable key, Result rs, Context context) throws IOException, InterruptedException {
            byte[] keyBytes = key.get();
            JSONObject value = new JSONObject();
            String rk = new String(keyBytes);
            List<Cell> list = rs.listCells();
            for (int i = 0; i < list.size(); i++) {
                Cell cell = list.get(i);
                value.put(new String(CellUtil.cloneQualifier(cell)), new String(CellUtil.cloneValue(cell)));
            }
            context.write(new Text(rk + "\t" + value.toJSONString()), NullWritable.get());
        }
    }
}
```
**NOTE:**需要注意的是`<output_path>和<tmp_output_path>`根目录要一致,然后就是不能使`<input_path>`的子目录.
编译打包之后提交运行，注意打包需要用到`assembly`插件,对应的`pom.xml`配置为:
```
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id> <!-- this is used for inheritance merges -->
            <phase>package</phase> <!-- bind to the packaging phase -->
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>`配置为:
```
然后打包之后运行:
```
hadoop jar ./target/xxx-1.0.0-SNAPSHOT-jar-with-dependencies.jar com.xxx.HBase2HDFSApp \
<snapshot_name> \
<output_path> \
<input_path> \
<tmp-output_path>
```
然后就出现了一个经典的错误:
```
Caused by: java.lang.IllegalAccessError: class com.google.protobuf.HBaseZeroCopyByteString cannot access its superclass com.google.protobuf.LiteralByteString
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
        at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
        xxxx
```

# 问题原因
google了一下,找到这个问题的解决方法,详细见链接: http://www.voidcn.com/article/p-hhmhpejc-bh.html. 大概就是引入了一个优化措施导致的,
这个问题的发生是由于优化了[HBASE-9867](https://issues.apache.org/jira/browse/HBASE-9867)引起的，无意间引进了一个依赖类加载器。它影响使用-libjars参数和使用 fat jar两种模式的job. 
fat jar模式Hadoop的一个特殊功能：可以读取操作目录中/lib目录下包含的所有库的JAR文件，把运行job依赖的jar放在jar中的lib目录下。

解决方式也比较简单:
1. 把缺的这个包拷贝到hadoop lib目录
2. 环境变量中导入这个缺失的包

由于我是临时跑一次，而且hadoop环境是公用的，直接破坏了不好，就采用的临时方案.首先定位到`com.google.protobuf.HBaseZeroCopyByteString`位于`hive-server`包中，具体对应的jar包是
> hbase-protocol-0.98.21-hadoop2-xxxx.jar

具体的版本看你们公司集群编译之后对应的包版本即可,然后调整运行命令为:
```
export HADOOP_CLASSPATH=xxxx/hbase-protocol-0.98.21-hadoop2-xxx.jar
hadoop jar ./target/xxx-1.0.0-SNAPSHOT-jar-with-dependencies.jar com.xxx.HBase2HDFSApp \
<snapshot_name> \
<output_path> \
<input_path> \
<tmp-output_path>
```
然后运行就行了，大工告成。


