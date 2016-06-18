title: Shell 反引号，单引号，双引号
date: 2015-11-06 13:22:53
tags: [Linux,Shell]
categories: Shell 
---
最近碰到个坑,在写shell脚本查询Hive数据的时候,有一个脚本在Shell里怎么都执行不出结果,但是拷贝到Hive界面里却能正常执行出结果,因为Hive表是按日期分区的,有两种格式:
一种是`%Y%m%d`,另一种是`%Y-%m-%d`.dw_source的表是第一种格式,而dw_transform层的表是第二种格式:
### 错误样例
```bash
# sql用单引号 执行成功,可以返回正常结果
sql1='select uid,first_ip,first_date from dw_source.user_ext where dt='20151104';'
sudo -uhive_user hive -e "$sql1" > source.txt

# sql用单引号 执行成功,但是没有返回值
sql2='select uid,first_ip,first_date from dw_transform.user_ext where dt='2015-11-04';'
sudo -uhive_user hive -e "$sql2" > transform.txt
```
我们可以用`echo`命令来看看具体执行的命令:
```bash
sql1='select uid,first_ip,first_date from dw_source.user_ext where dt='20151104';'
sql2='select uid,first_ip,first_date from dw_transform.user_ext where dt='2015-11-04';'

echo sudo -uhive_user hive -e "$sql1"
echo sudo -uhive_user hive -e "$sql2"
```
看看返回结果:
```bash
sudo -uhive_user hive -e select uid,first_ip,first_date from dw_source.user_ext where dt=20151104;
sudo -uhive_user hive -e select uid,first_ip,first_date from dw_transform.user_ext where dt=2015-11-04;
```
看到这里发现了一个问题,本来在日期上应该有的单引号消失了,因为第一种时间格式没有分隔符,可以被正常的识别,但是第二种就比较悲催了,应该是被`-`分隔了,导致日期不对,解决办法就是最外面用双引号,即改成如下内容:
```bash
sql1="select uid,first_ip,first_date from dw_source.user_ext where dt='20151104';"
sql2="select uid,first_ip,first_date from dw_transform.user_ext where dt='2015-11-04';"
```
### 反引号 双引号 单引号用法
这几个符号长得比较像，但是功能还是有很大区别的。

#### 反引号(`)
起着命令替换的作用。命令替换是指shell能够将一个命令的标准输出插在一个命令行中任何位置。,举个简单的例子:
```bash
➜  ~  echo The date is `date`
The date is 2015年 11月 06日 星期五 16:27:46 CST
```

#### 双引号(")
双引号是字符串的界定符,而不是字符的界定符,取消除[\`,$,",]以外,其他的都变成字符串的内容了.双引号是弱引用，引号里的值若再包含变量，那在赋值的时候，所有这些变量就被立即替换了。
用双引号的时候:
```
$加变量名可以取变量的值
反引号仍表示命令替换
\$表示$的字面值
\`表示`的字面值
\"表示"的字面值
\\表示\的字面值
```
看个简单的例子:
```bash
➜  ~  name=World
➜  ~  echo $name
World
➜  ~  sayHello="Hello $name"
➜  ~  echo $sayHello
Hello World
```

#### 单引号(')
单引号告诉shell忽略所有特殊字符,保持引号内所有字符的字面值，即使引号内的\和回车也不例外,单引号是强引用,但是字符串中不能出现单引号。之前的问题就是犯了这个错,两个单引号嵌套了.
```bash
➜  ~  echo "`date`"
2015年 11月 06日 星期五 16:42:29 CST
➜  ~  echo '`date`'
`date`
➜  ~  echo "'`date`'"
'2015年 11月 06日 星期五 16:42:29 CST'
➜  ~  
```
**注意:**如果`'`出现在`"`里面，`'`就退化为一个单纯的字符了，而不再有强引用一说。换句话说，只有最外层的`'`才具有强引用效果。

### 应用
这个对我们有啥影响呢？基本在我们使用Shell的时候都会碰到这个坑，尤其是正则表达式，有时候匹配数字用`\d`就可以了，但是像Hive这种用Java写的，内部会有转义，所以需要传入两个`\\`的时候，就容易踩坑，以Hive中的正则为例,看下面:
```
reg_str1='.*id=(\\d).*'
reg_str2=".*id=(\\\\d).*"
```
这两种写法都是正确的写法，效果是一样的，其他的写法都是不对的
