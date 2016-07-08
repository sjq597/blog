title: Git 撤销冲突，用远端代码覆盖本地
date: 2016-07-07 16:09:32
tags: [Git, Linux]
categories: Git
---
本地改了一个代码，结果忘了推送到远端，晚上下班之后又改动了代码推送到了远端，结果第二天来公司一pull代码，果断悲剧了，冲突了。其实解冲突到没什么，但是其实我是想以远端的代码为准，完全丢弃我本地的代码更改，但是此时已经位于合并分支上，也没办法
```
git stash	# 将更改的代码压栈
```
如果你已经处在合并代码的分支了，不合并完代码`commit`是没办法使用压栈命令把代码还原到更改之前的，当然暴力的办法当然是直接把项目删了，重新`clone`到本地，但是这个方法显然不是很好，如果项目特别大就很慢了:
```
➜  data git:(init) ✗ git revert 5d49773
error: revert is not possible because you have unmerged files.
提示：请在工作区改正文件，然后酌情使用 'git add/rm <文件>' 命令标记
提示：解决方案并提交。
fatal: 还原失败
➜  data git:(init) ✗ git stash 
shell/ordercenter/order_snap/order_common_scene_snap.sh: needs merge
shell/ordercenter/order_snap/order_common_scene_snap.sh: needs merge
shell/ordercenter/order_snap/order_common_scene_snap.sh: unmerged (0abc52f84a64ad5d5e8157303b0b50cd095e4319)
shell/ordercenter/order_snap/order_common_scene_snap.sh: unmerged (eb8bd3095d572d0c6bde26eec1b7de0c3359ac78)
shell/ordercenter/order_snap/order_common_scene_snap.sh: unmerged (dce00518150b3a26f5fb04315cd7b3fedf1c2396)
fatal: git-write-tree: error building trees
无法保存当前索引状态
```
网上查阅了一下，有个命令可以将代码还原到指定历史:
```
➜  data git:(init) ✗ git fetch --all
正在获取 origin
➜  data git:(init) ✗ git reset --hard origin/init
HEAD 现在位于 612af0e update
➜  data git:(init) git pull
Already up-to-date.
```
**备注:**git fetch 只是下载远程的库的内容，不做任何的合并 git reset 把HEAD指向刚刚下载的最新的版本
