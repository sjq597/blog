title: Hello World
tags: Hexo
categories: [博客搭建]
---
Welcome to [Hexo](http://hexo.io/)! This is your very first post. Check [documentation](http://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](http://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](http://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](http://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](http://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](http://hexo.io/docs/deployment.html)

### 给文章添加分类
在根目录下的`scaffolds`目录下，修改`post.md`文件，内容改为如下：
```
title: {{ title }}
date: {{ date }}
tags:
categories:
---
```

以后每次生成新文章都会带有`categories`，貌似只能填一个，如果要多个，自己填标签吧。