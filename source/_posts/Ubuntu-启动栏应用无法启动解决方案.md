title: Ubuntu 启动栏应用无法启动解决方案
date: 2016-05-29 19:36:35
tags: [Linux,Shell]
categories: Linux使用
---
最近刚装好了Ubuntu16.04，好不容易配置好了开发环境，突然发现IDEA,和PyCharm无法从启动栏和Dash菜单中启动，摸索来一阵之后，查阅资料发现来问题所在,记录一下，也希望给其他人一点参考，简单来说就是，环境变量不对导致无法启动。
以IntelliJ IDEA为例，我的IDEA和Pycharm都只能从终端启动，很不方便，即是从终端启动了，我把图标固定在左侧的快速启动栏，仍然启动不了，下面看看解决办法。
首先去`/usr/share/applications`文件夹看看，找到你的程序，双击，如果能够启动则应该没是没有问题的，但是双击不能启动，则即使你把图标固定到快速启动栏，程序仍然启动不了。
双击`IntelliJ IDEA`图标，提示报错信息，找不到JDK,问题就出在这里，但是奇怪的是我明明装了JDK,也设置来环境变量，怎么会找不到呢？于是我用vim查看了一下这个启动项的具体配置:
```
$ cd /usr/share/applications
$ cat jetbrains-idea.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=IntelliJ IDEA
Icon=/usr/dev/idea-IU-139.1117.1/bin/idea.png
Exec="/usr/dev/idea-IU-139.1117.1/bin/idea.sh" %f
Comment=Develop with pleasure!
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-idea
```
可以看到启动程序调用的脚本路径`/usr/dev/idea-IU-139.1117.1/bin/idea.sh`，这个就是我IDEA的解压安装路径，但是奇怪的是我从终端启动也是运行的这个脚本，就可以启动，为什么系统调用就不行呢？而且错误信息提示的是找不到JDK,后来我才明白，原来我安装了zsh,所以设置在`~/.zshrc`里面的环境变量并不能对bash启动的程序生效，所以找不到JDK。
既然明白了问题所在，解决办法也就有了,要么把环境变量设置成系统级别的环境变量，要么直接在对应的启动脚本中加入JDK的路径:
```
$ cat /usr/dev/idea-IU-139.1117.1/bin/idea.sh
# ---------------------------------------------------------------------
# Locate a JDK installation directory which will be used to run the IDE.
# Try (in order): IDEA_JDK, JDK_HOME, JAVA_HOME, "java" in PATH.
# ---------------------------------------------------------------------
if [ -n "$IDEA_JDK" -a -x "$IDEA_JDK/bin/java" ]; then
  JDK="$IDEA_JDK"
elif [ -n "$JDK_HOME" -a -x "$JDK_HOME/bin/java" ]; then
  JDK="$JDK_HOME"
elif [ -n "$JAVA_HOME" -a -x "$JAVA_HOME/bin/java" ]; then
  JDK="$JAVA_HOME"
else
  JAVA_BIN_PATH=`which java`
.......
```
分析启动脚本，反正就是找不到JAVA_HOME,JDK的路径了,简单粗暴的直接在检查变量之前定义好,在注释下面，条件判断的前面加上JAVA的配置
```
JAVA_HOME=/usr/dev/jdk1.7.0_40
JRE_HOME=${JAVA_HOME}/jre
```
这样再点击图标就可以运行了。
