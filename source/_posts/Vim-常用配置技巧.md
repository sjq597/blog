title: Vim 常用配置技巧
date: 2015-11-05 20:44:29
tags: [Vim,Linux]
categories: Vim
---
Vim虽然很好用,但是平时有时候还不够好,需要自己配置一下,比如说,每次粘贴到vim里一大片内容,可能就会被弄乱,常用的vim配置设置都在`~/.vimrc`里.

### 基本配置
基本配置就是平时的一些简单的配置,会经常用到的,下面都是我从平时工作中需要用到的配置中慢慢总结出来的,下面讲的如果没有特殊说明,都是默认配置在`~/.vimrc`文件中.

#### 设置table的宽度
写shell的时候,经常会在`if`的下一行给我空出两个制表符,每个制表符都是8个字符宽度,看着很别扭,`ts`是一个制表符显示宽度,`sw`是制表符实际占宽度.
```
set ts=4
set sw=4
```

#### 设置粘贴不自动缩进
粘贴之前在vim编辑器里先:
```bash
:set paste
```
然后再`insert`,把内容粘贴到vim里即可,粘贴完还需要关闭:
```bash
:set nopaste
```
相信你也觉得好麻烦,其实这个有个开关模式,只要选择打开开关,关闭开关就能达到我们要的效果,所以利用键盘映射到这个开关,在配置文件里加上:
前两行是为了提示的,这样你在粘贴的时候就可以区分`paste`模式和`nopaste`模式.
```
nnoremap <F2> :set invpaste paste?<CR>
imap <F2> <C-O>:set invpaste paste?<CR>
set pastetoggle=<F2>
```

#### 更改Esc映射
`Vim`里使用`Esc`的频率很高,但是一般的键盘设计使得按`Esc`很不方便,而且在Vim里,`Caps Lock`就是多余的,为什么这么说?因为有快捷键可以直接将小写转大写,单个的大写就使用`Shift`即可.
在终端里面执行下面命令:
```bash
xmodmap -e 'clear Lock' -e 'keycode 0x42 = Escape'
```
这样你的建就被映射了,但是每次你重启之后,还得重新做映射,而且我们只想这个功能在Vim里是这样的,当我们退出Vim,键盘的按键恢复正常,可以在`.vimrc`文件里加上下面的内容:
```
" 当进入vim的时候将Esc映射成大写键,退出的时候将大写键映射回来
au VimEnter * !xmodmap -e 'clear Lock' -e 'keycode 0x42 = Escape'
au VimLeave * !xmodmap -e 'clear Lock' -e 'keycode 0x42 = Caps_Lock'
```
为了使用更加友好,还可以在`.vimrc`文件里加上下面这句:
在insert模式下,按<Ctrl+u>将光标所在单词变大写,记得要重启vim才会生效.
```
inoremap <C-u> <esc>gUiwea
```
这个需要安装`xorg-xmodmap`安装包.
