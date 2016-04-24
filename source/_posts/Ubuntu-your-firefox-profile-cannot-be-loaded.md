title: Ubuntu your firefox profile cannot be loaded
date: 2016-04-23 13:43:51
tags: Linux
categories: Linux使用
---
一时手贱把`~/tmp/`文件夹删了，结果火狐打不开了,最后在网上找了很久才找到解决方案，记录一下防止下次手贱了。
大部分文章只提到，把`~/.mozilla`文件的权限更改成当前用户和用户组，就可以了：
* 察看.mozilla文件件的权限属性

```
$ ls -l
```
如果`~/.mozilla`文件夹的属性是`root`则改变文件夹的属性为当前用户。命令如下：
* 需改文件夹权限

```
$ sudo chown -hR zhangsan:zhangsan ~/.mozilla/
```
如此操作后，其实firefox还是启动不了，因为，还有一个文件夹的属性仍然是root，这个文件夹就是`~/.cache/mozilla`。
```
$ cd ~/.cache
$ ls -l 
$ sudo chown -hR zhangsan:zhangsan mozilla/
```
经过如此折腾，双击firefox图标，才能正常启动firefox浏览器。
太坑爹了。
