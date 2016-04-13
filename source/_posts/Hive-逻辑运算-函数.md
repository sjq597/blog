title: 'Hive 逻辑运算,函数'
date: 2015-11-05 14:05:57
tags: [大数据,Hive,SQL]
categories: Hive笔记
---
Hive自身提供了逻辑运算以及数学上的一些函数,基本和mysql
里差不多.简单的不再多说了，只记录一些常用的

## 函数
Hive自带的函数非常多,大致分为一下几块介绍,详细信息在[Hive Wiki官网](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)
### 数学函数

返回类型 | 函数 | 说明
---------|------|-------
bigint | round(double a) | 四舍五入
double | round(double a, int d) | 小数部分d位之后数字四舍五入
bigint | floor(double a) | 对给定数据进行向下舍入最接近的整数
bigint | ceil(double a)<br>ceiling(double a) | 将参数向上舍入为最接近的整数
double | rand()<br>rand(int seed) | 返回大于或等于0且小于1的平均分布随机数（依重新计算而变）
double | pow(double a, double p)<br>power(double a, double p) | 返回某数的乘幂
double | sqrt(double a) | 返回数值的平方根
int<br>double | positive(int a)<br>positive(double a) | 返回A的值
int<br>double | negative(int a)<br>negative(double a) | 返回A的相反数

### 日期函数

返回类型 | 函数 | 说明
---------|------|-------
string | from_unixtime(bigint unixtime[, string format]) | UNIX_TIMESTAMP参数表示返回一个值’YYYY- MM – DD HH：MM：SS’或YYYYMMDDHHMMSS.uuuuuu格式，这取决于是否是在一个字符串或数字语境中使用的功能。该值表示在当前的时区。
bigint | unix_timestamp() | 如果不带参数的调用，返回一个Unix时间戳（从’1970-01–0100:00:00′到现在的UTC秒数）为无符号整数。
bigint | unix_timestamp(string date) | 指定日期参数调用UNIX_TIMESTAMP（），它返回参数值’1970- 01 – 0100:00:00′到指定日期的秒数。
bigint | unix_timestamp(string date, string pattern) | 指定时间输入格式，返回到1970年秒数：unix_timestamp(’2009-03-20′, ‘yyyy-MM-dd’) = 1237532400
string | to_date(string timestamp) | 返回时间中的年月日： to_date(“1970-01-01 00:00:00″) = “1970-01-01″
string | to_dates(string date) | 给定一个日期date，返回一个天数（0年以来的天数）
int | year(string date) | 返回指定时间的年份
int | month(string date) | 返回指定时间的月份
int | day(string date) | dayofmonth(date) 返回指定时间的日期
int | hour(string date) | 返回指定时间的小时，范围为0到23。
int | minute(string date) | 返回指定时间的分钟，范围为0到59。
int | second(string date) | 返回指定时间的秒，范围为0到59。
int | weekofyear(string date) | 返回指定日期所在一年中的星期号，范围为0到53。
int | datediff(string enddate, string startdate) | 两个时间参数的日期之差。
int | date_add(string startdate, int days) | 给定时间，在此基础上加上指定的时间段。
int | date_sub(string startdate, int days) | 给定时间，在此基础上减去指定的时间段。

**NOTE:**所以想把一个**20160302**转换成**2016-03-02**这种时间格式，要么用**strsub()**配合**concat()**来实现，或者就使用`from_unixtime(unix_timestamp(create_time, 'yyyyMMdd'), 'yyyy-MM-dd)`
### 字符函数

返回类型 | 函数 | 说明
---------|------|-------
int | length(string A) | 返回字符串的长度
string | reverse(string A) | 返回倒序字符串
string | concat(string A, string B…) | 连接多个字符串，合并为一个字符串，可以接受任意数量的输入字符串
string | concat_ws(string SEP, string A, string B…) | 链接多个字符串，字符串之间以指定的分隔符分开。
string | substr(string A, int start, int len)<br>substring(string A, int start, int len) | 从文本字符串中指定的位置指定长度的字符。
string | upper(string A)<br>ucase(string A) | 将文本字符串转换成字母全部大写形式
string | lower(string A)<br>lcase(string A) | 将文本字符串转换成字母全部小写形式
string | trim(string A) | 删除字符串两端的空格，字符之间的空格保留
string | ltrim(string A) | 删除字符串左边的空格，其他的空格保留
string | rtrim(string A) | 删除字符串右边的空格，其他的空格保留
string | regexp_replace(string A, string B, string C) | 字符串A中的B字符被C字符替代
string | regexp_extract(string subject, string pattern, int index) | 通过下标返回正则表达式指定的部分。需要注意的是：原来的`\` 转义，这里变成了双斜杠了`\\`,所以对于`[,{`这些特殊字符，需要注意使用两个斜杠，包括原来的正则表达式，如果是`\d`匹配数字，就需要使用`\\d`
string | parse_url(string urlString, string partToExtract [, string keyToExtract]) | 返回URL指定的部分。parse_url(‘http://facebook.com/path1/p.php?k1=v1&k2=v2#Ref1′, ‘HOST’) 返回：’facebook.com’,key处理指定为host,还可以指定为path,[query,key]
string | get_json_object(string json_string, string path) | select a.timestamp, get_json_object(a.appevents, ‘$.eventid’), get_json_object(a.appenvets, ‘$.eventname’) from log a;
string | space(int n) |	返回指定数量的空格
string | repeat(string str, int n) | 重复N次字符串
int | ascii(string str) | 返回字符串中首字符的数字值
string | lpad(string str, int len, string pad) | 返回指定长度的字符串，给定字符串长度小于指定长度时，由指定字符从左侧填补。
string | rpad(string str, int len, string pad) | 返回指定长度的字符串，给定字符串长度小于指定长度时，由指定字符从右侧填补。
array |	split(string str, string pat) | 将字符串转换为数组。
int | find_in_set(string str, string strList) |	返回字符串str第一次在strlist出现的位置。如果任一参数为NULL,返回NULL；如果第一个参数包含逗号，返回0。
array<array<string>> | sentences(string str, string lang, string locale) | 将字符串中内容按语句分组，每个单词间以逗号分隔，最后返回数组。 例如sentences(‘Hello there! How are you?’) 返回：( (“Hello”, “there”), (“How”, “are”, “you”) )

### 内置聚合函数
还有一些是内置的统计函数,可以免去不少计算的步骤

返回类型 | 函数 | 说明
---------|------|-------
bigint | count(*)<br>count(expr)<br>count(DISTINCT expr[, expr_., expr_.]) | 返回记录条数。
double | sum(col)<br>sum(DISTINCT col) | 求和
double | avg(col)<br>avg(DISTINCT col) | 求平均值
double | var_pop(col) | 返回指定列的方差
double | var_samp(col) | 返回指定列的样本方差
double | stddev_pop(col) | 返回指定列的偏差
double | stddev_samp(col) | 返回指定列的样本偏差
double | covar_pop(col1, col2) | 两列数值协方差
double | covar_samp(col1, col2) | 两列数值样本协方差
double | corr(col1, col2) | 返回两列数值的相关系数
double | percentile(col, p) | 返回数值区域的百分比数值点。0<=P<=1,否则返回NULL,不支持浮点型数值。
