title: Hexo使用next.Pisces主题
date: 2018-05-15 22:17:45
tags: [Hexo]
categories: 博客搭建
---
之前Hexo博客一直是使用的[Jacman](https://github.com/wuchong/jacman)主题,用久的有点儿审美疲劳，最近看上比较简洁的主题[Next](https://github.com/theme-next/hexo-theme-next),视觉上确实要好看很多,配色简洁看着比较舒服.

# 安装
首先进入到你的博客的根目录
```
➜  Blog git:(master) ✗ 
➜  Blog git:(master) ✗ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

# 配置
接下来就是把主题配置成`Next`,修改博客根目录下的配置文件`_config.yml`:
```
# Extensions
## Plugins: http://hexo.io/plugins/
## Themes: http://hexo.io/themes/
theme: next 
```
然后启动博客就可以看到效果了.

# 修改next样式
不过默认的博客样式不是很好,一些标签页展示的不是很好,所以还需要改下样式.进入`themes/next`文件夹,修改`_config.yml`

## 修改导航菜单

```
menu:
  home: / || home
  archives: /archives/ || archive
  categories: /categories/ || th
  tags: /tags/ || tags
  about: /about/ || user
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```
修改了这几个之后还不够,需要创建这几个的导航页面(home导航页为根目录不需要创建):

### 创建归档页面
创建页面
```
hexo new page categories
```
修改内容:
```
title: 分类
date: 2018-05-14 23:34:12
type: "categories"
---
```

### 创建标签页
创建标签页
```
hexo new page tags 
```
修改内容:
```
title: 标签
date: 2018-05-14 23:36:18
type: "tags"
---
```

### 个人主页
创建个人主页
```
hexo new page about
```
修改内容:
```
title: 个人简介
date: 2018-05-14 23:38:55
---
```
### 切换主题布局
可以看到`_config.yaml`文件中默认是使用的`Muse`主题,这个主题是把标签之类的放到顶部,我更喜欢双栏布局，所以把对应部分改成下面这样:
```
# Schemes
#scheme: Muse
#scheme: Mist
scheme: Pisces
#scheme: Gemini
```

## 设置语言
默认语言在`~/Blog/next/languages`下面:
```
➜  next git:(master) ✗ ls -l languages/zh-*
-rw-rw-r-- 1 anonymous anonymous 2100 5月  15 10:47 languages/zh-CN.yml
-rw-rw-r-- 1 anonymous anonymous 2094 5月  15 10:47 languages/zh-HK.yml
-rw-rw-r-- 1 anonymous anonymous 2094 5月  15 10:47 languages/zh-TW.yml
```
默认`zh-CN.yml`就已经给我们映射好了，只需要把博客设置成对应的语言,修改`~/Blog/_config.yml`:
```
# Site
language: zh-CN
timezone: Asia/Chongqing
```
