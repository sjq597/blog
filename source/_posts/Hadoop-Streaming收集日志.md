title: Hadoop Streaming收集日志
date: 2016-07-18 15:58:11
tags: [Linux, Hadoop]
categories: 数据挖掘
---
公司的日志一般会有专门的日志收集系统，但是上传到hdfs上目录太多，一般都是按机房，按小时分割日志文件的。路径类似于下面这样:
```
/user/xxx/l-xxxx1.pay.cn1/20160717/log.20160717-18.gz
/user/xxx/l-xxxx1.pay.cn1/20160717/log.20160717-19.gz
/user/xxx/l-xxxx2.pay.cn1/20160717/log.20160717-18.gz
/user/xxx/l-xxxx3.pay.cn1/20160717/log.20160717-19.gz
```
即日志文件会按小时打包`log.20160717-xx.gz`,另外日志可能会标注机房`l-xxxx[1-9].pay.cn[1-9]`不同机器,这样会对应很多个目录，这样就无法通过在Hive里面建一个表指向一个固定的hdfs路径来分析日志数据。我们必须通过一个方法把这些日志转化到一个路径，然后把数据`load`到Hive表中，然后就可以对日志的数据做一些分析，或者使用了。

### Hadoop Streaming
这里使用到的是Hadoop原生的`Streaming`，毕竟我们的目的也不是很负复杂，就是一个数据收集汇总的过程，当然这中间也可以做一些简单的处理，例如过滤掉不需要的日志，毕竟日志不比MySQL里面的结构化数据，日志的量级一般都很大，都装到Hive表里面数据量大分析的时候也需要更多的计算资源，也更慢

由于服务器上的`hadoop`是2.2.0版本的，所以我这里使用的是`hadoop-streaming-2.2.0.jar`。主要使用`shell`,`python`简单介绍一下如何使用这个。
```
$HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
    -input myInputDirs \
    -output myOutputDir \
    -mapper /bin/cat \
    -reducer /bin/wc
```
上面是具体的在shell里面执行的时候参数的指定，这里最简单的用法就是执行输入目录或者输入文件，以我上面的例子为例，涉及到多个机房，日志还按小时分割打包，可以写成
```
-input /user/xxx/l-xxxx*.pay.cn*/20160717/log.20160717-*.gz
```
即路径可以使用通配符`*`指定,然后还要指定一个汇总日志输出路径:
```
-output /user/xxx/logs/xxxx/output/20160717/log_parse 
```

这里主要是想讲一下`mapper`和`reducer`文件的编写，采用python实现:
先看一下`mapper.py`文件的实现
```
#!/usr/bin/env python
# coding=utf-8

import re
import sys

reg_pattern = r'(\d+-\d+-\d+ \d+:\d+:\d+.\d+).*GwRouteBizImpl\s*-\s*(\w*)\{(.*)\}.*'
re_gx = re.compile(reg_pattern)

for line in sys.stdin:
    line = line.strip().replace('\t', ' ').replace('\r', ' ')

    if re_gx.match(line):
        print '%s' % re.sub(reg_pattern, r'\1\t\2\t\3', line)
```
这就是一个最简单的过滤日志，并简单做一些字段的提取，因为我们采用`\t`分割，所以需要把每一行的特殊字符去掉，然后用正则匹配满足格式的日志，如果不满足，这条日志就直接过滤掉了。
**注意:**使用Hadoop原生的`Streaming`程序，只能处理标准输入输出,另外如果对输入不做任何处理，可以直接使用系统自带的`/bin/cat`，不用单独写`mapper`或者`reducer`，当然你也可以使用python写一个,什么也不干，原样输出`reducer.py`:
```
import sys

for line in sys.stdin:
    line = line.strip()
    print '%s\t' % line
```

具体使用就很简单了，在shell里面调用:
> sudo -u${HADOOP_USER} ${HADOOP_HOME}/bin/hadoop jar ${HADOOP_HOME}/share/hadoop/tools/lib/hadoop-streaming-2.2.0.jar -D stream.non.zero.exit.is.failure=false -mapper "python mapper.py" -reducer "python reducer.py" -input ${HADOOP_LOG_FILE} -output ${HADOOP_OUTPUT_DIR} -file reducer.py -file mapper.py

**备注:**如果`mapper`,`reducer`不是使用系统的shell命令,那么就需要加上`-file`参数来把我们用其他程序写的代码分发到所有节点。另外还有一个地方需要注意，要确保`-output output_dir`路径不存在，你需要在调用之前调用删除命令:
```
sudo -u${HADOOP_USER} ${HADOOP_HOME}/bin/hadoop fs -rm -r output_dir
```

还有一些常用的参数设置:
```
-jobconf <key>=<value>
```
例如:
```
-jobconf mapreduce.job.queue.name=xxx -jobconf mapreduce.job.name=xxx
```

### 常用操作
对于日志这种，其实是多路输入，合并到一路输出，中间只是把一行内容变成一行内容，用特定字符分隔开来而已，所以并没有`reduce`过程，如果我们用了上面那个参数，最后会又有一个`reducer`，特别慢，关键是有可能内存不够，所以可以取消掉
```
-jobconf mapred.reduce.tasks=0
```
这样就不会有`reducer`了，会快很多。
