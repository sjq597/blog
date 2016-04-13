title: 'Hive:SemanticException [Error 10025]: Line 1:7 Expression not in GROUP BY key'
date: 2015-11-11 00:46:26
tags: [Hive,SQL]
categories: Hive笔记
---
在Hive表里同时用`group by`和`order by`的时候，出现了错误：
```bash
FAILED: SemanticException [Error 10025]: Line 1:7 Expression not in GROUP BY key 'sys_code'
```
具体的SQL语句如下：
```sql
select sys_code,status, count(*) as total from ap_user_order_his
where link_end_date='2099-12-31' and sys_code in (1000,1001,1003,1004)
group by status
order by total desc;
```
后来上网查了一下，这个主要问题还是，`sys_code`也有多个种类，如果只是对`status`进行`group by`。那么`sys_code`没有办法分类，不知道该如何排序，所以解决办法有两种：

* 既然按某一个字段分类，那么其他字段也只能有一种情况，所以可以使用集合，针对上面的这个语句就是：
```sql
select collect_set(sys_code),status, count(*) as total from ap_user_order_his
where link_end_date='2099-12-31' and sys_code in (1000,1001,1003,1004)
group by status
order by total desc;
```
或者可以只取一个值：
```sql
select collect_set(sys_code)[0],status, count(*) as total from ap_user_order_his
where link_end_date='2099-12-31' and sys_code in (1000,1001,1003,1004)
group by status
order by total desc;
```

* 第二种就比较简单了，直接报错的字段也加到`GROUP BY`选项里即可。
