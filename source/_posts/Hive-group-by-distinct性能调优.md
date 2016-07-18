title: Hive group by distinct性能调优
date: 2016-04-24 17:51:15
tags: Hive
categories: Hive笔记
---
### Hive去重统计
相信使用Hive的人平时会经常用到去重统计之类的吧，但是好像平时很少关注这个去重的性能问题，但是当一个表的数据量非常大的时候，会发现一个简单的`count(distinct order_no)`这种语句跑的特别慢，和直接运行`count(order_no)`的时间差了很多，于是研究了一下。
**先说结论:**能使用`group by`代替`distinc`就不要使用`distinct`，例子：
### 实际论证
order_snap为订单的快照表 总记录条数763191489，即将近8亿条记录,总大小:108.877GB,存储的是公司所有的订单信息，表的字段大概有20个,其中订单号是没有重复的,所以在统计总共有多少订单号的时候去重不去重结果都一样，我们来看看:
统计所有的订单有多少条条数，一个`count`函数就可以搞定的sql性能如何。
* DISTINCT

```
select count(distinct order_no) from order_snap;
Stage-Stage-1: Map: 396  Reduce: 1   Cumulative CPU: 7915.67 sec   HDFS Read: 119072894175 HDFS Write: 10 SUCCESS
Total MapReduce CPU Time Spent: 0 days 2 hours 11 minutes 55 seconds 670 msec
OK
_c0
763191489
Time taken: 1818.864 seconds, Fetched: 1 row(s)
```

* GROUP BY

```
select count(t.order_no) from (select order_no from order_snap group by order_no) t;
Stage-Stage-1: Map: 396  Reduce: 457   Cumulative CPU: 10056.7 sec   HDFS Read: 119074266583 HDFS Write: 53469 SUCCESS
Stage-Stage-2: Map: 177  Reduce: 1   Cumulative CPU: 280.22 sec   HDFS Read: 472596 HDFS Write: 10 SUCCESS
Total MapReduce CPU Time Spent: 0 days 2 hours 52 minutes 16 seconds 920 msec
OK
_c0
763191489
Time taken: 244.192 seconds, Fetched: 1 row(s)
```

**结论:**第二种写法的性能是第一种的`7.448499541`倍
注意到为什么会有这个差异，Hadoop其实就是处理大数据的，Hive并不怕数据有多大，怕的就是数据倾斜,我们看看两者的输出信息:
```
# distinct
Stage-Stage-1: Map: 396  Reduce: 1   Cumulative CPU: 7915.67 sec   HDFS Read: 119072894175 HDFS Write: 10 SUCCESS
# group by
Stage-Stage-1: Map: 396  Reduce: 457   Cumulative CPU: 10056.7 sec   HDFS Read: 119074266583 HDFS Write: 53469 SUCCESS

```
发现猫腻了没有，使用distinct会将所有的order_no都shuffle到一个reducer里面，这就是我们所说的数据倾斜，都倾斜到一个reducer这样性能能不低么？再看第二个，直接按订单号分组，起了457个`reducer`，将数据分布到多台机器上执行，时间当然快了.
由于没有手动指定Reduce的个数，Hive会根据数据的大小动态的指定Reduce大小，你也可以手动指定
```
hive> set mapred.reduce.tasks=100；
```
类似这样,所以如果数据量特别大的情况下，尽量不要使用`distinct`吧。
但是如果你想在一条语句里看总记录条数以及去重之后的记录条数，那没有办法过滤，所以你有两个选择，要么使用两个sql语句分别跑，然后union all或者就使用普通的distinct。具体来说得看具体情况，直接使用distinct可读性好，数据量如果不大的话推荐使用，如果数据太大了，性能受到影响了，再考虑优化。
