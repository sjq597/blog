title: Ubuntu 14.04 Sublime Text 3 不能输入中文
date: 2015-10-02 00:09:30
tags: [Linux,开发工具,Web,Sublime Text,开发环境]
categories: Linux使用
---
### Ubuntu 14.04 Sublime Text 3 不能输入中文
`sublime`这个编辑器很好用，小巧，比`IDE`要轻，主要是想用来写`markdown`。但是`ubuntu`下不能输入中文，这是一个很大的问题。

#### 测试环境
> Ubuntu 14.04 64bits
sougou pingyin v1.2.0.0056
Sublime Text 3

#### 解决方法
* 1.打开终端，下载所需要的库

```
➜  ~  git clone git@github.com:lyfeyaj/sublime-text-imfix.git
```
**NOTE:** 如果你没有装`git`，直接去[项目github](https://github.com/lyfeyaj/sublime-text-imfix)地址下载也行。下载下来的项目的目录结构如下：
```
➜  sublime-text-imfix git:(master) tree -L 3
.
├── image
│   └── fcitx.png
├── lib
│   └── libsublime-imfix.so
├── README.md
├── src
│   ├── anran.tar.gz
│   ├── subl
│   └── sublime-imfix.c
└── sublime-imfix

3 directories, 7 files
```
* 2.将subl移动到/usr/bin/，并且将sublime-imfix.so移动到/opt/sublime_text/（sublime的安装目录）

```
➜  ~  cd sublime-text-imfix
➜  ~  sudo cp ./lib/libsublime-imfix.so /opt/sublime_text 
➜  ~  sudo cp ./src/subl /usr/bin/
```

* 3.用subl命令试试能不能启动sublime，如果成功启动的话，应该就可以输入中文了。

```
➜  ~  subl
```
