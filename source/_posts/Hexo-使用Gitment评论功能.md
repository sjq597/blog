title: Hexo 使用Gitment评论功能
date: 2018-05-18 22:11:36
tags: [Hexo]
categories: 博客搭建
---
一直想整一下博客的评论系统,以前听说多说比较有名气．但是当我想搞的时候发现多说居然关闭了，找了一圈发现除了`Gitment`这个`Github`自家的东西比较靠谱,所以就折腾了一下,期间碰到不少问题.

## 安装Gitment

### 安装模块
在你的blog根目录安装
```
npm i --save gitment
```

### 申请应用
首先去[New OAuth App](https://github.com/settings/applications/new)为你的博客应用一个密钥:
```
Application name:随便写
Homepage URL:这个也可以随意写,就写你的博客地址就行
Application description:描述,也可以随意写
Authorization callback URL:这个必须写你的博客地址
```
申请好之后点注册,然后就可以看到两个东西`ClientID`和`Client Secret`,后面会用到.

### 配置
下面就是配置`Gitment`,主要编辑在`themes/next/_config.yml`:
```
# Gitment
# Introduction: https://imsun.net/posts/gitment-introduction/
gitment:
  enable: true
  mint: true # RECOMMEND, A mint on Gitment, to support count, language and proxy_gateway
  count: true # Show comments count in post meta area
  lazy: false # Comments lazy loading with a button
  cleanly: false # Hide 'Powered by ...' on footer, and more
  language: # Force language, or auto switch by theme
  github_user: {you github user id}
  github_repo: 随便写一个你的公开的git仓库就行,到时候评论会作为那个项目的issue
  client_id: {刚才申请的ClientID}
  client_secret: {刚才申请的Client Secret}
  proxy_gateway: # Address of api proxy, See: https://github.com/aimingoo/intersect
  redirect_protocol: # Protocol of redirect_uri with force_redirect_protocol when mint enabled
```

### 开通评论
注意到这里基本上已经OK了,再看你的博客应该可以显示评论了.不过每篇博客都需要你手动初始化评论功能(如果你的历史博客很多那就一篇一篇去点吧，不过貌似有人写了批量处理脚本,没试过哈).


### 问题
- Error: Validation Failed

issue的Label有长度限制,对于中文博客来说,中文标题很容易就超过长度限制,所以需要做一下特殊处理,修改`themes/next/layout/_third-party/comments/gitment.swig`:
```
var gitment = new {{CommentsClass}}({
     id: '{{ page.date }}',
     owner: '{{ theme.gitment.github_user }}',
     repo: '{{ theme.gitment.github_repo }}',
```
主要是那个id改一下,一般而言你写博客不可能同一时间创建两份博客,所以这个一般而言是不会重的．
**NOTE:**这一需要特别强调一下缓存问题,必须清除浏览器缓存,否则会一直报`Not Found`,具体表现就获取issue的地址里面有`undefined`,这个折腾了差不多一天才搞定.

- object ProgressEvent

出现这个问题是由于作者在`gitment.browser.js`中硬编码了自己的服务器人证域名:`https://gh-oauth.imsun.net`，现在域名过期了,要么自己搭一个认证服务器，要么可以用其他人的，在gitment的issues里面找到了答案:[#175](https://github.com/imsun/gitment/issues/175),需要把Hexo你所用的主题中的引用文件给改一下，我用的是next主题,改动方法如下:
修改`themes/next/layout/_third-party/comments/gitment.swig`
```
<!-- LOCAL: You can save these files to your site and update links -->
    {% if theme.gitment.mint %}
        {% set CommentsClass = "Gitmint" %}
        <link rel="stylesheet" href="https://aimingoo.github.io/gitmint/style/default.css">
        <script src="https://aimingoo.github.io/gitmint/dist/gitmint.browser.js"></script>
    {% else %}
        {% set CommentsClass = "Gitment" %}
        <!-- <link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css"> -->
        <!-- <script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script> -->
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/theme-next/theme-next-gitment@1/default.css">
        <script src="https://cdn.jsdelivr.net/gh/theme-next/theme-next-gitment@1/gitment.browser.js"></script>
    {% endif %}
<!-- END LOCAL -->
```
注释的部分是原来的地址，后面的是引用的网友的解决方案地址,需要注意的是确保`themes/next/_config.yml`中`theme.gitment.mint`设置为false,才会走到我们改动的分支.
