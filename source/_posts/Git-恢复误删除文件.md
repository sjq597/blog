title: Git 恢复误删除文件
date: 2016-07-07 16:37:59
tags: [Git, Linux]
categories: Git
---
有时候在项目里面不小心把某个文件删除了，并且代码也推送到远端了，或者我们删除了一个我们认为没啥用的文件,但是过了很长一段时间之后我们又想把这个文件找回来，当然笨一点的办法也有，就是去git的Web端找到删除那个文件的提交版本号，然后通过改动历史把那个文件内容复制，然后在本地新建那个文件，然后把内容复制过来，添加到项目中，自然这个文件也回来了，但是这个方法比较麻烦，下面介绍一下恢复一个被删掉的文件的过程
```
git log --graph

* commit e4985f8fdac2b14468c0348e88180b5348417d04
| Author: san.zhang <san.zhang@gmail.com>
| Date:   Fri Dec 18 18:03:44 2015 +0800
| 
|     xxxxx bugfix2
|  
* commit 5bb4f8e49f26e27991d9ab84812cd6eb5a0a3dd6
| Author: san.zhang <san.zhang@gmail.com>
| Date:   Fri Dec 18 17:27:21 2015 +0800
| 
|     xxxxx,buggix2,删除配置文件
|  
* commit d835c652209ad430decdef28498d03379b06ae06
| Author: san.zhang <san.zhang@gmail.com>
| Date:   Fri Dec 18 16:44:04 2015 +0800
| 
|     commit 1
```
在**commit**`5bb4f8e49f26e27991d9ab84812cd6eb5a0a3dd6`这次提交删错了文件,具体日志为：
```
* commit 5bb4f8e49f26e27991d9ab84812cd6eb5a0a3dd6
| Author: san.zhang <san.zhang@gmail.com>
| Date:   Fri Dec 18 17:27:21 2015 +0800
| 
|     xxxxx,buggix2,删除配置文件
```

这次提交版本号为:`5bb4f8e`,这个commit之前的commit为`d835c65`,也就是说在`5bb4f8e`这个版本中其实是没有这个文件的，在`d835c65`这个版本中，这个文件的状态为被删之前的样子，我们只要能把文件恢复到这个版本就可以了，怎么做呢？
假设被删的文件叫`test.conf`,恢复`test.conf`有两个办法：

```
git checkout "5bb4f8e~1" test.conf
git checkout d835c65 test.conf
```

其实这两个命令是一样的，`5bb4f8e~1`就是指这个commit的上一次，同理`~2`指的是前两次


