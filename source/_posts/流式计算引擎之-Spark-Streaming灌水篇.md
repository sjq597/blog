title: 流式计算引擎之-Spark Streaming灌水篇
date: 2020-05-18 22:44:52
tags: [Spark, 流计算, 大数据]
categories: 数据架构
---
### 废话
一直想写一篇关于`Spark Streaming`的文章，但是实在是事情太多，当然主要还是我比较懒，身边的同事已经明示我博客好久都不更新来催更了，技术都快荒废了，所以先水一篇。这篇文章虽然是叫灌水篇，但实际上都是我在实际项目中学习和总结到的一些切身经验，如果你真的想在生产环境去用Spark Streaming实现一些简单的实时计算或者实时监控，看板之类的，有一些问题是你一开始就得考虑到的，并且得有非常可靠的解决方案。

算起来差不多有快两年没用过`Spark Streaming`了，毕竟现在的公司也是主推`Flink`了，从趋势来看，的确是`Flink`的势头更猛一些，虽说`Spark Streaming`后来引入`Structed Streaming`，据说性能提升不少,这个我也没用过，暂时就不做深入讨论了。

其实这个系列计划后面还有一篇`JStorm`，一篇`Flink`，之所以把`Spark Streaming`放在第一篇来讲主要是有两个原因：首先是这个框架算是在实际做项目中钻研过一段时间，对其使用中的坑也算踩了不少，最后也算圆满完成了项目；其次是之前在招人的时候面试过很多人，简历上写了做过的一些`Spark Streaming`项目，在考察他们项目的时候，问的深了慢慢就懵了，从来没有碰到过一个人有认真思考过那些问题，当然这里面有很多人的简历大概率是培训机构的一些模板(数仓分层 + Spark Streaming + 用户画像，基本是这个结构，技术选型一模一样,另外也吐槽下那些培训机构，你告诉别人用户画像放在`HBase`这没啥问题，问题是你总得好好讲下`rowkey`这个吧，连怎么设计，为什么要这么设计也不讲讲，一问就完蛋)，本身也没真正做过`Spark Streaming`的项目，对框架的一些设计理念和底层的东西更加没有了解过。

### 正文
所以这篇文章主要会说一下一些实际项目中比较关注的问题，这些问题大概率也是面试官比较喜欢问的，做实时计算的确也绕不开那几个问题,好久没用了，想到什么写什么，截图是肯定没有了,但是如果你看到这篇文章，又或者在找解决方案，我想没有图应该也看得懂,下面就讲几个比较重要的问题。

#### Spark Streaming会丢数据吗
相信每个做实时计算的人都会碰到这个问题，或者说被别人追问过这个问题:为什么我从xxx能查到数据，从你实时处理之后就查不到了?又或者是这段时间的指标看着好像有问题，线上实时计算程序没漏掉数据吧？如果你非常自信的告诉别人，不可能丢的，这个`Spark Streaming`官网说了，xxx和xxx机制可以保证即使程序挂了，也能恢复，数据不会丢的。但是实际情况会比较复杂，很可能会被立马打脸，或者说运气好，丢了一两条也没人看的出来，但是其实有一些项目是连一条数据都不能丢的。

这里先说下结论，其实`Spark Streaming`是会丢数据的，要想保证数据100%不丢失，你需要根据实际情况，也就是输入输出的存储系统来做很多的处理，否则，丢数据的情况会有很多种，丢不丢数取决于你的集群和系统的稳定性。

所以每次碰到简历上写着`Spark Streaming`项目的人，我的第一个问题就是：你这个程序会丢数据吗？他们一开始都会很确定的告诉我不会，并且会开始讲自己是通过`Direct`的方式去消费`Kafka`的数据，手动维护`offset`的，只有当数据处理成功了才会提交offset,所以即使程序挂了，重启之后还是会从之前的offset开始消费，数据不会丢。当然有时候也会顺带问下他们`Receiver`模式和`Direct`的区别和为什么选后者，算是比较基本的问题，但是有的培训机构居然连这个也没讲明白。很多人都认为是不能手动维护`offset`，所以有丢数的风险,其实主要原因并不是这个，这里就不展开讲了。

但是我们面临的实际问题是，手动维护`offset`,确保每次处理成功了才提交`offset`就真的不会丢数据吗？从表面上来看，这个想法确实没什么问题，但是如果你对`Spark Sgreaming`了解的够深，就会发现这个想法是错的。其实要想知道会不会丢数，就得稍微了解一下底层的东西，或者说多看看监控和SparkUI界面的信息，了解一下`Application`,`Job`,`Task`,`Batch`这些概念，如果你把这几个东西搞清楚了，我相信你应该可以大致判断出来你写的程序是否有丢数据的风险以及在什么情况下会丢数据。

##### 介绍
正式讲问题之前，先得看几个基本概念

名称 | 说明
----|-------
Application | 应用程序，也就是你每次提交一个Spark Streaming任务的时候，运行在集群上的一个计算任务
Batch | `Spark Streaming`核心理念就是`micro batch`也就是算子和程序并不是时时刻刻都在计算和处理数据的，而是每隔一段时间成成一个批次
Job | 一个计算任务可能会包含多个Job,大部分应该就是一个
Task | 任务最终执行计算最小单位，对于`source`是`kafka`的情况，总的数量一般和需要消费的分区数相等，最大同时执行`task`个数可以通过一些参数来设置

所以其实是每次开始调度执行一个Batch，会生成job，然后job会分stage,最细粒度就是到task执行具体的逻辑。因为我写`Spark Streaming`程序中也没见过有两个job的，所以暂且以只有一个job的程序为例，讨论下什么场景下会丢数据以及深层次的原因，下面主要介绍下`Task`和`Batch`：

1. Task 
作为最终消费和处理数据的执行单元，也就是你程序里面真正处理数据的算子和最终的`action`操作，对于手动维护`offset`来说，一般就是在所有数据处理完，就调用Kafka的api来`commit offset`，这样就确保了如果程序出现异常情况挂掉，`offset`不会更新，当程序再次重启，会从zk上读取消费的信息，从上一次最后提交的`offset`后开始消费数据。这也就是大家普遍认为不会丢数的依据;
2. Batch
一般会在程序中设置批次的间隔，也就是多长时间生成一个批次，这个取决于实际场景，假设是5min一个批次，所以每隔5分钟程序会自动生成新的批次，然后根据资源情况和上一个批次的执行情况来决定是否开始调度此批次。需要说明下，大部分情况，也就是默认情况下，同时执行的批次只能是一个，也就是如果数据太多导致上一个批次没有执行完，后面生成的批次就会被pending住。并且默认的调度算法是FIFO，所以基本上就是按数据的消费顺序来处理数据;
3. 异常
可以看到其实整个`Spark Streaming`程序从表格上来看，从上到下是一个任务的逐渐细化过程，所以问题来了，计算任务失败是如何定义的,`Task`失败了`Job`会失败吗？`Batch`会失败吗？`Application`会失败吗？
 * Task会有默认的重试次数，好像是4次，可能不同的平台会额外设置，这个参数是可以设置的。Task执行失败了会自动重试，如果超过了最大重试次数还是执行失败，那这个Task所在的Job就失败了,所以当前批次的状态就是`Failed`；
 * Job失败了就失败了，因为Task已经重试过了仍然失败，最终这个`Batch`失败了, 但是`Application`并不会失败，一般而言，Task重试了很多次，会耗费很多时间，所以会有pending状态的Batch，这个时候其实会直接调用下一个pending的批次继续执行。

##### 案例分析
所以到这里你就应该明白了为什么说，即使你使用直连方式消费kafka，手动维护offset,仍然会丢数据。这里还需要再额外讲一下，每个批次在生成之前，就会计算出这个批次当前需要消费的数据范围，也就是[offset1,offset2]这种，对应到kafka的每个分区，哪怕你上一个batch还没执行完，下一个批次他会算出他此时需要消费的数据。至于怎么算的，有时间后面展开说，总之不是每次开始执行才去zk里面读取最新offset开始消费，这里记住就行了，因为和丢数有关。

上面讲到了每个批次要处理的数据其实是根据以往任务执行的情况来估算出来的，假设第一个批次在执行过程中由于在不停的重试，时间超过了一个批次，后面又生成了几个批次，并且由于同时最多只能有一个batch在执行，其他的都在pending状态。这个时候当Task的失败重试次数超过了设置的最大失败重试次数，即最终还是失败了，于是第一个批次就挂了。并且由于程序设置的是执行成功之后才会commit offset,导致偏移量也没有提交，到此为止还算正常毕竟虽然失败了，但是offset也没有更新，如果重启的话，还是会从上一次成功的地方接着消费。

但事实是紧接着由于第一个批次失败了，资源空闲出来了，后面pending的第一个批次就开始调度了，然后呢比较顺利，马上就执行成功了，这个时候程序触发了`commit offset`,将最新的消费情况更新到了zk。依次类推，后面的批次按照之前算好的消息范围继续消费，成功后`commit offset`。于是在执行成功了几个批次之后，突然发现，第一个批次失败了，offset被第二个和后面的批次更新覆盖了，然而实际上消息并没有被消费处理，因为第二个批次处理的消息是提前算好的，这就是数据丢失的真实场景。

##### 解决方案
要怎么做才可以保证数据不丢失呢？其实通过上面的分析，会导致丢数据的根本原因其实是因为Batch失败了，但是Application并没有失败，后续执行的Batch成功了把offset给覆盖了。所以解决问题的方法其实有两种:
1. 不让Batch失败，这样就不会存在前面的批次失败了，后面的批次成功了这种情况。但是实际情况下总会存在程序出异常的情况，所以可以在程序出问题的时候将程序hang住，比如将Task的最大失败重试次数设置成int最大值。不过这样也有问题，一是如果就是程序的确就有问题，重试也不会成功，白白重试浪费资源；另外就是如果程序计算很快，那么重试也会有试完的时候，然后开始调度下一个`Batch`，所以并不推荐这种方式；
2. `Batch`失败了让`Application`也失败，根本原因其实是`Batch`失败了`Application`没有失败，继续调度后续`Batch`导致`offset`被覆盖。这里需要借助因为第二个批次处理的消息是提前算好的，

这里需要借助下`StreamingListener`这个类，你需要继承这个接口，来监听Job的执行状态，从而来控制当Job失败了，程序直接重启，不要直接调度下一个`Batch`。
```scala
class Batch extends Serializable {
  var failedCnt: Long = 0
  var successCnt: Long = 0
}

class BasedataSparkBatchListener(ssc: StreamingContext) extends StreamingListener {
  val batch = new Batch()
  /** Called when a batch of jobs has been submitted for processing. */
  def onBatchSubmitted(batchSubmitted: StreamingListenerBatchSubmitted) { }

  /** Called when processing of a batch of jobs has started.  */
  def onBatchStarted(batchStarted: StreamingListenerBatchStarted) { }

  /** Called when processing of a batch of jobs has completed. */
  def onBatchCompleted(batchCompleted: StreamingListenerBatchCompleted) { 
   val batchInfo = batchCompleted.batchInfo
    val outputOperations = batchInfo.outputOperationInfos
    val numFailedOutputOp = outputOperations.values.count(_.failureReason.nonEmpty)
 
    if (numFailedOutputOp != 0) {
      batch.failedCnt += 1
    } else {
      batch.failedCnt += 1
    }
  }
 }
```
可以看到有很多方法，这里其实只需要实现`onBatchCompleted`方法获取失败的Batch数量就行,然后主程序需要注册下这个`listener`:
```
val listener = new BasedataSparkBatchListener(ssc)
ssc.addStreamingListener(listener)
```
然后在提交offset的地方需要做一个判断，如果是`listener.batch.failedCnt > 0`，执行`ssc.stop()`将程序杀掉，如果`listener.batch.failedCnt = 0`,则执行`commit offset`操作。

#### 限速和背压设置

#### Exactly once真的必要吗


#### Speculative机制有什么问题
