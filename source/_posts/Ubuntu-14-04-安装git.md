title: Ubuntu 14.04 安装git
date: 2015-10-25 11:24:51
tags: [Git,开发工具]
categories: 开发环境
---
首先测试一下电脑安装git没有
```bash
git --version
```
### 安装git
没有安装按照以下步骤来：
```bash
sudo apt-get install git-core git-gui git-doc gitk
```
安装完毕看看是否安装成功，如果安装成功的版本低于1.9.5则说明Ubuntu版本太低，进行如下操作:
```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
```
安装成功之后再次测试，版本就应该是比较高了。

### 配置git
基本配置
```bash
git config --global user.name "zhang.san"             # 请换成你自己的名字，除非你凑巧也叫zhang.san
git config --global user.email "zhang.san@163.com"    # 同上
git config --global merge.tool "kdiff3"               # 要是没装KDiff3就不用设这一行
git config --global push.default simple               # 要是你非要用低版本的Git（比如1.7.x），好吧，那就不设simple设current，否则你的Git不支持
git config --global core.autocrlf false               # 让Git不要管Windows/Unix换行符转换的事
git config --global gui.encoding utf-8                # 避免git gui中的中文乱码
git config --global core.quotepath off                # 避免git status显示的中文文件名乱码
```
如果你不习惯使用`Vim`,可以修改git默认的编辑器
```bash
git config --global core.editor nano
```

### 设置SSH
要和git服务器打交道，还要设置ssh，如果你之前已经有则不需要再次生成,命令:
```bash
ssh-keygen -t rsa -C "zhang.san@163.com"
```
然后一路回车，不要输入任何密码之类，生成ssh key pair。如果在Linux上，需要把其中的私钥告诉本地系统
```bash
ssh-add ~/.ssh/id_rsa
```
查看公钥内容
```bash
cat ~/.ssh/id_rsa.pub
```
最后把公钥的内容复制到git服务器上，以github为例
登陆github，在`Settings-->SSH keys--> Add SSH key`,把公钥的内容复制到key里面，title不用管。(windows下的用户目录找到.ssh文件夹进去就可以看到)的内容paste进去。不需要填title，title会自动生成。
要是Git服务器报“不是有效的key”之类的错误，可能是你没去除注意去除多余的回车符，也可能是paste之前copy的时候，没copy最开头的ssh-rsa这几个字。
测试一下成功了没
```bash
ssh -T git@github.com
Warning: Permanently added the RSA host key for IP address '192.30.252.129' to the list of known hosts.
Hi zhang.san! You've successfully authenticated, but GitHub does not provide shell access.
```

