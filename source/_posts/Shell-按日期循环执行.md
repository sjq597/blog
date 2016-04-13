title: Shell 按日期循环执行
date: 2015-11-03 20:41:17
tags: [Linux,Shell]
categories: Shell
---
服务器上有些脚本,执行的参数用到了日期,也就是每天执行一次,日期取当天时间作为参数,当有些时候需要把这些脚本过去一段时间的运行结果重新运行一次时,手动指定脚本日期没有办法大批量执行,需要在shell中写循环,让脚本批量执行.

以一个小的例子来说:把指定日期时间段内的日期都输出来,看看下面的代码:
```bash
#! /bin/bash

start_date=20151101
end_date=20151103
start_sec=`date -d "$start_date" "+%s"`
end_sec=`date -d "$end_date" "+%s"`
for((i=start_sec;i<=end_sec;i+=86400)); do
    day=$(date -d "@$i" "+%Y-%m-%d")
    echo $day
done
```
看看输出结果:
```
2015-11-01
2015-11-02
2015-11-03
```
执行结果是循环输出日期,把需要循环的脚本放在循环里调用,把`$day`当参数使用即可.如果脚本里面的变量是日期,那就把脚本代码拷贝到循环中间.日期的格式可以按需求自己调整.例如:
```bash
for((i=start_sec;i<=end_sec;i+=86400)); do
    day=$(date -d "@$i" "+%Y-%m-%d")
    echo $day
    sudo /usr/python/ /usr/dev/job.py $day
done
```
这样就可以把时间参数批量传入脚本.
