title: Vim 编程常用高效必备插件
date: 2015-11-02 20:06:28
tags: [Linux,Vim]
categories: Vim
---
假设你已经了解基本的vim使用,并且喜欢在vim下写代码,你需要几个非常好用的插件,可以极大的提升你的工作效率.

### 插件管理利器
原始的插件管理方法就是下一个插件,拷贝到一个目录下就行了,但是这样有点儿乱,插件管理工具常用的用`pathogen`和`Vundle`.就管理插件而言,不需要太过于关注哪个好,都挺好用的,我使用的就是`pathogen`,其安装方法详见[Ubuntu vim安装markdown插件](../Ubuntu-vim安装markdown插件)这篇笔记里的介绍.

### 常用插件
列举了几个比较常用的插件
1. nerdtree
2. snipMate
3. vim-multiple-cursors

以下所有安装方法都是使用`pathogen`插件管理器的安装方法,其他方法自行琢磨.

#### nerdtree插件
```bash
cd ~/.vim/bundle
git clone https://github.com/scrooloose/nerdtree.git
```
这个插件可以使用vim来打开文件夹,按回车进入文件夹,展开文件夹下面的所有文件,可以用vim的光标控制命令来移动,选择打开某个文件.

#### snipMate插件
```bash
cd ~/.vim/bundle
git clone https://github.com/tomtom/tlib_vim.git
git clone https://github.com/MarcWeber/vim-addon-mw-utils.git
git clone https://github.com/garbas/vim-snipmate.git
git clone https://github.com/honza/vim-snippets.git
```
这个工具可以快速,减少重复代码,关于具体的使用,你可以在插件目录`bundle/`存放插件的`snippets`目录下找到对应的语言的`snippets`,并且修改对应的`snippets`.

#### vim-multiple-cursors插件
```bash
cd ~/.vim/bundle
git clone https://github.com/terryma/vim-multiple-cursors.git
```
看名字的意思就是多光标编辑,比如你有个变量,在很多地方被引用了,突然有一天你觉得这个变量名字起的不好,想换一个变量名,这个时候可以使用这个,具体效果如下:
测试代码`multiple.py`:
```python
def hell(poorly_named_var)
    poorly_named_var ||= "Nameless"
        puts("Hi, " + poorly_named_var)
	end 
```
![vim-multiple-cursors插件多光标效果](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Vim%20编程常用高效必备插件01.gif)
上图的演示过程就是，具体的操作就是将光标移到变量的第一个字母，然后按`<Crrl-n>`就是增加一个，按`<Ctrl-p>`就是减少一个。连按三下都选中，然后按`c`,然后输入name即是上面的效果。
