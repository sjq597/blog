title: MySQL过滤中文
date: 2016-04-23 13:36:48
tags: MySQL
categories: SQL
---

有时候在统计的时候，需要把包含中文的列过滤掉，主要是存在历史脏数据，比如我们的客户端存在新老客户端之分，日志记录不一样，所有有些活动或者名字在pv,uv的时候不一致，例如活动的名字一般就算有英文也不可能全是英文，所以我们在计算的时候，需要把全为英文的过滤掉:
```
where LENGTH(`col_name`) <> CHARACTER_LENGTH(`col_name`) 
```
这里的`col_name`是mysql里表的列名
