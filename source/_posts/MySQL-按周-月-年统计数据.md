title: 'MySQL 按周,月,年统计数据'
date: 2015-12-07 22:14:27
tags: [SQL, Shell]
categories: MySQL
---
通常对于一个标准的时间,例如`2015-12-05 12:34:40`,如果想获取它的年月日,我们可以采取取字串的形式,即:
```
substr(create_time,1,10)	# 取2015-12-05
substr(create_time,1,7)		# 取2015-12
substr(create_time,1,4)		# 取2015
```
也确实,对于一般的需求这么处理也够用了,但是有一个问题,如果要按周来统计,这个取字串就无法做到了,这个时候就需要用到`mysql`内置函数:
```
date_format(date,format)
```
如果要按星期,天,月份来统计数据,可以这么来写,假设时间字段是`create_time`.
```sql
SELECT date_format(create_time,'%y%u') week,count(id) FROM table GROUP BY week;		# 按星期统计
SELECT date_format(create_time,'%y%m%d') day,count(id) FROM table GROUP BY day;		# 按天统计
select date_format(create_time,'%y%m') month,count(id) FROM table GROUP BY month;	# 按月统计
```

详细参数:
根据`format`字符串格式化代`date`的值.具体的格式取值有:

format 取值 | 含义
------------|-------
%M	| 	月名字(January...December)
%W	|	星期名字(Sunday...Saturday)
%D	|	有英语前缀的月份的日期(1st, 2nd, 3rd, 等等。）
%Y	|	年, 数字,4 位(2013,2014,2015)
%y	|	年, 数字, 2 位(13,14,15)
%a	|	缩写的星期名字(Sun...Sat)
%d	|	月份中的天数, 数字(00...31)
%e	|	月份中的天数, 数字(0...31)
%m	|	月, 数字(01...12)
%c	|	月, 数字(1...12)
%b	|	缩写的月份名字(Jan...Dec)
%j 	|	一年中的天数(001...366)
%H 	|	小时(00...23)
%k 	|	小时(0...23)
%h 	|	小时(01...12)
%I 	|	小时(01...12)
%l 	|	小时(1...12)
%i 	|	分钟, 数字(00...59)
%r 	|	时间,12 小时(hh:mm:ss [AP]M)
%T 	|	时间,24 小时(hh:mm:ss)
%S 	|	秒(00...59)
%s 	|	秒(00...59)
%p 	|	AM或PM
%w 	|	一个星期中的天数(0=Sunday...6=Saturday ）
%U 	|	星期(0...52), 这里星期天是星期的第一天
%u 	|	星期(0...52), 这里星期一是星期的第一天
%% 	|	一个文字“%”。
