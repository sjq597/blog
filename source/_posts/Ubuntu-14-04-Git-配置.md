title: Ubuntu 14.04 Git 配置
date: 2015-10-06 17:24:32
tags: [Git,开发环境,开发工具]
categories: [开发环境]
---
### 安装Git
首先测试一下电脑安装git没有
```bash
git --version
```

没有安装则按照以下步骤来
```bash
sudo apt-get install git-core git-gui git-doc gitk
```

安装完毕看看是否安装成功，如果安装成功的版本低于1.9.5则说明Ubuntu版本太低，进行如下操作
```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
```
安装成功之后再次测试，版本就应该是比较高了。

以上安装的是git，但是没有图形化界面，如果需要图形化界面，需要安装工具KDiff3
```bash
sudo apt-get install kdiff3
```

安装好以上几个基本的组件之后，还需要做一些简单的配置
```bash
git config --global user.name "san.zhang"            # 请换成你自己的名字，除非你凑巧也叫san.zhang
git config --global user.email "san.zhang@qq.com"   # 同上
git config --global merge.tool "kdiff3"             # 要是没装KDiff3就不用设这一行
git config --global push.default simple             # 要是你非要用低版本的`Git`（比如1.7.x），好吧，那就不设`simple`设`current`，否则你的`Git`不支持
git config --global core.autocrlf false             # 让Git不要管Windows/Unix换行符转换的事
git config --global gui.encoding utf-8              # 避免git gui中的中文乱码
git config --global core.quotepath off              # 避免git status显示的中文文件名乱码
git config --global core.editor nano                # 辑器，默认是vim，但是可以自己配置成其他的，可以使用nano 
```

### 设置SSH
如果要跟Git服务器打交道，还要设ssh。注意，不要在跳板机等Ops/IT已经为你设了`ssh key pair`的地方做下面的操作。
注意，少数童鞋如果以前为连接`GitHub/oschina`等已经生成过`ssh key pair`，这里不必再次生成，复用即可。
在`Linux`的命令行下，或`Windos`上`Git Bash`命令行窗口中（总之不要用iOS），键入：
```bash
ssh-keygen -t rsa -C "san.zhang@qq.com"
```

然后一路回车，不要输入任何密码之类，生成`ssh key pair`。如果在`Linux`上，需要把其中的私钥告诉本地系统：
```bash
ssh-add ~/.ssh/id_rsa
```

再把其中公钥的内容复制到`Git`服务器上。具体方法是：
显示ssh公钥的内容
```bash
cat ~/.ssh/id_rsa.pub
```

密钥内容如下：
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCE6avWtobESaS6nngDJnLFPPHZ+enhp+twpRw/t+9uaM/IFRhgP8YO09jG4Vdq8h5tzUe+ijM4b/rtVNAROCngkegrefopMwcqVNxQiFP/9Dl67/qOxorcYWhizQLjIzAdQxMGoNTebozjbLElWLO1pramWaK+nqO1PQL13olUinZa1Hxhv3XTCpODoPdz1woyVfYaPu4knjODQp2E3aawtmeZ5A7EJP7696XWi1tjK44iMWwZMWTOYbSGTyXq62xT5YfVmQFwxhG5tJYD6h27R65b0/WKOM7Y8cwVmo9RqpgFRJ5EPd42Fr6pjyBkPOGpVQkUn+V/GVpKrC+LWIJ san.zhang@qq.com
```

打开`Github`网页，具体位置自行百度。点击`Add SSH Key`，然后把刚才`ssh`公钥`id_rsa.pub`（`windows`下的用户目录找到`.ssh`文件夹进去就可以看到）的内容paste进去。不需要填`title`，`title`会自动生成。
要是`Git`服务器报“不是有效的`key`”之类的错误，可能是你没去除注意去除多余的回车符，也可能是`paste`之前`copy`的时候，没`copy`最开头的`ssh-rsa`这几个字。
