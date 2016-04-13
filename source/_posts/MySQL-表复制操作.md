title: MySQL 表复制操作
date: 2015-11-09 15:29:36
tags: [SQL]
categories: SQL
---
在操作数据库的时候,往往是保留想要修改的库,先建一个一样的来操作,例如我想给一个库加个字段,然后从另一个很复杂的sql语句导入数据,看字段添加是否正确,最好的做法就是自己建一张临时表,把查询结果插到临时表,这样一旦出错,也不会影响到正常数据.下面是一些涉及到表复制和修改的常用命令.

### MySql表复制
这个分为表是存在和不存在两种,具体使用不同的语句.

#### 新表不存在
* 复制表结构即数据到新表

```sql
create table new_table
select * from old_talbe;
```
 这种方法会将`old_table`中所有的内容都拷贝过来,用这种方法需要注意,`new_table`中没有了`old_table`中的`primary key,Extra,auto_increment`等属性,需要自己手动加,具体参看后面的修改表即字段属性.

* 只复制表结构到新表

```sql
# 第一种方法,和上面类似,只是数据记录为空,即给一个false条件
create table new_table
select * from old_table where 1=2;

# 第二种方法
create table new_table like old_table;
```

#### 新表存在
* 复制旧表数据到新表(假设两个表结构一样)

```sql
insert into new_table
select * from old_table;
```

* 复制旧表数据到新表(假设两个表结构不一样)

```sql
insert into new_table(field1,field2,.....)
select field1,field2,field3 from old_table;
```

* 复制全部数据

```sql
select * into new_table from old_table;
```

* 只复制表结构到新表

```sql
select * into new_talble from old_table where 1=2;
```

### MySql修改命令

* 增加字段

```sql
alter table table_name add column column_type [other];
```
 例如:
```sql
alter table user_info add mobile_phone varchar(30) default '0';
```

* 增加索引

```sql
alter table table_name add index index_name (field1[，field2 …]);
```
 用法样例:
```sql
alter table user_info add index idx_user_no(user_no);
```

* 加主关键索引

```sql
alter table table_name add primary key(field);
```
 用法举例:
```sql
alter table table_name add primary key(id,user_no);
```

* 加唯一限制条件索引

```sql
alter table table_name add unique idx_name(field);
```
 用法举例:
```sql
alter table user_info add unique idx_user_name(user_name);
```

* 删除索引

```sql
alter table table_name drop index idx_name;
```

* 修改字段名称或类型

```sql
alter table table_name change old_field_name new_field_name field_type;
```

* 删除字段

```sql
alter table table_name drop field_name;
```
