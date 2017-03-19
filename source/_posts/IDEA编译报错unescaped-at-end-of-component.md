title: IDEA编译报错unescaped  at end of component
date: 2017-03-17 10:42:00
tags: [Linux,IDEA,Java,Tomcat]
categories: Java笔记
---
刚接手个项目，从代码仓库里面直接clone下来,在idea里面运行的时候报错，但是直接在命令行用`maven`打包又没啥问题。所以我猜想这个错误应该只和idea这个ide有关,网上查了一圈发现并没有什么收获,后来参考一个正确的项目的结构配置把问题解决了,在这里记录一下.

问题特征就是几乎每一个类都会报标题那个错误，而且打开几个java文件,发现一片红,但是idea里面的以来显示jar包都导入了,但是就是无法识别,引入不了,直接打包没啥问题，但是想在Idea里面通过Tomcat里面来启动就会报错,最后也是查了很多资料没解决，在同事的帮助下解决了这个问题。

主要分两步

* 项目结构不对

每一个Spring工程必须在Idea里面配置正确，不然有些路径的文件或者包可能读取不到，具体的看`Project Structure`里面的配置,可以找个正确的项目可以运行的参考参考,需要注意的地方是:
```
Modules: 这个地方的模块名和一些Source Roots还有web resource directories`看看设置的有没有问题，还有Spring的配置，主要是路径

* 打包问题

在idea里面打包有两个方式，一个是选war包，另一种是选exploded,这个要在`Edit Configguration`那看,记得要选第二种方式，即exploded,这个方式和第一个方式有啥不同:
```
# war
需要把这个包拷贝到tomcat服务器上，然后解压
# exploded
不需要拷贝解压，tomcat会直接读取我们项目的target里面的classes文件，做到调试可更新


