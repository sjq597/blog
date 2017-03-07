title: Ubuntu 16.04 软件无法切换搜狗输入法
date: 2016-08-29 17:21:16
tags: [Linux,Python]
categories: Linux使用
---
出去玩了一趟，回来发现有的软件无法切换中文输入法了，总的来说就是浏览器还有系统自带的一些应用好像没啥影响，但是我安装的第三方然简就有的不行了，包括我又重装了输入法等还是不行，最后的解决办法可能和输入法模块的路径有关，不知道为啥找不到了。
会出现问题的软件一般为`WPS, IDEA, PyCharm`这些，所以只好在`Pycharm`启动文件里面去手动指定了,我的安装路径启动文件为:
```
/usr/dev/pycharm-2016.2/bin/pycharm.sh
```
修改文件，在顶端加入以下内容:
```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
然后重启`Pycharm`就可以切换搜狗输入法了。

但是有时候`WPS`的中文输入法也不起作用，这个时候用这个方法去改`/usr/bin/wps`，然后打开文档也不起作用，最后实验了发现，要想用`wps`编辑输入中文，必须是从应用那先打开`wps`软件，然后从软件中打开一个文档才行，如果直接用右键，选择`wps`打开`xls`的文档是不行的。

**备注:**如果你之前可以，但是照这个方法弄了以后还不行，那么你需要运行一下`fcitx`诊断命令:
```
fcitx-diagnose
```
一般可以发现问题所在，如果没啥异常，那么你需要完全卸载`fcitx`,然后重新装一遍就好了，卸载命令如下:
```
sudo apt-get purge  fcitx
sudo apt-get autoremove 
```
然后重新安装一下`fcitx`,为了保险起见，你可以把搜狗也卸载了重装。
