title: fedroa 25 开发常用配置及问题
date: 2017-02-13 18:04:05
tags: [Linux]
categories: Linux使用
---
尝鲜使用了一段时间的`fedroa`,大概使用了一个月吧，之前一直是使用的`Ubuntu 16.04`,想着和服务器上的`CentOS`尽量保持一致就装了个`Fedroa 25`最为日常开发机器，大概用了一个月就放弃了，说句实话，不是很好用,引用一句在`stackoverflow`上的答案吧，我觉得确实说的很有道理，问题大概是`Ubuntu和Fedroa`有什么区别，我相信很多同时用过这两个系统的人都可能会问这个问题，那就是作为桌面开发机来说，到底选那个，或者又说这两者都是桌面环境的`Linux`,哪个更适合作为日常开发使用?
```
Ubuntu的使命是使Linux对人更友好，更易于交互
```
其实你都不可否认，`Ubuntu`应该是目前为止非常稳定的一个`Linux桌面开发环境`,这句话在我使用了一个月的`Fedroa`之后感受尤为深刻。

虽然也能用。碰到了各种问题，花了很多的时间，所以想记下来这些解决的笔记。

##
系统Fedroa 25 64bit,以下操作都是在此基础上，其他系统仅做参考。可以先把源设置成阿里的源，那个源比较快。

### mysqldb-python

安装`mysqldb`报错,错误信息如下:
```
 gcc: error: /usr/lib/rpm/redhat/redhat-hardened-cc1: No such file or directory
  error: command 'gcc' failed with exit status 1
```

解决方案在`stackoverflow`上,链接为:
[http://stackoverflow.com/questions/34624428/g-error-usr-lib-rpm-redhat-redhat-hardened-cc1-no-that-file-and-directory](http://stackoverflow.com/questions/34624428/g-error-usr-lib-rpm-redhat-redhat-hardened-cc1-no-that-file-and-directory)

### vim

fedora直接安装vim是会失败的，这个实在是太操蛋了解决方案,先更新系统自带`vi`,然后才能装.
好不容易装好了之后发现，vim里面的东西没有办法用鼠标选中复制到剪贴板，解决方案:
[http://vi.stackexchange.com/questions/84/how-can-i-copy-text-to-the-system-clipboard-from-vim](http://vi.stackexchange.com/questions/84/how-can-i-copy-text-to-the-system-clipboard-from-vim)

### 右键菜单打开终端

之前用`Ubuntu`一直没觉得这个功能有多么贴心，直到`Fedroa`中右键菜单没了这个才发现没了这个功能好麻烦，有时候就像在当前目录打开一个终端，还必须打开一个然后一路`cd`到当前目录，解决方案:
[right clik open terminnal](https://www.if-not-true-then-false.com/2011/nautilus-open-in-terminal-on-fedora-centos-red-hat-rhel/)

### 启动栏快捷键

有时候需要把常用的一些应用固定在左侧启动栏，但是有些解压的软件好像无法`Add favourate`,此时可以手动编辑
```
~/.local/share/applications
```
下面的`xxxx.desktop`文件，照着写一个就行.
然后写完了可以用系统自带命令验证一下:
```
desktop-file-install xxx.desktop
```

### sougo中文输入法

中文社区有源:https://www.fdzh.org/blog/2015/10/18/fedora-sougoupinyin/

### 同步盘

公司把所有外网的网盘封了，内部提供的网盘是`ownCloud`,安装客户端程序:
地址:[https://software.opensuse.org/download/package?project=isv:ownCloud:desktop&package=owncloud-client](https://software.opensuse.org/download/package?project=isv:ownCloud:desktop&package=owncloud-client)

### 取消左下角托盘

TopIcons
```
dnf install gnome-tweak-tool
```
然后照着github上来:
https://github.com/phocean/TopIcons-plus

### fedroa截屏

The default behaviour when pressing the PrintScreen key is to automatically place your screenshot in the Pictures folder in your home directory (i.e. "~/Pictures"). The click and the flash mean that the screenshot has been taken, so just check the Pictures folder for your screenshot.

Other than just the "Print Screen' key, which saves your whole Desktop to the Pictures folder, GNOME3 also has the following shortcuts enabled by default for screenshot actions:

PrintScreen -- Takes a screenshot of your entire desktop and saves it to the Pictures folder.
```
Alt + PrintScreen -- Saves a screenshot of the focused window to the Pictures Folder
Shift + PrintScreen -- Lets you select an area of the screen, and saves to the Pictures Folder
Ctrl + PrintScreen -- Takes a screenshot of your entire desktop and copies it to the clipboard.
Ctrl + Alt + PrintScreen -- copies a screenshot of the focused window to the clipboard.
Ctrl + Shift + PrintScreen -- Lets you select an area of the screen, and copies it to the clipboard.
Ctrl + Shift + Alt + R -- Records a Screencast) of your entire desktop and saves it to your Videos folder.
```

### fedroa ssh server
安装完之后执行:
```
systemctl start sshd
```

### 安装mysql
这个最蛋疼,装完了还有一堆配置非常的麻烦。
https://www.if-not-true-then-false.com/2010/install-mysql-on-fedora-centos-red-hat-rhel/
如果无法初始化用户的密码，可以在mysql中试试下面的命令:
```
update user set authentication_string=password('Root@123') where user='root' and host='localhost';
FLUSH PRIVILEGES;
```

**备注:**其实可以多折腾一下，但是说实话吧，这个系统的桌面版真的不好用，装个软件非常不方便，尤其是mysql.
