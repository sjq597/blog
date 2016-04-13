title: Linxu下安装Hexo
date: 2015-10-05 23:28:05
tags: [Hexo,Linux,开发工具,开发环境]
categories: [博客搭建]
---
为了搭建博客，需要安装hexo，但是在Ubuntu 14.04下怎么都装不上，最后改用了淘宝的源

首先确保你安装了node.js
#### 开始安装：

```bash
npm install hexo -g
```

**NOTE:** 不过这样一直装不上，最后没办法改用了[taobao的npm源](https://npm.taobao.org/)
使用教程也很简单，有介绍：

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install [name]
```

所以如果安不上，试试上面的命令，然后接着安装：

```bash
npm install hexo -g  #-g表示全局安装, npm默认为当前项目安装
cnpm install hexo-cli -g
cnpm install hexo --save
```

#### 创建Hexo文件夹

```bash
#安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
hexo init <folder>
cd <folder>
cnpm install
```

最后的目录结构如下：

```
➜  hexo  tree -L 2
.
├── _config.yml
├── node_modules
│   ├── hexo
│   ├── hexo-generator-archive
│   ├── hexo-generator-category
│   ├── hexo-generator-index
│   ├── hexo-generator-tag
│   ├── hexo-renderer-ejs
│   ├── hexo-renderer-marked
│   ├── hexo-renderer-stylus
│   └── hexo-server
├── package.json
├── scaffolds
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source
│   └── _posts
└── themes
    └── landscape

15 directories, 5 files
```

#### 启动服务看效果

```bash
hexo server
```

然后访问`http://localhost:4000/`就可以打开网站了。


#### 将`Hexo`发布到`github`上

这里踩了很多坑，特别注意一下，因为大多数教程都是以2.x.x版本为例的，问题就在这里，3.x.
x的版本很不一样。所以很多教程根本不对，先看看我的环境：

```bash
➜  $  hexo -v
hexo: 3.1.1
os: Linux 3.13.0-32-generic linux x64
http_parser: 2.5.0
node: 4.1.1
v8: 4.5.103.33
uv: 1.7.4
zlib: 1.2.8
ares: 1.10.1-DEV
modules: 46
openssl: 1.0.2d
```

现在我们来重新看看如何发布一个项目到之前你的`github`博客仓库里，并且直接在`github`上显示

* 初始化一个项目

```bash
➜  hexo init note    # 初始化一个文件夹
➜  cd note            # 进入到note文件夹执行生成命令
➜  note  hexo generate
ERROR Local hexo not found in ~/docments/note
ERROR Try running: 'npm install hexo --save'
```

忘了执行install命令了。

```bash
➜  note  npm install
npm WARN optional dep failed, continuing fsevents@1.0.0
npm WARN optional dep failed, continuing fsevents@0.3.8
 
> dtrace-provider@0.6.0 install /home/junqiangshen/docments/note/node_modules/hexo/node_modules/bunyan/node_modules/dtrace-provider
> node scripts/install.js

hexo-renderer-ejs@0.1.0 node_modules/hexo-renderer-ejs
├── lodash@2.4.2
└── ejs@1.0.0

hexo-generator-index@0.1.3 node_modules/hexo-generator-index
├── object-assign@2.1.1
└── hexo-pagination@0.0.2 (utils-merge@1.0.0)

hexo-generator-tag@0.1.2 node_modules/hexo-generator-tag
├── object-assign@2.1.1
└── hexo-pagination@0.0.2 (utils-merge@1.0.0)

hexo-generator-category@0.1.3 node_modules/hexo-generator-category
├── object-assign@2.1.1
└── hexo-pagination@0.0.2 (utils-merge@1.0.0)

hexo-generator-archive@0.1.3 node_modules/hexo-generator-archive
├── object-assign@2.1.1
└── hexo-pagination@0.0.2 (utils-merge@1.0.0)

hexo-renderer-marked@0.2.5 node_modules/hexo-renderer-marked
├── object-assign@2.1.1
├── marked@0.3.5
├── strip-indent@1.0.1 (get-stdin@4.0.1)
└── hexo-util@0.1.7 (ent@2.2.0, bluebird@2.10.1, highlight.js@8.8.0)

hexo-renderer-stylus@0.3.0 node_modules/hexo-renderer-stylus
├── stylus@0.52.4 (css-parse@1.7.0, debug@2.2.0, sax@0.5.8, source-map@0.1.43, mkdirp@0.5.1, glob@3.2.11)
└── nib@1.1.0 (stylus@0.49.3)

hexo-server@0.1.2 node_modules/hexo-server
├── object-assign@2.1.1
├── open@0.0.5
├── mime@1.3.4
├── bluebird@2.10.1
├── morgan@1.6.1 (basic-auth@1.0.3, on-headers@1.0.0, depd@1.0.1, on-finished@2.3.0, debug@2.2.0)
├── connect@3.4.0 (utils-merge@1.0.0, parseurl@1.3.0, debug@2.2.0, finalhandler@0.4.0)
├── serve-static@1.10.0 (escape-html@1.0.2, parseurl@1.3.0, send@0.13.0)
├── compression@1.5.2 (vary@1.0.1, bytes@2.1.0, on-headers@1.0.0, debug@2.2.0, compressible@2.0.5, accepts@1.2.13)
└── chalk@0.5.1 (ansi-styles@1.1.0, escape-string-regexp@1.0.3, supports-color@0.2.0, strip-ansi@0.3.0, has-ansi@0.1.0)

hexo@3.1.1 node_modules/hexo
├── hexo-front-matter@0.2.2
├── pretty-hrtime@1.0.0
├── abbrev@1.0.7
├── titlecase@1.0.2
├── archy@1.0.0
├── text-table@0.2.0
├── tildify@1.1.1 (os-homedir@1.0.1)
├── strip-indent@1.0.1 (get-stdin@4.0.1)
├── hexo-i18n@0.2.1 (sprintf-js@1.0.3)
├── moment-timezone@0.3.1
├── bluebird@2.10.1
├── minimatch@2.0.10 (brace-expansion@1.1.1)
├── through2@1.1.1 (xtend@4.0.0, readable-stream@1.1.13)
├── swig-extras@0.0.1 (markdown@0.5.0)
├── chalk@1.1.1 (escape-string-regexp@1.0.3, ansi-styles@2.1.0, supports-color@2.0.0, has-ansi@2.0.0, strip-ansi@3.0.0)
├── warehouse@1.0.3 (graceful-fs@4.1.2, cuid@1.2.5, JSONStream@1.0.6)
├── js-yaml@3.4.2 (esprima@2.2.0, argparse@1.0.2)
├── hexo-cli@0.1.8 (minimist@1.2.0)
├── moment@2.10.6
├── nunjucks@1.3.4 (optimist@0.6.1, chokidar@0.12.6)
├── cheerio@0.19.0 (entities@1.1.1, dom-serializer@0.1.0, css-select@1.0.0, htmlparser2@3.8.3)
├── bunyan@1.5.1 (safe-json-stringify@1.0.3, mv@2.1.1, dtrace-provider@0.6.0)
├── swig@1.4.2 (optimist@0.6.1, uglify-js@2.4.24)
├── hexo-util@0.1.7 (ent@2.2.0, highlight.js@8.8.0)
├── hexo-fs@0.1.4 (escape-string-regexp@1.0.3, graceful-fs@4.1.2, chokidar@1.1.0)
└── lodash@3.10.1
```

再来执行一下：

```bash
➜  note  hexo g            
INFO  Files loaded in 325 ms
INFO  Generated: js/script.js
INFO  Generated: fancybox/jquery.fancybox.pack.js
INFO  Generated: fancybox/jquery.fancybox.js
INFO  Generated: fancybox/jquery.fancybox.css
INFO  Generated: fancybox/helpers/jquery.fancybox-thumbs.js
INFO  Generated: fancybox/helpers/jquery.fancybox-thumbs.css
INFO  Generated: fancybox/helpers/jquery.fancybox-media.js
INFO  Generated: fancybox/helpers/jquery.fancybox-buttons.js
INFO  Generated: fancybox/helpers/jquery.fancybox-buttons.css
INFO  Generated: fancybox/helpers/fancybox_buttons.png
INFO  Generated: fancybox/fancybox_sprite@2x.png
INFO  Generated: fancybox/fancybox_sprite.png
INFO  Generated: fancybox/fancybox_overlay.png
INFO  Generated: fancybox/fancybox_loading@2x.gif
INFO  Generated: fancybox/fancybox_loading.gif
INFO  Generated: fancybox/blank.gif
INFO  Generated: css/style.css
INFO  Generated: css/images/banner.jpg
INFO  Generated: css/fonts/fontawesome-webfont.woff
INFO  Generated: css/fonts/fontawesome-webfont.ttf
INFO  Generated: css/fonts/fontawesome-webfont.svg
INFO  Generated: css/fonts/fontawesome-webfont.eot
INFO  Generated: css/fonts/FontAwesome.otf
INFO  Generated: 2015/09/29/hello-world/index.html
INFO  Generated: archives/index.html
INFO  Generated: archives/2015/index.html
INFO  Generated: archives/2015/09/index.html
INFO  Generated: index.html
INFO  28 files generated in 1.01 s
```

看一下文件目录结构：

```bash
➜  note  ls
_config.yml  node_modules  public     source
db.json      package.json  scaffolds  themes
```

* 修改配置文件_config.yml

```bash
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: LittleQ
subtitle: 笔记分类整理
description: Java Web
author: LittleQ
language: zh-CN
timezone: Asia/Chongqing
```

**NOTE:** 主要解释一下这个时区`timezone`,我选的是重庆，具体想选择可以去这里[维基百科时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。

```bash
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://sjq597.github.io/ 
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```

**NOTE:** `url`就填你的`github`博客地址就行，其他的不要改。

```bash
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: http://github.com/sjq597/sjq597.github.io.git
  branch: master
```

**NOTE:** `type`填`git`，不要填`github`，网上的教程都是填`github`，但是这是`hexo 2.x.x`的方法，对于`3.x.x`的版本，必须填`git`。`repository`也不要填`ssh`地址，要填`https`地址，并且把`https`改为`http`，分支都是`master`。

#### 发布到`github`

发布到`github`之前还必须要安装一个东西：

```bash
➜  note  npm install hexo-deployer-git --save
npm WARN optional dep failed, continuing fsevents@1.0.0
hexo-deployer-git@0.0.4 node_modules/hexo-deployer-git
├── moment@2.10.6
├── chalk@0.5.1 (ansi-styles@1.1.0, escape-string-regexp@1.0.3, supports-color@0.2.0, strip-ansi@0.3.0, has-ansi@0.1.0)
├── hexo-util@0.1.7 (ent@2.2.0, bluebird@2.10.1, highlight.js@8.8.0)
├── swig@1.4.2 (optimist@0.6.1, uglify-js@2.4.24)
└── hexo-fs@0.1.4 (escape-string-regexp@1.0.3, graceful-fs@4.1.2, bluebird@2.10.1, chokidar@1.1.0)
```

然后就可已发布了。
```bash
➜  note  hexo d                              
INFO  Deploying: git
INFO  Clearing .deploy folder...
INFO  Copying files from public folder...
On branch master
nothing to commit, working directory clean
Username for 'https://github.com': sjq597
Password for 'https://sjq597@github.com': 
To http://github.com/sjq597/sjq597.github.io.git
 + 5183f8b...aef1a55 master -> master (forced update)
Branch master set up to track remote branch master from http://github.com/sjq597/sjq597.github.io.git.
INFO  Deploy done: git
```

#### 常见问题

* YAMLException

```bash
FATAL Something\'s wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
YAMLException: can not read a block mapping entry; a multiline key may not be an implicit key at line 13, column 1:
    # URL
    ^
```

**NOTE:** YML语法校验比较严格，冒号后面必须要有个空格，切记。配置文件里所有的修改地方都要记得在冒号后空格一下。

* Deployer not found: github

```bash
➜  note  hexo d           
ERROR Deployer not found: github
```
**NOTE:** 参见前面说的，`3.x.x`版本需要把`type`设置为`git`而不是`github`,并且仓库地址要写`git`的`https`地址，并且要改为`http`开头。

然后再去访问你在`github`上的博客地址，就会发现变成了`hexo`的`index.html`。