title: 'Hive count(*)和count(expr)'
date: 2015-11-11 00:15:42
tags: [Hive,SQL]
categories: Hive笔记
---
最近在给PM做报表时，要统计登陆订单的下单比例，也就是说哪些订单是登陆用户的订单，那些没有用户名，即用户名为`null`的用户不需要统计到最后的登陆订单中。
一开始我想这直接在`count`函数内写一个条件判断:
```sql
select count(case when user_name is not null then 1 else null end) as login_order
from table_name
```
但是后来我在查询Hive里count函数的用法时，发现了网上有人说，其实count在统计行数的时候，会根据使用的用法进行不同的逻辑运算，具体看Hive参考文档：
```
count(*) - Returns the total number of retrieved rows, including rows containing NULL values;
count(expr) - Returns the number of rows for which the supplied expression is non-NULL;
count(DISTINCT expr[, expr]) - Returns the number of rows for which the supplied expression(s) are unique and non-NULL.
```
上面的意思简单明了，除了`count(*)`之外，其他的计算的是非空值的条数，并且加上`distinct`还会合并重复记录到一类里面。
看到这里，我才发现，上面的写法有些多余，可以直接就count(user_name)，会自动略过那些user_name为空的记录。但是后面我还是采用了这个多余的写法，为什么？其实还是出于可读性和可维护性，这样写虽然复杂，但是后来维护的人起码知道你没有统计null用户。

### Hive count高级用法
count不是简单只有`count(*)`用法，下面还有一些更为高级的用法，加上条件语句:
```sql
select type,
	count(*),
	count(distinct u),
	count(case when plat=1 then u else null end),
	count(distinct case when plat=1 then u else null end),
	count(case when (type=2 or type=6) then u else null end),
	count(distinct case when (type=2 or type=6) then u else null end)
from t
where dt in ('2015-11-01', '2015-11-01)
group by type
order by type;
```
