title: Git 常用命令
date: 2015-10-08 17:58:25
tags: [Git]
categories: [Git]
---
### 初级命令

#### 拉取项目
初次拉取项目也就是克隆项目
```bash
➜  docments  git clone git@repository_url/project_name.git
Cloning into 'project_name'...
remote: Counting objects: 14930, done.
remote: Compressing objects: 100% (5769/5769), done.
remote: Total 14930 (delta 10903), reused 12396 (delta 9013)
Receiving objects: 100% (14930/14930), 127.75 MiB | 11.20 MiB/s, done.
Resolving deltas: 100% (10903/10903), done.
Checking connectivity... done.
```
**注意：** `git@repository_url/project_name.git`就是`git`服务器上项目的`ssh`地址
另外项目更新需要用到两个命令:
```bash
git fetch   # 拉取远端数据到本地，不和并
git pull    # 相当于git fetch 和 git merge
```

#### 新建分支
把项目从服务器上拉取下来之后，一般都需要建立一个自己的分支方便开发，比如新建一个本地`test`分支
```bash
➜  data git:(master) git checkout test
```
**注意：** 如果你只是想看看项目里的分支代码，不想新建一个分支，有时候拉取的分支可能没有包括你想要看的分支，这个时候可以按照下面来做：
```bash
➜  data git:(master) git branch -r
  origin/HEAD -> origin/master
  origin/init
  origin/master
➜  data git:(master) git checkout -b init origin/init 
Branch init set up to track remote branch init from origin.
Switched to a new branch 'init'
```
如果有时候建分支忘了指定本地分支和远端的哪个分支对应，需要设定分支的远端对应分支可以这样，假设我想让我的本地master分支追踪远端的master分支：
```bash
git branch --track master origin/master
```

#### 提交修改
添加一个新文件，`git`不会自动跟踪，需要手动
```bash
git add file1 file2 file3       # 把file1 file2 file3添加到git索引中
git add .                       # 把当前项目中的所有文件添加到git索引中，
git commit -am "add thefile"    # 添加新文件到本地暂存区
```

#### 提交代码
```bash
git push origin test
```
关于`git push`使用还有很高级的用法，具体参见[git push 详解](http://www.yiibai.com/git/git_push.html)

#### 忽略文件
编辑项目下的`.gitignore`文件，添加
```
*.txt   # 忽略所有的txt文本文件
dir/    # 忽略项目根目录下dir文件夹里所有的文件
```

#### 指定关联分支
例如远端有个Python分支,那么想让本地分支追踪远端的Python分支，可以：
```bash
# 将本地的Python和远端仓库origin的Python分支关联
git branch --set-upstream-to=origin/Python Python
```

#### 查看某次修改内容
```bash
git show commit_id
```

#### 配置git

```bash
# 全局配置
git config --global user.name yourname
git config --global user.email emailaddress

# 本地配置，进到具体的项目目录下
git config --local user.name yourname
git config --local user.email emailaddress

# 指定git编辑器
git config --global core.editor vim
```

