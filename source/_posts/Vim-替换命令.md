title: Vim 替换命令
date: 2015-11-10 19:36:42
tags: [Linux,Vim]
categories: Vim
---
### 替换[substitute]
```
:[range]s/pattern/string/[c,e,g,i]
```

### 参数
命令 | 参数解释
-----|---------
range | 范围,`1,7`表示第一行至第七行.<br>`1,$`表示第一行至最后一行,即整篇文章,也可以使用`%`来替代,`%`代表目前编辑的文章<br>`#`是前一次编辑的文章.
pattern | 将要被替换掉的字符串,也可以用正则来表示
string | 用来替换到文本中的字符串
c | confirm,每次替换前会询问
e | 不显示error
g | global,不询问,整行替换
i | ignore 不分大小写

**注意:**`g`一般都要加上,否则只会替换每一行的第一个符合`pattern`的字符串.当然后面的四个参数可以一起用,不用逗号隔开,例如`cgi`表示:每次都询问,整行替换,不分大小写.

### 用法举例
替换就这几种开关,用法千变万化,以下命令都是在Vim里操作的.
命令 | 命令作用解释
-----|--------------
:s/test/dev/ | 替换当前行第一个test为dev
:s/test/dev/g | 替换当前行所有test为dev
:n,$s/test/dev/ | 替换从第n行开始到最后一行,每一行第一个test为dev
:n,$s/test/dev/g | 替换从第n行开始到最后一行,每一行所有test为dev<br>`n`为数字,若`n`为`.`或者省略`n`,表示从当光标所在行开始。
:%s/test/dev/ | 等同于`:1,$s/test/dev/`
:%s/test/dev/g | 等同于`:1,$s/test/dev/g`
:s#test/#dev/# | 替换当前行第一个test为dev<br>可以使用#作为分隔符，此时中间出现的`/`不会作为分隔符
:%s+/test/beta/+/dev/product/+ | 使用`+`来替换`/`，将`/test/beta/`替换成`/dev/product/`
:%s/^M$//g | windows先回车是`0A0D`,Unix下是`0A`，所以会多一个`^M`<br>替换条件是一个正则表达式,意思是以`^M`结尾
