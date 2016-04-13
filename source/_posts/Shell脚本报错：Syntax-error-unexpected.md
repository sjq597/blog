title: "Shell脚本报错：Syntax error: '(' unexpected"
date: 2015-11-10 23:51:37
tags: [Linux,Shell]
categories: Shell
---
执行脚本报错:
> Syntax error: "(" unexpected

与实际使用的Shell版本有关，判断方法可以使用：
```bash
⚡ ⇒ ls -l /bin/*sh
-rwxr-xr-x 1 root root 1021112 10月  8  2014 /bin/bash
-rwxr-xr-x 1 root root  121272  2月 19  2014 /bin/dash
lrwxrwxrwx 1 root root       4 10月  8  2014 /bin/rbash -> bash
lrwxrwxrwx 1 root root      22 10月 24 18:08 /bin/rzsh -> /etc/alternatives/rzsh
lrwxrwxrwx 1 root root       4 10月 24 14:21 /bin/sh -> dash
lrwxrwxrwx 1 root root       7 10月 24 14:21 /bin/static-sh -> busybox
lrwxrwxrwx 1 root root      21 10月 24 18:08 /bin/zsh -> /etc/alternatives/zsh
```
可以看到，`sh`果然被重定向到`dash`。因此，如果执行`./scirpt.sh`，使用的是`dash`。
避免报错方法很多，可以手动指定用：
```bash
bash script.sh
```
或者，在脚本第一行制定用什么Shell来执行：
```bash
#! /bin/bash
```
