title: Hive 常用命令 01
date: 2015-10-13 16:46:05
tags: [Hive,大数据]
categories: Hive笔记
---
最近在进行数据迁移，把常用的`Hive`命令整理一下。

### 导入分区数据
把一个`Hive`表中的数据导入到另一个表中，但是这两个表都是有分区的，需要动态导入，假设想把表`table_a`的数据导入到表`table_b`。这两个表都是按照时间分区的，例如表A的建表语句如下：
```sql
CREATE TABLE table_a (
  id int,
  name string comment '姓名',)
comment '表A'
PARTITIONED BY (dt string)
ROW format delimited fields terminated by '\;'
lines terminated by '\n'
stored as textfile;
```
现在新建一个表B,建表语句和表A完全一样，把表A的数据导入到表B，可以这么做：
```sql
INSERT OVERWRITE TABLE table_b partition(dt)
select * from table_a
```

### 终端命令

1. 登陆Hive
```bash
sudo -uuser_name /home/q/hive/hive-0.12.0/bin/hive -database database_name
```

### 修改命令

1. 重命名表名
```sql
alter table table_name rename to new_table_name
```
2. 重命名分区
```sql
alter table table_name partition(dt='20151014') rename to partition(dt='20151014old');
```
3. 删除表
```sql
drop table if exists table_name;
```
4. 删除/添加分区
```sql
alter table table_name drop partition(dt='20151014');
alter table table_name add if not exists partition(dt='20151018');
```
5. 清空表数据
```sql
insert overwrite table table_name select * from table_name where 1=0;
```
6. 删除指定条件的数据
```sql
# 把Hive表中link_end_date字段为'2099-12-31'的数据删掉,即只要不等于这个值就再插回源表中
insert overwrite table ap_fuwu_tb_complaint_his
select * from ap_fuwu_tb_complaint_his where
link_end_date<>'2099-12-31';
```
7. 将数据插入到指定分区
```sql
alter table table_name add if not exists partition(dt='20151021old');
insert overwrite table table_name partition(dt='20151021old')
SELECT * FROM table_name WHERE dt='20151021';
```
**注意：** 如果列不等，则把`*`换成对应的列。
8. 重命名列名
```sql
alter table table_name CHANGE old_col_name new_col_name field_type;
```
9. 改变列顺序
```sql
# 把old_col_name改名为new_col_name并且把这列放在another_col_name列后面
alter table table_name CHANGE old_col_name new_col_name field_type after another_col_name;
```
10. 复制表
```sql
create table new_table like old_table;
```

### 展示信息命令

1. 展示建表语句
```sql
show create table table_name;
```
2. 展示表详情
```sql
desc table_name;
```
3. 模糊查询
使用`like`可以进行模糊查询使用`like`可以进行模糊查询,,其中`_`表示单个字符,`%`表示任意数量的字符.要注意如果是用否定,语法是:
```sql
# 语法类似下面这样
select * from table where NOT ‘key' like 'fff%'; 
```
**RLIKE**:
```bash
# 字符串a符合java正则表达式b的正则语法则返回true
# 语法:A rlike b
hive> select 1 from tabe_name where 'footbar’ rlike '^f.*r$’;
1
```
还有个和`rlike`功能一样的操作:`REGEXP`
```bash
hive> select 1 from table_name where 'footbar' REGEXP '^f.*r$';
1
```
