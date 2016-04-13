title: Ubuntu 14.04 多终端窗口合并
date: 2015-10-14 15:16:42
tags: [Linux, 开发工具]
categories: Linux使用
---
最近刚接触数据处理，线上的服务器有大概10台，并且`hadoop`和`Hive`的数据非常多，有些数据需要迁移，首先要登陆跳板机，然后跳板机再登陆线上的机器，这样切换非常麻烦，多个终端窗口切换非常乱，所以找了一下如何使用一款多窗口终端软件

### Terminator(终结者)
#### 终端安装
```
sudo apt-get install terminator
cd ~/.config/
sudo mkdir terminator
cd terminator
sudo touch config
```
安装完了之后，打开一个窗口，类似于下面：
![Terminator 窗口](http://7xn9y9.com1.z0.glb.clouddn.com/note_Ubuntu%2014.04%20多终端窗口合并01.png)

#### 修改默认配置
下面配置一下这个终端的颜色，我使用`solarized`配色,修改配置文件：
```bash
sudo subl ~/.config/terminator/config
```
`subl`是`Sublime Text 3`编辑器，用你有的工具修改即可，把文件内容替换成如下：
```
[global_config]
  title_transmit_bg_color = "#d30102"
  focus = system
[keybindings]
[profiles]
  [[default]]
    # solarized-dark
    #palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
    #foreground_color = "#eee8d5"
    #background_color = "#002b36"
    #cursor_color = "#eee8d5"

  [[solarized-dark]]
    palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
    foreground_color = "#eee8d5"
    background_color = "#002b36"
    cursor_color = "#eee8d5"

  [[solarized-light]]
    palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
    background_color = "#eee8d5"
    foreground_color = "#002b36"
    cursor_color = "#002b36"

[layouts]
  [[default]]
    [[[child1]]]
      type = Terminal
      parent = window0
      profile = solarized-dark
    [[[window0]]]
      type = Window
      parent = ""
[plugins]
```
或者还可以直接导入配置：
```bash
sudo apt-get install curl
curl https://raw.githubusercontent.com/ghuntley/terminator-solarized/master/config > ~/.config/terminator/config
```
**NOTE:** 这个配置项目的[github 地址](https://github.com/ghuntley/terminator-solarized)有可能会变动，如果不行可以复制我的配置文件内容，按前一种方式修改即可,，我的配置文件如下：
```
[global_config]
    title_transmit_bg_color = "#d30102"
    focus = system
    suppress_multiple_term_dialog = True
[keybindings]
[profiles]
    [[default]]
        palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
        copy_on_selection = True
        background_image = None
        background_darkness = 0.95
        background_type = transparent
        use_system_font = False
        cursor_color = "#eee8d5"
        foreground_color = "#839496"
        show_titlebar = False
        font = Monospace 11
        background_color = "#002b36"
    [[solarized-dark]]
        palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
        background_color = "#002b36"
        background_image = None
        cursor_color = "#eee8d5"
        foreground_color = "#839496"
    [[solarized-light]]
        palette = "#073642:#dc322f:#859900:#b58900:#268bd2:#d33682:#2aa198:#eee8d5:#002b36:#cb4b16:#586e75:#657b83:#839496:#6c71c4:#93a1a1:#fdf6e3"
        background_color = "#fdf6e3"
        background_image = None
        cursor_color = "#002b36"
        foreground_color = "#657b83"
[layouts]
    [[default]]
        [[[child1]]]
            type = Terminal
            parent = window0
            profile = default
        [[[window0]]]
            type = Window
            parent = ""
[plugins]
```

#### 更换目录配色方案
不过这个主题在文件和文件夹的颜色上区分不明显，都是灰色的，配置目录的颜色,修改目录配色：
```bash
curl https://raw.githubusercontent.com/seebi/dircolors-solarized/master/dircolors.ansi-dark > ~/.dircolors
```
吐过下载不下来,同样可以自己建一个文件叫`.dircolors`,内容如下:
```
# Below, there should be one TERM entry for each termtype that is colorizable
TERM ansi
TERM color_xterm
TERM color-xterm
TERM con132x25
TERM con132x30
TERM con132x43
TERM con132x60
TERM con80x25
TERM con80x28
TERM con80x30
TERM con80x43
TERM con80x50
TERM con80x60
TERM cons25
TERM console
TERM cygwin
TERM dtterm
TERM dvtm
TERM dvtm-256color
TERM Eterm
TERM eterm-color
TERM fbterm
TERM gnome
TERM gnome-256color
TERM jfbterm
TERM konsole
TERM konsole-256color
TERM kterm
TERM linux
TERM linux-c
TERM mach-color
TERM mlterm
TERM nxterm
TERM putty
TERM putty-256color
TERM rxvt
TERM rxvt-256color
TERM rxvt-cygwin
TERM rxvt-cygwin-native
TERM rxvt-unicode
TERM rxvt-unicode256
TERM rxvt-unicode-256color
TERM screen
TERM screen-16color
TERM screen-16color-bce
TERM screen-16color-s
TERM screen-16color-bce-s
TERM screen-256color
TERM screen-256color-bce
TERM screen-256color-s
TERM screen-256color-bce-s
TERM screen-256color-italic
TERM screen-bce
TERM screen-w
TERM screen.linux
TERM screen.xterm-256color
TERM screen.xterm-new
TERM st
TERM st-meta
TERM st-256color
TERM st-meta-256color
TERM vt100
TERM xterm
TERM xterm-new
TERM xterm-16color
TERM xterm-256color
TERM xterm-256color-italic
TERM xterm-88color
TERM xterm-color
TERM xterm-debian
TERM xterm-termite

# EIGHTBIT, followed by '1' for on, '0' for off. (8-bit output)
EIGHTBIT 1

#############################################################################
# Below are the color init strings for the basic file types. A color init
# string consists of one or more of the following numeric codes:
#
# Attribute codes:
#   00=none 01=bold 04=underscore 05=blink 07=reverse 08=concealed
# Text color codes:
#   30=black 31=red 32=green 33=yellow 34=blue 35=magenta 36=cyan 37=white
# Background color codes:
#   40=black 41=red 42=green 43=yellow 44=blue 45=magenta 46=cyan 47=white
#
# NOTES:
# - See http://www.oreilly.com/catalog/wdnut/excerpt/color_names.html
# - Color combinations
#   ANSI Color code       Solarized  Notes                Universal             SolDark              SolLight
#   ~~~~~~~~~~~~~~~       ~~~~~~~~~  ~~~~~                ~~~~~~~~~             ~~~~~~~              ~~~~~~~~
#   00    none                                            NORMAL, FILE          <SAME>               <SAME>
#   30    black           base02
#   01;30 bright black    base03     bg of SolDark
#   31    red             red                             docs & mm src         <SAME>               <SAME>
#   01;31 bright red      orange                          EXEC                  <SAME>               <SAME>
#   32    green           green                           editable text         <SAME>               <SAME>
#   01;32 bright green    base01                          unimportant text      <SAME>
#   33    yellow          yellow     unclear in light bg  multimedia            <SAME>               <SAME>
#   01;33 bright yellow   base00     fg of SolLight                             unimportant non-text
#   34    blue            blue       unclear in dark bg   user customized       <SAME>               <SAME>
#   01;34 bright blue     base0      fg in SolDark                                                   unimportant text
#   35    magenta         magenta                         LINK                  <SAME>               <SAME>
#   01;35 bright magenta  violet                          archive/compressed    <SAME>               <SAME>
#   36    cyan            cyan                            DIR                   <SAME>               <SAME>
#   01;36 bright cyan     base1                           unimportant non-text                       <SAME>
#   37    white           base2
#   01;37 bright white    base3      bg in SolLight
#   05;37;41                         unclear in Putty dark


### By file type

# global default
NORMAL 00
# normal file
FILE 00
# directory
DIR 34
# 777 directory
OTHER_WRITABLE 34;40
# symbolic link
LINK 35

# pipe, socket, block device, character device (blue bg)
FIFO 30;44
SOCK 35;44
DOOR 35;44 # Solaris 2.5 and later
BLK  33;44
CHR  37;44


#############################################################################
### By file attributes

# Orphaned symlinks (blinking white on red)
# Blink may or may not work (works on iTerm dark or light, and Putty dark)
ORPHAN  05;37;41
# ... and the files that orphaned symlinks point to (blinking white on red)
MISSING 05;37;41

# files with execute permission
EXEC 01;31  # Unix
.cmd 01;31  # Win
.exe 01;31  # Win
.com 01;31  # Win
.bat 01;31  # Win
.reg 01;31  # Win
.app 01;31  # OSX

#############################################################################
### By extension

# List any file extensions like '.gz' or '.tar' that you would like ls
# to colorize below. Put the extension, a space, and the color init string.
# (and any comments you want to add after a '#')

### Text formats

# Text that we can edit with a regular editor
.txt 32
.org 32
.md 32
.mkd 32

# Source text
.h 32
.c 32
.C 32
.cc 32
.cpp 32
.cxx 32
.objc 32
.sh 32
.bash 32
.csh 32
.zsh 32
.el 32
.vim 32
.java 32
.pl 32
.pm 32
.py 32
.rb 32
.hs 32
.php 32
.htm 32
.html 32
.shtml 32
.erb 32
.haml 32
.xml 32
.rdf 32
.css 32
.sass 32
.scss 32
.less 32
.js 32
.coffee 32
.man 32
.0 32
.1 32
.2 32
.3 32
.4 32
.5 32
.6 32
.7 32
.8 32
.9 32
.l 32
.n 32
.p 32
.pod 32
.tex 32
.go 32
.sql 32

### Multimedia formats

# Image
.bmp 33
.cgm 33
.dl 33
.dvi 33
.emf 33
.eps 33
.gif 33
.jpeg 33
.jpg 33
.JPG 33
.mng 33
.pbm 33
.pcx 33
.pdf 33
.pgm 33
.png 33
.PNG 33
.ppm 33
.pps 33
.ppsx 33
.ps 33
.svg 33
.svgz 33
.tga 33
.tif 33
.tiff 33
.xbm 33
.xcf 33
.xpm 33
.xwd 33
.xwd 33
.yuv 33

# Audio
.aac 33
.au  33
.flac 33
.m4a 33
.mid 33
.midi 33
.mka 33
.mp3 33
.mpa 33
.mpeg 33
.mpg 33
.ogg  33
.ra 33
.wav 33

# Video
.anx 33
.asf 33
.avi 33
.axv 33
.flc 33
.fli 33
.flv 33
.gl 33
.m2v 33
.m4v 33
.mkv 33
.mov 33
.MOV 33
.mp4 33
.mp4v 33
.mpeg 33
.mpg 33
.nuv 33
.ogm 33
.ogv 33
.ogx 33
.qt 33
.rm 33
.rmvb 33
.swf 33
.vob 33
.webm 33
.wmv 33

### Misc

# Binary document formats and multimedia source
.doc 31
.docx 31
.rtf 31
.odt 31
.dot 31
.dotx 31
.ott 31
.xls 31
.xlsx 31
.ods 31
.ots 31
.ppt 31
.pptx 31
.odp 31
.otp 31
.fla 31
.psd 31

# Archives, compressed
.7z   1;35
.apk  1;35
.arj  1;35
.bin  1;35
.bz   1;35
.bz2  1;35
.cab  1;35  # Win
.deb  1;35
.dmg  1;35  # OSX
.gem  1;35
.gz   1;35
.iso  1;35
.jar  1;35
.msi  1;35  # Win
.rar  1;35
.rpm  1;35
.tar  1;35
.tbz  1;35
.tbz2 1;35
.tgz  1;35
.tx   1;35
.war  1;35
.xpi  1;35
.xz   1;35
.z    1;35
.Z    1;35
.zip  1;35

# For testing
.ANSI-30-black 30
.ANSI-01;30-brblack 01;30
.ANSI-31-red 31
.ANSI-01;31-brred 01;31
.ANSI-32-green 32
.ANSI-01;32-brgreen 01;32
.ANSI-33-yellow 33
.ANSI-01;33-bryellow 01;33
.ANSI-34-blue 34
.ANSI-01;34-brblue 01;34
.ANSI-35-magenta 35
.ANSI-01;35-brmagenta 01;35
.ANSI-36-cyan 36
.ANSI-01;36-brcyan 01;36
.ANSI-37-white 37
.ANSI-01;37-brwhite 01;37

#############################################################################
# Your customizations

# Unimportant text files
# For universal scheme, use brightgreen 01;32
# For optimal on light bg (but too prominent on dark bg), use white 01;34
.log 01;32
*~ 01;32
*# 01;32
#.log 01;34
#*~ 01;34
#*# 01;34

# Unimportant non-text files
# For universal scheme, use brightcyan 01;36
# For optimal on dark bg (but too prominent on light bg), change to 01;33
#.bak 01;36
#.BAK 01;36
#.old 01;36
#.OLD 01;36
#.org_archive 01;36
#.off 01;36
#.OFF 01;36
#.dist 01;36
#.DIST 01;36
#.orig 01;36
#.ORIG 01;36
#.swp 01;36
#.swo 01;36
#*,v 01;36
.bak 01;33
.BAK 01;33
.old 01;33
.OLD 01;33
.org_archive 01;33
.off 01;33
.OFF 01;33
.dist 01;33
.DIST 01;33
.orig 01;33
.ORIG 01;33
.swp 01;33
.swo 01;33
*,v 01;33

# The brightmagenta (Solarized: purple) color is free for you to use for your
# custom file type
.gpg 34
.gpg 34
.pgp 34
.asc 34
.3des 34
.aes 34
.enc 34
.sqlite 34


```
修改`~/.bashrc`配置，我安装了`oh-my-zsh`，所以我修改的文件是`~/.zshrc`。加入如下配置：
```bash
# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'
 
    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi
 
# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```
[dircolors-solarized github地址](https://github.com/seebi/dircolors-solarized)最后效果如下：
![dircolors-solarized配色](http://7xn9y9.com1.z0.glb.clouddn.com/note_Ubuntu%2014.04%20多终端窗口合并02.png)

#### 分屏操作
在终端上的某个终端窗口右键，就可以`Split Horizontally`以及`Split Vertically`进行屏幕分割，这个操作可以类推，效果类似于：
![最终分屏效果](http://7xn9y9.com1.z0.glb.clouddn.com/note_Ubuntu%2014.04%20多终端窗口合并03.png)

