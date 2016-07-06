title: 终极 Shell——zsh
date: 2015-10-05 23:53:18
tags: [Linux,开发工具]
categories: 开发环境
---
目前常用的`Linux`系统和`OS X`系统的默认`Shell`都是`bash`，但是真正强大的`Shell` 是深藏不露的`zsh`， 这货绝对是马车中的跑车，跑车中的飞行车，史称『终极 Shell』，但是由于配置过于复杂，所以初期无人问津，很多人跑过来看看 zsh 的配置指南，什么都不说转身就走了。直到有一天，国外有个穷极无聊的程序员开发出了一个能够让你快速上手的zsh项目，叫做「oh my zsh」，[Github网址](https://github.com/robbyrussell/oh-my-zsh) 。

好，下面我们以Ubuntu 14.04为例看看如何安装、配置和使用 zsh。

### 安装oh my zsh
首先确保你安装了`git, zsh`,如果没有安装，按下面的命令来安装：
```
sudo apt-get install -y git
sudo apt-get install -y zsh
```
装好了`zsh`还不够,还需要设置一下系统默认的shell环境:
```
chsh -s $(which zsh)
```
然后重启，或者注销系统重新登陆。
然后就可以安装`oh-my-zsh`了:
```bash
git clone git@github.com:robbyrussell/oh-my-zsh.git
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```
**NOTE:** 有个地方要特别注意，之前的`Java`开发等环境变量都配置在了`.bashrc`文件里，如下图所示：
```bash
export JAVA_HOME=/usr/dev/jdk1.7.0_40
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
export M2_HOME=/usr/dev/apache-maven
export PATH=$PATH:${M2_HOME}/bin
export M2_HOME=/usr/dev/apache-maven
export PATH=$PATH:${M2_HOME}/bin
```
当我安装好`oh my zsh`之后，这些环境变量都失效了，研究一番才明白，我已经把默认的shell换成`.zshrc`了。所以需要把所有的环境变量重新配置到`zshrc`里，把你需要的这些内容拷贝到`.zshrc`里即可。

### 配置
`zsh`的配置主要集中在用户当前目录的`.zshrc`里，用`vim`或你喜欢的其他编辑器打开`.zshrc`

#### 配置快捷命令
可以在此处定义自己的环境变量和别名，当然，`oh my zsh`在安装时已经自动读取当前的环境变量并进行了设置，你可以继续追加其他环境变量。

接下来进行别名的设置，我自己的部分配置如下：
```bash
alias cls='clear'
alias ll='ls -l'
alias la='ls -a'
alias vi='vim'
alias javac="javac -J-Dfile.encoding=utf8"
alias grep="grep --color=auto"
alias -s html=mate   # 在命令行直接输入后缀为 html 的文件名，会在 TextMate 中打开
alias -s rb=mate     # 在命令行直接输入 ruby 文件，会在 TextMate 中打开
alias -s py=vi       # 在命令行直接输入 python 文件，会用 vim 中打开，以下类似
alias -s js=vi
alias -s c=vi
alias -s java=vi
alias -s txt=vi
alias -s gz='tar -xzvf'
alias -s tgz='tar -xzvf'
alias -s zip='unzip'
alias -s bz2='tar -xjvf'
```
**NOTE:** `zsh`的牛逼之处在于不仅可以设置通用别名，还能针对文件类型设置对应的打开程序，比如：
```
alias -s html=mate，意思就是你在命令行输入 hello.html，zsh会为你自动打开 TextMat 并读取  
hello.html 
alias -s gz='tar -xzvf'，表示自动解压后缀为gz的压缩包。
```
总之，只有想不到，木有做不到。

#### 配置主题
设置完环境变量和别名之后，基本上就可以用了，如果你是个主题控，还可以玩玩`zsh`的主题。在`.zshrc`里找到`ZSH_THEME`，就可以设置主题了，默认主题是：
```
ZSH_THEME=”robbyrussell”
```

`oh my zsh`提供了数十种主题，相关文件在`~/.oh-my-zsh/themes`目录下，你可以随意选择。

#### 插件
`oh my zsh`项目提供了完善的插件体系，相关的文件在`~/.oh-my-zsh/plugins`目录下，默认提供了100多种，大家可以根据自己的实际学习和工作环境采用，想了解每个插件的功能，只要打开相关目录下的 zsh 文件看一下就知道了。插件也是在.zshrc里配置，找到plugins关键字，你就可以加载自己的插件了，系统默认加载 git ，你可以在后面追加内容，如下：
```
plugins=(git textmate ruby autojump osx mvn gradle)
```

下面简单介绍几个：

* git

当你处于一个`git`受控的目录下时，`Shell 会明确显示 「git」和 branch，如下图所示：
![zsh-git 插件效果](http://7xn9y9.com1.z0.glb.clouddn.com/note_终极%20Shell——zsh01.png)

另外对`git`很多命令进行了简化，例如`gco=’git checkout’、gd=’git diff’、gst=’git status’、g=’git’`等等，熟练使用可以大大减少`git`的命令长度，命令内容可以参考`~/.oh-my-zshplugins/git/git.plugin.zsh`

* textmate

mr可以创建 ruby 的框架项目，tm finename 可以用 textmate 打开指定文件。

* osx

tab 增强，quick-look filename 可以直接预览文件，man-preview grep 可以生成 grep手册 的pdf 版本等。

* autojump

zsh 和 autojump 的组合形成了 zsh 下最强悍的插件，今天我们主要说说这货。

首先安装autojump，如果你用 Mac，可以使用 brew 安装：
```bash
brew install autojump
```

如果是 Linux，去下载 autojump 的最新版本，比如：
```bash
wget https://github.com/downloads/joelthelion/autojump/autojump_v21.1.2.tar.gz
```

解压缩后进入目录，执行
```
./install.sh
```

最后把以下代码加入.zshrc：
```bash
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

至此，安装、配置、插件三位一体，终极`Shell`全面登场。退出终端会话重新登录，开始感受 zsh 的训疾如风！

### 使用 zsh
1、兼容 bash，原来使用 bash 的兄弟切换过来毫无压力，该咋用咋用。

2、强大的历史纪录功能，输入 grep 然后用上下箭头可以翻阅你执行的所有 grep 命令。

3、智能拼写纠正，输入gtep mactalk * -R，系统会提示：zsh: correct ‘gtep’ to ‘grep’ [nyae]? 比妹纸贴心吧，她们向来都是让你猜的……

4、各种补全：路径补全、命令补全，命令参数补全，插件内容补全等等。触发补全只需要按一下或两下 tab 键，补全项可以使用 ctrl+n/p/f/b上下左右切换。比如你想杀掉 java 的进程，只需要输入 kill java + tab键，如果只有一个 java 进程，zsh 会自动替换为进程的 pid，如果有多个则会出现选择项供你选择。ssh + 空格 + 两个tab键，zsh会列出所有访问过的主机和用户名进行补全

5、智能跳转，安装了autojump之后，zsh 会自动记录你访问过的目录，通过 j + 目录名 可以直接进行目录跳转，而且目录名支持模糊匹配和自动补全，例如你访问过hadoop-1.0.0目录，输入j hado 即可正确跳转。j –stat 可以看你的历史路径库。

6、目录浏览和跳转：输入 d，即可列出你在这个会话里访问的目录列表，输入列表前的序号，即可直接跳转。

7、在当前目录下输入 .. 或 … ，或直接输入当前目录名都可以跳转，你甚至不再需要输入 cd 命令了。

8、通配符搜索：ls -l **/*.sh，可以递归显示当前目录下的 shell 文件，文件少时可以代替 find，文件太多就歇菜了。
