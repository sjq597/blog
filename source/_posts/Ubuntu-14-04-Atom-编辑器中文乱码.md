title: Ubuntu 14.04 Atom 编辑器中文乱码
date: 2015-12-17 22:48:24
tags: Linux
categories: 开发环境
---
最近尝鲜试了一把`Atom`编辑器，发现在Ubuntu上这个编辑器果然还是刚出来，不是很成熟，中文显示不了，换了中文编码`GBK`，`GB18030`也不行。网上很多人说换成文泉驿字体就可以了，但是我试了发现，还是乱码。很多人都是抄那一种方法，都没有去试试到底行不行，其实方法没有错，只是不是所有的人都可以这么解决乱码，正确的步骤应该是：
1. 从菜单中打开`Edit->Open your config`选项，或者`setting views`中的`font-family`选项，把字体设置成你机器上有的中文字体

2. 怎么看自己机器上的中文字体，你可以在终端输入：
```bash
fc-list :lang=zh
/usr/share/fonts/truetype/arphic/uming.ttc: AR PL UMing TW MBE:style=Light
/usr/share/fonts/truetype/arphic/ukai.ttc: AR PL UKai CN:style=Book
/usr/share/fonts/truetype/arphic/ukai.ttc: AR PL UKai HK:style=Book
/usr/share/fonts/simsun.ttf: SimSun,宋体:style=Regular
/usr/share/fonts/truetype/arphic/ukai.ttc: AR PL UKai TW:style=Book
/usr/share/fonts/truetype/droid/DroidSansFallbackFull.ttf: Droid Sans Fallback:style=Regular
/usr/share/fonts/truetype/arphic/ukai.ttc: AR PL UKai TW MBE:style=Book
/usr/share/fonts/truetype/arphic/uming.ttc: AR PL UMing TW:style=Light
/usr/share/fonts/truetype/arphic/uming.ttc: AR PL UMing CN:style=Light
/usr/share/fonts/truetype/arphic/uming.ttc: AR PL UMing HK:style=Light
```
 之前由于`IDEA`这款`IDE`有乱码我下载了`SimSun`字体，所以我就没有按照网上的去设置什么文泉驿字体了，我压根就没有，再设置也是乱码。

3. 最终配置如下：
```
editor:

invisibles:
  {}
  tabLength: 4
  fontFamily: "SimSun"
```
