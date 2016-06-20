title: Ubuntu an operation is already pending
date: 2016-02-06 12:06:29
tags: [Linux]
categories: Linux使用
---
在Ubuntu 14.04 上插硬盘，结果,无法挂载
```
An operation is already pending
```

解决方案，在终端中运行
```bash
gsettings set org.gnome.desktop.media-handling automount false
```
然后把硬盘拔了重新插上就可以挂载了。
