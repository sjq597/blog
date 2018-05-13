title: awk 使用总结
date: 2018-05-12 16:26:55
tags: [Linux,Shell]
categories: [Linux使用]
---
* 单引号双引号输出

1.双引号:
```
awk '{print "\""}'
```
使用`“”`双引号把一个双引号括起来,然后用转义字符`\`对双引号进行转义,输出双引号.


2.单引号:
```
awk '{print "'\''"}'
```
使用一个双引号`“”`然后在双引号里面加入两个单引号`‘’`,接着在两个单引号里面加入一个转义的单引号`\'`，输出单引号.

* 字符串操作

1.取子串
```
substr(String, M, [N])
```

2.分割域访问
`NF`代表域个数,`$NF`最后一个,`$(NF-1)`倒数第二个
字符串,数字比较直接`<=`;`>=`;`==`这样
```
-F'xxx'用xxx分割
```

3.分割结果保存到数组
`($())`,例如想将`ls -l`的文件列表分割出来:
```
file_list=($(hadoop fs -ls /user/hive/warehouse/xxx.db | grep user | awk -F' ' '{print $NF}'))
```
这个就是获取hdfs上指定库目录的所有表名,然后就可以for遍历
```
for table in ${file_list[@]};do
  echo $table
done
```
