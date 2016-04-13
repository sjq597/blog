title: 命令行建github仓库
date: 2016-02-20 18:39:22
tags: [Git,Linux]
categories: Git
---
经常使用github的人应该不陌生，大多数人使用都是先在github的网站上建一个仓库，然后拷贝ssh或者http的地址，然后使用`git clone`命令来初始化项目，这样比较麻烦，其实可以直接通过命令行的方式来在github上建立一个仓库，然后直接把本地项目推送到github上的。

### 命令行建立仓库
1. 认证用户发一个POST就可以新建，于是使用curl构造POST，没有的自己安装：
```bash
curl -u 'username' https://api.github.com/user/repos -d '{"name":"RepoName"}'
```
2. 然后把刚才建的仓库的地址加到git配置里
```bash
git remote add origin git@github.com:username/RepoName.git
```
3. 推送到github
```bash
git push -u origin master
```

**注意:**username换成你在github上的注册名,RepoName就是你打算在github上建立的仓库名，注意不要替换user

通过这几步就可以在命令行里完成所有的操作。
