title: Hive get_json_object用法
date: 2015-11-05 18:14:09
tags: [大数据,Hive,SQL]
categories: Hive笔记
---
数据库中存放的json串,有时候需要对某个元素判断来查询结果,非常笨的一个方法就是直接把查询结果的字符串做模糊查询,即`like '%str%'`,当一个json串非常长的时候,本来效率就很慢,况且是在hadoop海量数据里查找,其实我们并不需要那些其他的字符串,只是需要某个元素的值而已,这个时候就需要用到Hive的字符函数`get_json_object()`函数.

### get_json_object()函数
函数用法:
```
get_json_object(string json_string, string path)
```
具体看一个例子,数据库test定义如下:
```
id                  	int                 	自增id
content                	string              	内容	
```
其中content是个json串,内容如下:
```json
{
  "status": {
     "person": {
        "name": false
    }
  }
}
```
查询一下看看结果:
```bash
select get_json_object(content,'$.status') from test limit 1;
OK
{"person":{"name":false}}
Time taken: 0.066 seconds, Fetched: 1 row(s)

select get_json_object(content,'$.status.person') from test limit 1;
OK
{"name":false}
Time taken: 0.081 seconds, Fetched: 1 row(s)

select get_json_object(content,'$.status.person.name') from test limit 1;
OK
false
Time taken: 0.077 seconds, Fetched: 1 row(s)
```
**注意:**如果要判断`name`的值,这个值并不是布尔型,而是一个string,所以需要加上`''`,像下面这样:
```bash
select id,content from test where get_json_object(content,'$.status.person.name')='false' limit 2;
OK
7	{.status":{"person":{"name":false}}}
31	{.status":{"person":{"name":false}}}
Time taken: 0.085 seco	nds, Fetched: 2 row(s)
```
