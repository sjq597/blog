title: Hexo+next主题开启搜索功能
date: 2023-10-09 18:00:19
tags: [Hexo]
categories: 博客搭建
---

> 最近重新把博客捡起来了，由于好久都没更新，导致在新的机器上各种跑不起来，于是索性把依赖都更新成最新的了。另外好久不用，hexo跟next主题更新了不少新功能，也顺便都加上了，加了几个插件，感觉还不错，尤其是搜索插件，很多年以前用Hexo些博客最头疼的就是没有检索功能，只能依靠标签跟归档文件夹去找。在添加搜索插件的时候，一开始并不生效，后面解决了，在此记录录一下开启搜索功能的过程。

### 准备工作
本次是以[NexT](https://theme-next.js.org/)主题为例，所以确保你也是使用的这个主题，其他的请自行Google搜索，由于配置用的是最新的，所以也贴一下版本号
```bash
➜  blog git:(master) ✗ hexo -v
INFO  Validating config
hexo: 6.3.0
hexo-cli: 4.3.1
os: darwin 22.5.0 13.4
```
NexT版本是
```
"name": "hexo-theme-next",
"version": "8.18.1",
```

### 安装搜索插件
在博客根目录执行命令
```bash
npm install hexo-generator-searchdb
```
然后重新生成
```bash
hexo g
```
都操作完之后，可以在`public`目录下看到多了一个文件`search.xml`，到这里我以为已经可以了，但是启动本地服务之后，反复试了几次还是不行，导航栏没有任何变化，根本没有搜索入口。
然后去看了下官方文档，发现好像要自己写代码，官方文档解释如下：
```
How to use this plugin in my Hexo blog?
You have two choices:

you don't want to write search engine by yourself. There are many themes that take use this plugin for local searching that works out of box.

you are familiar with JavaScript and would like to write your own search engine. You can implement one by yourself according to the template code search.js. There is no documentation at present, but you can find its usage in the source code of the theme NexT. Generally there are 3 steps:

write a search view. This is the place for displaying a search form and search results;
load the search.js script via CDN, for example:
<script src="https://cdn.jsdelivr.net/npm/hexo-generator-searchdb@1.4.0/dist/search.js"></script>
A LocalSearch class is provided in the search.js which tells the browser how to grab search data and filter out contents what we're searching;

write a search script, make use of the previous LocalSearch class.
```
乍一看好像得自己写JS代码才能行，准备放弃了，后来仔细看了下第一段，好多主题都是用的这个插件来实现本地搜索的，于是我搜了下NexT的本地配置文件`_config.yml`，发现了这么一段配置
```yml
# Local Search
# Dependencies: https://github.com/next-theme/hexo-generator-searchdb
local_search:
  enable: false
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```
原来NexT的本地搜索用的就是`hexo-generator-searchdb`插件，只是开关默认是关闭的，于是将`enable`属性改成`true`，再次重新启动服务，本地已经出现搜索入口了，如图所示：
![本地搜索功能截图](https://blog-1254088983.cos.ap-guangzhou.myqcloud.com/Hexo%2Bnext%E4%B8%BB%E9%A2%98%E5%BC%80%E5%90%AF%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD001.png)
