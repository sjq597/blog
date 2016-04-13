title: Hexo 基本配置
date: 2015-10-25 10:41:50
tags: [Hexo]
categories: [博客搭建]
---
前面的文章介绍了如何安装hexo和使用hexo，但是关于hexo的常用配置，还是需要自己折腾一下。

### 给文章添加分类
在根目录下的`scaffolds`目录下，修改`post.md`文件，内容改为如下：
```
title: {{ title }}
date: {{ date }}
tags:
categories:
---
```

**注意：** 目前貌似标签可以有很多个，语法类似于`tags: [tag1,tag2,tag3]`这样，但是分类好像只能填一个，类似于`categories: 测试`。

### 更换Hexo博客模板
这里以Jackman配置为例，首先在github官网下载主题
```bash
git clone https://github.com/wuchong/jacman.git themes/jacman
```
**NOTE:** [百度云备份地址](http://pan.baidu.com/s/1o67ZUvK),提取密码：s7r6。

将`themes`文件夹下的`jacman`文件夹复制到你的Hexo项目的`themes`文件夹下面
```bash
cd themes
sudo mv jacman ~/Documents/blog/themes
```

然后修改`blog`文件夹下的`_config.yml`文件:
```
# Extensions
## Plugins: http://hexo.io/plugins/
## Themes: http://hexo.io/themes/
theme: landscape
```
把`theme:landscape`修改为`theme:jacman`。


### Hexo使用公式
我是使用的`jacman`的主题,这个主题使用公式很简单,只需要修改主题文件夹下的`_config.yml`文件即可.具体为修改`themes/jacman/_config.yml`文件.
将文件中的:
```yml
close_aside: false  #close sidebar in post page if true
mathjax: false      #enable mathjax if true
```
里的`mathjax`对应的值改为`true`就可以了,测试一下:
$$E=MC^2$$
公式写法参见[MathJax使用LaTeX语法编写数学公式教程](http://iori.sinaapp.com/17.html/comment-page-1?replytocom=2)
