title: Python fetchall()字段特殊字符过滤处理
date: 2015-11-01 10:50:04
tags: [Python,SQL]
categories: [Python笔记]
---
最近在做数据仓库的迁移工作,之前数据仓库的数据都是用的shell脚本来抽取,后来换了python脚本.
但是在把数据抽取存放到hadoop时,出现了一个问题:
由于数据库字段很多,提前也不知道数据库字段会存储什么内容,hive建表是以`\t\n`做分隔,这就导致了一个问题,如果mysql字段内容里面本身含有`\t\n`,那么就会出现字段错位情况,并且很头疼的是mysql有100多个字段,也不知道哪个字段会出现这个问题.
shell脚本里的做法是在需要抽取的字段上用mysql的replace函数对字段进行替换,例如,假设mysql里的字段是`column1 varchar(2000)`,那么很可能就会出现有特殊字符的情况,在查询的sql语句里加上
```sql
select replace(replace(replace(column1,'\r',''),'\n',''),'\t','')
```
之前一直是这么干的,但是这样写sql特别长,特别是有100多个字段,也不知道哪个有特殊字符,只要都加上.
所以在python中对字段不加处理,最终导致hive表字段对应出现偏差,所以在python里从mysql查询到的字段在写到文件之前需要对每个字段进行过滤处理

看个例子,我就以mysql测试为例,首先建一张测试表
```sql
CREATE TABLE `filter_fields` (
  `field1` varchar(50) DEFAULT NULL,
  `field2` varchar(50) DEFAULT NULL,
  `field3` varchar(50) DEFAULT NULL,
  `field4` varchar(50) DEFAULT NULL,
  `field5` varchar(50) DEFAULT NULL,
  `field6` varchar(50) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
有六个字段,都是`varchar`类型,插入新数据可以在里面插入特殊字符.简单插入条数据测试看看:
```sql
insert into filter_fields(field1,field2,field3,field4,field5,field6) VALUES
('test01','test02','test03','test04','test05','test06');
insert into filter_fields(field1,field2,field3,field4,field5,field6) VALUES
('test11\ntest11','test12\n\n','test13','test14','test15','test16');
insert into filter_fields(field1,field2,field3,field4,field5,field6) VALUES
('test21\ttest21','test22\ttest22\ttest22','test23\t\t\t','test4','test5','test6');
insert into filter_fields(field1,field2,field3,field4,field5,field6) VALUES
('test21\rest21','test22\r\rest22\r\rest22','test23\r\r\r','test4','test5','test6');
```
其中数据里插入的特殊字符,可能连在一起,也有不连在一起的.

python测试代码:
```python
# coding=utf-8

import MySQLdb
import sys

db_host = '127.0.0.1'   # 数据库地址
db_port = 3306          # 数据库端口
db_user = 'root'        # mysql用户名
db_pwd = 'yourpassword' # mysql用户密码,换成你的密码
db_name = 'test'        # 数据库名
db_table = 'filter_fields'  # 数据库表

# 过滤sql字段结果中的\t\n
def extract_data(table_name):
    try:
        conn = MySQLdb.connect(host=db_host, port = db_port, user=db_user,
                               passwd = db_pwd, db = db_name, charset = "utf8")
        cursor = conn.cursor()
    except MySQLdb.Error, e:
        print '数据库连接异常'
        sys.exit(1)

    try:
        sql = 'select * from %s;'%(table_name)
        cursor.execute(sql)
        rows = cursor.fetchall()

        print '====字段未过滤查询结果===='
        for row in rows:
            print row

        print '====字段过滤之后结果===='
        rows_list = []
        for row in rows:
            row_list = []
            for column in row:
                row_list.append(column.replace('\t', '').replace('\n', '').replace('\r', ''))
            rows_list.append(row_list)
            print rows_list[-1]		# [-1]表示列表最后一个元素
        return rows_list
    except MySQLdb.Error, e:
        print '执行sql语句失败'
        cursor.close()
        conn.close()
        sys.exit(1)

if __name__ == '__main__':
    print 'begin:'
    rows = extract_data(db_table)
    pass
```
看看输出结果:
```
====字段未过滤查询结果====
(u'test01', u'test02', u'test03', u'test04', u'test05', u'test06')
(u'test11\ntest11', u'test12\n\n', u'test13', u'test14', u'test15', u'test16')
(u'test21\ttest21', u'test22\ttest22\ttest22', u'test23\t\t\t', u'test4', u'test5', u'test6')
(u'test21\rest21', u'test22\r\rest22\r\rest22', u'test23\r\r\r', u'test4', u'test5', u'test6')
====字段过滤之后结果====
[u'test01', u'test02', u'test03', u'test04', u'test05', u'test06']
[u'test11test11', u'test12', u'test13', u'test14', u'test15', u'test16']
[u'test21test21', u'test22test22test22', u'test23', u'test4', u'test5', u'test6']
[u'test21est21', u'test22est22est22', u'test23', u'test4', u'test5', u'test6']
```
可以看到,制表符,换行符,回车都被过滤了.
**建议:**最后说点题外话,不要小视`\r`,回车符.很多人以为回车符就是换行符,其实不是的,`\r`表示回车符,`\n`表示新行.之前代码里其实是过滤掉了`\t\n`的,但是抽取的数据还是不对,后来看了源码之后才发现,原来是没有过滤`\r`,就这个不同导致了很多数据抽取不对.
