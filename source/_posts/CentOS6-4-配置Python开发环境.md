title: CentOS6.4 配置Python开发环境
date: 2016-01-20 14:08:46
tags: [Python,Linux,Vim]
categories: 开发环境
---
最近写Python脚本,需要在服务器上运行,也没办法用PyCharm这种带界面的IDE来写Python,不过看了一篇博客,讲的十分不错,所以自己照着弄了一遍,中间有些不够详细的我也一并记录下来.

### 准备环境
OS:Linux/Unix(服务器是CentOS6.4 64bit)
Vim: >=vim 7.3(终端直接输入vim --version 可以查看版本)
Python: 具体的没试过,我的是Python2.7.4


### Vim扩展
Vim之所以好用,一方面是自身的快捷键很强大,记熟了确实很好用,但是纵观任何一个流行的编辑器,更为重要的一点是可扩展性极强,每个人都可以定制和扩展,最终打造一个最适合自己的编辑器,这个可以说是每个流行的编辑器,浏览器也是如此得以流行的不可或缺的原因.插件太多,一个一个来装太麻烦,并且多了管理起来也不方便,换个系统又得一个一个安装,太麻烦,所以我们需要的第一个东西是:好用的扩展管理器.
Vim的扩展通常我们叫bundle或者插件

### Vundle
Vim的编辑器很多,我就随便选一个了,普通的用也没区别,强烈推荐`Vundle`.
先来安装Vundle:
```bash
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/vundle
```
上面的这个只是把Vundle插件管理器下载下来了,并且把插件放在了`~/.vim/bundle/`目录中,现在通过编辑`~/.vimrc`来配置Vim编辑器来安装Vundle,如果没有则自己创建一个:
```bash
touch ~/.vimrc
```
接下在把下面的代码放到`~/.vimrc`文件顶部:
```bash
set nocompatible              " required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/vundle
call vundle#begin()

" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/vundle'

" Add all your plugins here (note older versions of Vundle used Bundle instead of Plugin)


" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
```
这样就完成了Vundle的设置,然后就可以安装了,打开Vim编辑器,输入下面的命令:
```
:PluginInstall
```
这个命令非常好用,它告诉Vundle施展它的魔法——自动下载所有的插件，并为你进行安装和更新。

### 开始打造IDE
如果你对Vim不熟,可以看看我的这篇文章:[Vim-常用命令总结](),常用的一些命令基本就在这了.好了,开始正式配置Vim了.

#### 代码折叠
```bash
" Enable folding
set foldmethod=indent
set foldlevel=99
```


