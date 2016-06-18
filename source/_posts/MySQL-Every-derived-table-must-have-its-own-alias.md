title: 'MySQL:Every derived table must have its own alias'
date: 2015-11-11 00:38:14
tags: [SQL]
categories: MySQL
---
在做多表联合查询时，报错：
> Every derived table must have its own alias

意思就是每个派生出来的表都必须有一个自己的别名，这个错误一般多出现在多表查询的时候。
之所以在多表查询的时候会出现这个问题，是因为多表在做联合查询的时候，有嵌套查询。子查询出来的结果是作为一个派生表来进行上一级的查询，所以子查询的结果必须要有一个别名，按照下面类似的改法：
```sql
select count(*) from (
select * from table_name) as t2;
```
也就是为每个子查询一个别名，其实加一个简写，也方便多个表作联合查询条件来作联合。
