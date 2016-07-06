title: Hexo 主题同步
date: 2016-07-06 23:41:49
tags: [Git,Linux]
categories: 博客搭建
---
由于经常在两台电脑上同步博客，一般博客文件夹下面的文件我会单独同步到Git上的一个项目。但是碰上重装系统了，虽然博客的内容可以直接从Github上同步下来，但是主题没了，所以研究了一下主题同步的方法。

系统环境
```
➜  Blog git:(master) ✗ hexo -v
hexo: 3.2.0
hexo-cli: 1.0.1
os: Linux 4.4.0-28-generic linux x64
http_parser: 2.5.0
node: 4.1.1
v8: 4.5.103.33
uv: 1.7.4
zlib: 1.2.8
ares: 1.10.1-DEV
modules: 46
openssl: 1.0.2d
```
我用的主题是[jacmam](https://github.com/wuchong/jacman),顺便说一下，Hexo的主题列表地址为[Hexo主题列表](https://hexo.io/themes/)

### 主题同步
这里采取的是`fork + subtree`来实现同步，下面都是以`jacman`主题为例
* fork目标主题

jacman的git地址为:`https://github.com/wuchong/jacman.git`，然后我fork这个主题,于是我的项目的地址为:`https://github.com/sjq597/jacman`,然后我先把我本地的主题删掉:

* 主题集成同步

```
➜  Blog git:(master) ✗ sudo rm -r themes/jacman
➜  Blog git:(master) ✗ git remote add -f jacman https://github.com/sjq597/jacman
更新 jacman 中
warning: no common commits
remote: Counting objects: 1292, done.
remote: Total 1292 (delta 0), reused 0 (delta 0), pack-reused 1292
接收对象中: 100% (1292/1292), 2.81 MiB | 311.00 KiB/s, 完成.
处理 delta 中: 100% (665/665), 完成.
来自 https://github.com/sjq597/jacman
 * [新分支]          closeaside -> jacman/closeaside
 * [新分支]          gh-pages   -> jacman/gh-pages
 * [新分支]          master     -> jacman/master
 * [新分支]          site       -> jacman/site
 * [新标签]          v0.9.0     -> v0.9.0
➜  Blog git:(master) ✗ git subtree add --prefix=themes/jacman jacman master --squash
git fetch jacman master
来自 https://github.com/sjq597/jacman
 * branch            master     -> FETCH_HEAD
Added dir 'themes/jacman'
➜  Blog git:(master) ✗ git fetch jacman master 
来自 https://github.com/sjq597/jacman
 * branch            master     -> FETCH_HEAD
```
这样就把`jacman`作为了我的博客`Blog`项目的一个子项目了，子项目可以单独更新以及推送，同理，如果父项目中更新了子项目，那么这个更新也会推送到父项目，下面有一些常用的命令：
```
git commit -a -m 'update some'
git subtree push --prefix=themes/jacman/ jacman master
git push origin master # 顺便主项目也 push 了
```
或者单独推送子项目:
```
git subtree push -P themes/jacman/ jacman master
```
命令参考:
```
git subtree add -P <prefix> <commit>
git subtree add -P <prefix> <repository> <ref>
git subtree pull -P <prefix> <repository> <ref>
git subtree push -P <prefix> <repository> <ref>
git subtree merge -P <prefix> <commit>
git subtree split -P <prefix> [OPTIONS] [<commit>]
```
