title: "Hive cannot recognize input near '<EOF>' '<EOF>' '<EOF>'"
date: 2015-11-17 12:07:10
tags: [SQL,Hive]
categories: Hive笔记
---
在Hive中复制表数据,把一个表的查询结果存储起来放到一个完整的表中,用到了
```sql
create table table1_name as select filed1[,field2][,field3...] from table2_name;
```
根据上面的语法,我想直接把一个表的查询结果作为`table2_name`,看看我的写法:
```sql
create table table1_name
select * from (
select substr(login_time,0,10) as day,count(user_name) as empty_user
from table2_name
where user_name='' or user_name is null
group by substr(login_time,0,10));
```
**备注:**`table2_name`表存放的是每天有登陆记录的用户,上面的语句其实是想统计每天用户名为空或者没有用户名的用户.
出现了报错信息:
```
FAILED: ParseException line 6:34 cannot recognize input near '<EOF>' '<EOF>' '<EOF>' in subquery source
```
意思是子查询的语法不对,语句的结尾不正确,后来仔细想了一下,想到别名这个sql里常用的语法,虽然表的别名本身并没有什么大用,但是在子查询中经常用到,而且没有别名没有时还会报错,于是把后面加了一个表别名,改为如下:
```sql
create table table1_name
select * from (
select substr(login_time,0,10) as day,count(user_name) as empty_user
from table2_name
where user_name='' or user_name is null
group by substr(login_time,0,10)) p1;
```
然后就好了.
