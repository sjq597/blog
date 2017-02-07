title: CentOS matplotlib freetype png
date: 2017-02-06 16:12:53
tags: [Linux, Python]
categories: Linux使用
---
服务器上用virtualenv部署安装matplotlib，始终无法安装，错误日志如下:
```
freetype: no  [The C/C++ header for freetype2 (ft2build.h)

          could not be found.  You may need to install the

          development package.]

     png: no  [pkg-config information for 'libpng' could not

          be found.]

   qhull: yes [pkg-config information for 'qhull' could not be

          found. Using local copy.]
```
安装缺失依赖:
```
sudo yum install freetype
```
之后重新安装还是不行,彻底解决方案为:
```
sudo yum -y install freetype freetype-devel libpng-devel
```
