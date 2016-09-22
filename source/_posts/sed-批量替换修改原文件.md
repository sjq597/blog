title: sed 批量替换修改原文件
date: 2016-09-18 15:23:30
tags: [Linux,Shell]
categories: [Linux使用]
---
网上下载某个js库，官网下载的源码一般都带有样例，但是好多html里面引用的js都是使用的cdn网址，类似于这样:
```
<script src="https://cdn.bootcss.com/d3/3.5.2/d3.min.js"></script>
```
但是好多优秀的JS插件都是国外的，所以很多作者的项目里面样例都是使用的国外的cdn,例如
```
https://cdnjs.com/
```
但是由于某些特殊的你懂得原因，上面这个网占经常没办法访问，或者访问不了,所以我只能替换成国内的cdn，比如我上面最开始写的那个就是，现在问题来了，我想把`example`下面的所有`html`文件里面的这个js都替换掉，即把:
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.2/d3.min.js" charset="utf-8"></script>
```
替换成:
```
<script src="https://cdn.bootcss.com/d3/3.5.2/d3.min.js"></script>
```

其实使用一条Linux命令就可以了:
```
sed 's/old_xxx/new_xxx/g' <file_name>
```

```
cd examples
sed -i 's#https://cdn.bootcss.com/d3/3.5.2/d3.min.js#https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.2/d3.min.js#g' *.html
```
上面需要稍作解释:

* -i

如果不加`-i`参数，那么sed会把结果输出到终端，加了之后替换的结果会写回到原文件

* s#old_xxx#new_xxx#g

原替换语法是`s/old_xxx/new_xxx/g`,但是如果原本的内容里面就有`/`，要么转义这个字符，或者为了不破外可读性，降低出错概率，直接用`#`做分隔符更好

* *.html

由于要替换所有的，所以最后面的文件名参数直接就写成`*.html`，这样就会把所有的`html`文件里面的内容替换并写回原文件了。
