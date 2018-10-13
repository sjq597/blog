title: Ubuntu 14.04 配置JDK
date: 2015-10-06 17:45:00
tags: [jdk,开发环境,开发工具]
categories: [开发环境]
---
### Ubuntu 14.04 安装`jdk`
根据自己的系统下载对应的压缩包，我的是`Ubuntu 14.04`，需要下载的是`jdk-7u40-linux-x64.tar.gz`。具体地址去`oracle`官网。

* 解压压缩包：

>tar -xvf jdk-7u40-linux-x64.tar.gz

新建一个目录，将解压的文件移动到该目录中
>mkdir /usr/java
sudo cp -a jdk1.7.0_40/ /usr/java/

进入vi编辑器，编辑环境变量
>vi ~/.bashrc

**NOTE:**如果你还装了其他的`shell`，比如我装的是`oh-my-zsh`，那么需要修改对应的文件而不再是`.bashrc`，而是`.zshrc`。

在最后加入以下内容
>export JAVA_HOME=/usr/java/jdk1.7.0_40
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

**注意：** 等号之间不要有空格，否则会出错
最后还需要更新一下
>source ~/.bashrc

好了，在终端中可以测试一下jdk是否安装成功了
>java -version

输出结果为
>java version "1.7.0_40"
Java(TM) SE Runtime Environment (build 1.7.0_40-b43)
Java HotSpot(TM) 64-Bit Server VM (build 24.0-b56, mixed mode)

至此，`jdk`就安装好了。
