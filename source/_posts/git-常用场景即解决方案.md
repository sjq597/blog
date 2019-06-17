title: Git 常用场景即解决方案
date: 2017-02-22 20:09:41
tags: [Linux,Git]
categories: Git
---
熟悉了怎么使用git,在基本了解了git的原理的基础上，要解决一些办法其实是知道一些方案的，只是有时候我们对git的众多命令记得不是那么熟。其实常用的一些场景也就那么几个，出了问题也一般就那些，记录一下，方便日后查找。

### merge错了分支
有时候我们在自己的分支A上开发了一段时间，然后我们想把B分支merge到我们的分支A上，然后测试完没啥问题就把此次的结果一起merge到master上。但是在操作的时候不小心把master先merge到了分支A,如果再去merge分支B，那按照正常的流程，只能先把第一次的merge冲突解完了然后才能去merge分支B,然后再解冲突。这样就完全乱了，如何撤销某次merge呢,即让各个分支回到merge之前的状态?

* reset直接重置到merge之前的版本号

此方法比较暴力，即抛弃掉merge之前那个版本号之后的所有提交，如果你在merge之后没有做什么其他的操作，可以直接简单一点，舍弃掉后面的所有提交,不会产生新的commit.
```
git checkout branch-A
git reset --hard <merge前的版本号>
```
**备注:**其实reset的意思就是用把当前分支的指针设置为你制定的那个版本号，以此达到还原或者说撤销目的。


* git revert 撤销之前的merge操作

如果你在merge错了之后并没有立即发现，然后又提交了几次，然后才发现合并错了，这个时候就不能重置舍弃掉merge后面的提交了，可以使用`revert`只撤销掉merge那次的变更,此次操作会产生一个commit.
```
git revert -m <用1或2指定需要保留的主线> <需要撤销的合并的sha版本号>
```
**备注:** 分支的编号一般是按时间顺序决定的，parent分支是1，后面的那个分支是2.这个方法比较绕，一般不会用这个。

* git revert一般做法

一般我们的解决办法其实只是处理我们弄错了的分支，因为虽然我们合并错了，但是master,分支B都是好的，没有多余的提交的，我么你只需要把分支A的问题搞定就行了，此操作也会产生一个新的commit:
```
git revert <merge操作的版本号>
```

### Git仓库无权限
这个问题会经常出现，就算是很多用了好几年的程序员，有时候在出现这个问题的时候也不知道怎么解决，一般也就是去网上查一下，然后照着帖子里面弄一下，如果可以了，就再也不管了，如果不行，就接着找，试下一个方法，反正就这么一直试，试到后面总会莫名奇妙的解决了。如果实在不行可能就放大招重装系统或者git了。其实出现这个问题的根本原因就是ssh私钥和公钥的问题，要么是找不到，要么是找到了文件的权限不对.所以在出现这个问题的时候就知道怎么解决了.
首先得说明一下下面的两个命令的区别:
```
sudo git pull	// 读取的配置文件为/root/.ssh/id_rsa
git pull	// 读取的配置文件为~/.ssh/id_rsa
```
可以试一下有什么区别:

* ssh -vT git@gitlab.xxx.xxx.com

```
debug1: identity file /home/anonymous/.ssh/id_rsa type 1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /home/anonymous/.ssh/id_ed25519-cert type -1
```

* sudo ssh -vT git@gitlab.xxx.xxx.com

```
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_rsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /root/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
```
不同权限加载的文件是不一样的，所以为了知道问题出在哪，首先你得知道在和git仓库同步代码的时候加载的是哪个地方的秘钥文件.
知道了加载哪个地方的文件之后，还有一个很重要的问题需要知道,秘钥文件的权限必须是`600`,很多时候可能关键的问题还是因为秘钥文件的权限不对

### 切换相关
切换有很多，主要是分支切换，版本切换以及文件的切换

* 不同分支间文件切换

```
git checkout <branch_name> -- <paths>
```
