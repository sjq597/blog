title: Ubuntu 14.04 安装vim-gnome失败解决办法
date: 2015-11-15 20:54:00
tags: [Linux,Vim]
categories: Vim
---
每次想拷贝Vim里的内容到其他地方,开始都是用鼠标的,但是后来发现,如果要复制大片内容,终端里没有显示出来就没辙了,所以网上搜了怎么把Vim里的内容复制到剪贴板.
网上搜了一下如何把Vim里的内容复制到系统剪贴板,大部分都是建议安装`vim-gnome`这个插件,仅针对于`Ubuntu`桌面环境,其他的我没有测试过:
```bash
sudo apt-get install vim-gnome
```
结果报错了,报错信息如下:
```
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 vim-gnome : Depends: vim-gui-common (= 2:7.4.052-1ubuntu3) but it is not going to be installed
             Depends: vim-common (= 2:7.4.052-1ubuntu3) but 2:7.4.826-1 is to be installed
             Depends: vim-runtime (= 2:7.4.052-1ubuntu3) but 2:7.4.826-1 is to be installed
E: Unable to correct problems, you have held broken packages.
```
看上面的意思是这个插件依赖的几个库没有安装,手动安装一下:
```bash
sudo apt-get install vim-gui-common
```
然后再试试可不可以安装`vim-gnome`:
```bash
sudo apt-get install vim-gnome
```
果不其然又报错了:
```
The following packages have unmet dependencies:
 vim-gnome : Depends: vim-runtime (= 2:7.4.052-1ubuntu3) but 2:7.4.826-1 is to be installed
```
这个意思就是说依赖的`vim-runtime`是一个老的版本,但是安装的是一个更加新的版本.那我们来手动装一下:
```bash
sudo apt-get install vim-runtime
```
看看提示信息:
```
vim-runtime is already the newest version.
The following packages were automatically installed and are no longer required:
  g++-4.8 libstdc++-4.8-dev linux-headers-3.13.0-32
  linux-headers-3.13.0-32-generic linux-image-3.13.0-32-generic
  linux-image-extra-3.13.0-32-generic
Use 'apt-get autoremove' to remove them.
0 upgraded, 0 newly installed, 0 to remove and 16 not upgraded.
```
好了,找到问题的关键所在了,我之前升级了`gcc 4.9.2`,我们需要把这些老的包移除掉,不要漏掉了,也可以逐一卸载:
```bash
sudo apt-get autoremove g++-4.8 libstdc++-4.8-dev linux-headers-3.13.0-32 linux-headers-3.13.0-32-generic linux-image-extra-3.13.0-32-generic
```
然后强制安装一下:
```bash
sudo apt-get install vim-runtime
```
好了,然后在Vim里的非`INSERT`模式下使用`"+y`就可以把Vim里的内容复制到系统剪贴板了,注意记得重启Vim才能生效.
