title: Python 连接数据库操作
date: 2015-11-07 23:24:03
tags: [Python,SQL]
categories: Python笔记
---
来一个简单的例子，看Python如何操作数据库，相比Java的JDBC来说，确实非常简单，省去了很多复杂的重复工作，只关心数据的获取与操作。

### 准备工作
需要有相应的环境和模块：

> Ubuntu 14.04 64bit
> Python 2.7.6
> MySQLdb

**注意:**Ubuntu 自带安装了Python，但是要使用Python连接数据库，还需要安装MySQLdb模块，安装方法也很简单：
```bash
sudo apt-get install MySQLdb
```
然后进入Python环境，import这个包，如果没有报错，则安装成功了：
```
~|⇒ python
Python 2.7.6 (default, Jun 22 2015, 17:58:13) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import MySQLdb
>>> 
```
Python标准的数据库接口的`Python DB-API`（包括Python操作MySQL）。大多数Python数据库接口坚持这个标准。不同的数据库也就需要不同额模块，由于我本机装的是MySQL，所以使用了`MySQLdb`模块，对不同的数据库而言，只需要更改底层实现了接口的模块，代码不需要改，这就是模块的作用。

### Python数据库操作
首先我们需要一个测试表
#### 建表语句：
```sql
CREATE DATABASE study;
use study;
DROP TABLE IF EXISTS python_demo;
CREATE TABLE python_demo (
  id int NOT NULL AUTO_INCREMENT COMMENT '主键，自增',
  user_no int NOT NULL COMMENT '用户编号',
  user_name VARBINARY(50) NOT NULL COMMENT '用户名',
  password VARBINARY(50) NOT NULL COMMENT '用户密码',
  remark VARBINARY(255) NOT NULL COMMENT '用户备注',
  PRIMARY KEY (id,user_no)
)ENGINE =innodb DEFAULT CHARSET = utf8 COMMENT '用户测试表';

INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1001,'张三01','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1002,'张三02','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1003,'张三03','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1004,'张三04','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1005,'张三05','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1006,'张三06','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1007,'张三07','admin','我是张三');
INSERT INTO python_demo(user_no, user_name, password, remark) VALUES
  (1008,'张三08','admin','我是张三');
```

#### Python代码
```python
# --coding=utf8--
import ConfigParser

import sys
import MySQLdb

def init_db():
    try:
        conn = MySQLdb.connect(host=conf.get('Database', 'host'),
                               user=conf.get('Database', 'user'),
                               passwd=conf.get('Database', 'passwd'),
                               db=conf.get('Database', 'db'),
                               charset='utf8')
        return conn
    except:
        print "Error:数据库连接错误"
        return None

def select_demo(conn, sql):
    try:
        cursor = conn.cursor()
        cursor.execute(sql)
        return cursor.fetchall()
    except:
        print "Error:数据库连接错误"
        return None

def update_demo():
    pass

def delete_demo():
    pass

def insert_demo():
    pass

if __name__ == '__main__':
    conf = ConfigParser.ConfigParser()
    conf.read('mysql.conf')
    conn = init_db()
    sql = "select * from %s" % conf.get('Database', 'table')
    data = select_demo(conn, sql)
    pass
```
