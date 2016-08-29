title: Ubuntu 16.04 Pycharm无法切换搜狗输入法
date: 2016-08-29 17:21:16
tags: [Linux,Python]
categories: Linux使用
---
出去玩了一趟，回来发现有的软件无法切换中文输入法了，总的来说就是浏览器还有系统自带的一些应用好像没啥影响，但是我安装的第三方然简就有的不行了，包括我又重装了输入法等还是不行，最后的解决办法可能和输入法模块的路径有关，不知道为啥找不到了，所以只好在`Pycharm`启动文件里面去手动指定了,我的安装路径启动文件为:
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
