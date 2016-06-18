title: Hive 正则
date: 2016-05-29 17:53:35
tags: Hive
categories: Hive笔记
---
分析日志经常要用到正则来提取需要分析的关键信息,其中需要注意的就是,Hive本身史用Java写的，所以HQL里面的正则和Java里面史一样的，但是具体在使用的时候，会碰到很多问题，就是转义的问题，总结了一下在各个语言中使用HQL时正则的不同情况。

### Hive Cli
平时使用的时候，很多情况下都是在Hive Cli客户端中使用，这里和正常的正则需要稍微有一些区别,例如匹配数字，正常情况下就`\d`，但是在Hive Cli中需要注意,应该使用`\\d`。
```
select
  regexp_extract(content, '.*id=(\\d*).*', 1) as id 
from
  test.table_test;
```
  
### Shell
还有一种更为常见的使用场景，就是在Shell脚本中使用，Shell脚本中又涉及到单引号和双引号的区别,单引号史强引用类型,`\`不会被转义，但是双引号字符串则会转义，例如:
```
reg_str1='.*id=(\\d*).*'
reg_str2=".*id=(\\\\d*).*"

sudo -uhiev_user /usr/dev/hive-1.2.0/bin/hive -e "
select
  regexp_extract(content, '${reg_str1}', 1) as id1, 
  regexp_extract(content, '${reg_str2}', 1) as id2 
from
  test.table_test;
"
```
这两个正则表达式是一样的，其实确认你的HQL正则表达式是否有用，可以先用echo在sudo前面看看，如果输出的史`\\d`则说明有用

### Python
有时候需要在Python中执行HQL,好像Hive并不支持Python接口,所以我采取的是在Python中执行Shel命令
```python
# bash_cmd为Shell命令
bash_cmd = """
sudo -uhive_user /usr/dev/hive-1.2.0/bin/hive -e "{0}"
""".format(sql)	# sql即为要执行的HQL

hql_process = subprocess.Popen(bash_cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
```
在Python中我定义来一个sql
```python
sql = """
select
  regexp_extract(content, '.*id=(\\\\\\d*).*', 1) as id 
from
  test.table_test;
"""
```
