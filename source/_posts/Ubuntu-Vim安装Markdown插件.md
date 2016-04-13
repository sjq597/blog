title: Ubuntu Vim安装Markdown插件
date: 2015-11-02 19:30:04
tags: [Vim,Linux,开发工具]
categories: Vim
---
在Ubuntu下使用vim开发,写博客,由于博客是使用markdown写的,之前是使用sublime Text来写markdown,不过还是想完全用键盘,书写更快,决定以后都使用vim来写markdown了.

### 安装前提 
首先要安装vim,没有的话终端安装也方便:
```bash
sudo apt-get install vim
```
然后安装插件管理器,有几种方式,就以`pathogen`为例
```bash
sudo mkdir -p ~/.vim/autoload ~/.vim/bundle && curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
```
**注意：** 如果执行这个命令报错：
```bash
curl: (23) Failed writing body (0 != 12140)
```
这是没有写权限所导致的，可以为三个文件夹赋权限
```bash
sudo chmod 777 ~/.vim
cd ~/.vim
sudo chmod 777 autoload bundle 
```
然后在`~/.vimrc`文件里添加如下内容
```
execute pathogen#infect()
```
**注意:** 如果刚装的vim,啥东西也没有加过,可能没有`.vimrc`文件,需要自己新建
```bash
sudo vim ~/.vimrc
```
然后添加如下内容:
```
execute pathogen#infect()
syntax on
filetype plugin indent on
```

### 安装vim-markdown插件
现在假设你的`pathogen`已经安装好了,安装`vim-markdown`插件
```bash
cd ~/.vim/bundle
git clone https://github.com/plasticboy/vim-markdown.git
```
然后重新打开一个`*.md`文件就可以生效了,注意,打开`*.md`文件,标题块之间会被折叠,光标跳转到折叠行,按空格就可以展开.

由于是用`hexo`写博客,还需要对`YAML`语法,需要在`~/.vimrc`中添加如下配置:
```
let g:vim_markdown_frontmatter=1
```
基本的插件安装就这几个步骤，其实也没有很特别的提示功能，只是在写错的时候可以看出来，还有就是按标题折叠成块比较实用。
