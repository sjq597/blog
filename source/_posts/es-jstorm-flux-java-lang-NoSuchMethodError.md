title: es jstorm flux java.lang.NoSuchMethodError
date: 2017-04-30 10:51:09
tags: [Java,ElasticSearch,Jstorm]
categories: Java笔记
---
最近在研究怎么通过Flux来解析日志，前前后后踩了很多坑,毕竟貌似网上也没人这么干过，写一个通用化的代码，只通过配置文件来定义流计算任务．基本上写好了代码,以后再有新的任务都不用再写java代码,然后打包上传提交任务了．
开始集群上的jstorm是2.1.1的,并不支持flux方式的提交任务，所有升级了一下到2.2.1版本,最后跑了一下官方的`word-count`demo成功了.
然后开发好了通用的jar包来解析日志，简单来说就是:

* CommonKafkaSpout
给构造函数传入指定topic来从kafka读取日志数据

* CommonBolt
传入解析规则，基本上就是正则表达式,细节不多说了,解析出`<key,value>`

* Write2ESBolt
把上游的内容写入到es中，构造函数也需要传入es的配置，基本上就是index,type这些

所有的基本走通了之后,开始测试的时候Topology是用java代码写的,在集群上测试之后没啥问题，然后改用`yaml`文件来定义jstorm任务,结果提交到集群就有问题了,十分的奇怪,照理说`word-count`demo跑通了应该没啥问题了，但是涉及到真正的实战需要连接kafka,es就出问题了，报错:
```
java.lang.NoSuchMethodError: com.google.common.util.concurrent.MoreExecutors.sameThreadExecutor
类似的还有
NoSuchMethodError: com.google.common.util.concurrent.MoreExecutors.directExecutor conflits on Elastic Search jar
```
然后google了很久，发现这个并不是es的问题，是guava的问题,原文在
> 
http://stackoverflow.com/questions/20791351/java-lang-nosuchmethoderror-com-google-common-util-concurrent-moreexecutors-sam

Check your classpath and see what version of guava is being used by your WAR. The error suggests that the version of guava jar being found at runtime does not match the version that was used at compile time.

简单一句话总结就是,jstorm集群上的guava版本和我们写的公共jar包里面的guava版本不一致.然后我去集群上的lib包上看了一下,发现jstorm 2.1.1用的guava是16.01版本的，于是我把集群上所有的机器里面的guava换成了20.0的，然后重启了集群,然后重新提交任务，果然就好使了,从kafka读取日志以及写入es也没问题了，任务终于不报错了.
