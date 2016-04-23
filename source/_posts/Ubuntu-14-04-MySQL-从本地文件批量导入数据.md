title: Ubuntu 14.04 MySQL 从本地文件批量导入数据
date: 2015-11-22 11:10:26
tags: [SQL,Linux]
categories: SQL
---
想做几个数据测试，于是把服务器上的`mysql`数据查询结果导入到服务器本地一个文件中，然后下载到本地机器想直接导入到自己机器的mysql表中:
假设我的MySQL用户名和密码分别为:root root123456,对应的库为test,表为user
```bash
# 下载的文件为result.txt,目录为当前用户根目录,即~/result.txt

# 进入mysql
$ mysql -uroot -proot123456

# 执行导入命令
mysql> use test;
mysql> load data local infile '~/result.txt' into table user;
```
结果报错，信息如下：
```
ERROR 1148 (42000): The used command is not allowed with this MySQL version
```
**备注:**网上查了一下：如果指定`local`关键词，则表明从客户主机读文件。如果`local`没指定，文件必须位于服务器上。既然是在本地机器上测试，那么我们也是服务器，所以我们可以去掉这个`local`参数，直接导入,结果又报新的错误：
```
mysql> load data infile '~/result.txt' into table user;
ERROR 13 (HY000): Can't get stat of '/var/lib/mysql/test/result.txt' (Errcode: 2)
```
在我们指定的路径处`var/lib/mysql/test/result.txt`没有找到这个文件,原来默认是在***当前数据库**(此处为我们的`test`库)目录下载入文件。我的文件放在当前用户的根目录下，当然会找不到，于是我把文件拷贝到这个目录下：
```bash
$ sudo cp ~/result.txt /var/lib/mysql/test
```
这回应该可以导入了吧，再次执行导入：
```sql
mysql> load data infile '~/result.txt' into table user;
ERROR 13 (HY000): Can't get stat of '/var/lib/mysql/user/result.txt' (Errcode: 2)
```
还是报错，奇怪了，难道文件没有拷贝到我们指定的目录？为了一探究竟，我决定去这个目录下看看
```bash
$ cd /var/lib/mysql
cd: permission denied: /var/lib/mysql
```
访问权限受限了，这个简单，加上就行：
```bash
$ sudo chmod 777 mysql
$ cd mysql
```
没问题了，然后进入`test`目录，果然还是一样没权限，同样处理，然后进入`test`目录，居然有我们的`result.txt`文件。然后重新在mysql里执行导入命令：
```sql
mysql> load data infile '~/result.txt' into table user;
Query OK, 44173 rows affected (0.22 sec)
Records: 44173  Deleted: 0  Skipped: 0  Warnings: 0
```
我擦，居然成功了，看来果然是权限的问题,虽然有点儿麻烦，但是可以导入数据就行。有时间再研究一下加`local`的用法。

其实不用这么麻烦，主要要先赋权限，然后再加一个参数就可以了:
```
$ sudo chown -R zhangsan:zhangsan /var/lib/mysql 
$ sudo chown -R mysql:mysql /var/lib/mysql
$ sudo mysql -uroot --local-infile=1 -proot123456
mysql> use test;
mysql> LOAD DATA LOCAL INFILE '/home/zhangsan/result.txt' REPLACE INTO TABLE oc_daily_order_cheche FIELDS TERMINATED BY '\t';
```
所以其实主要问题在一个`--local-infile=1`以及一个权限问题。
