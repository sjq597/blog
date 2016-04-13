title: Git 获取远端指定分支
date: 2015-10-08 17:31:33
tags: [Git]
categories: [Git]
---
通过`git`远端仓库地址拉取项目，结果只有`master`分支，使用`git fetch`后没有用，还是没有拉取到其他的分支。

问题定位： 通过`git clone`获取的远端`git`库，只包含了远端`git`库的当前工作分支。如果还想获取其他的分支信息，可以按照以下步骤来。

* 查看远端分支

```bash
➜  data git:(master) git branch -r
  origin/HEAD -> origin/master
  origin/init
  origin/master
```

* 拉取远端指定分支

```bash
git checkout -b <本地分支> <远程分支>
```
例如我想拉取远端的`init`分支：
```bash
➜  data git:(master) git checkout -b init origin/init 
Branch init set up to track remote branch init from origin.
Switched to a new branch 'init'
```

* 查看是否成功

```bash
➜  data git:(init) ls  
build.sh  database  pay_shell  report  UDF          userprofile2
cron      design    README.md  shell   userprofile
```
**NOTE:** 如果本地分支已经存在，则不需要`-b`参数，`远程分支名`的名字就是你`git branch -r`所列出来的，诸如`origin/分支名`。
