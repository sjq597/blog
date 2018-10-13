title: Ubuntu 16.04 aria2 百度盘下载
date: 2017-03-11 18:56:48
tags: [Linux,Shell]
categories: Linux使用
---
由于本人经常在我的vps上下载东西上传到百度云,然后再下载到本地机器,平时的工作环境又是Ubuntu,官方没有提供百度盘的Linux版本,就算有如果不是非会员也很麻烦,限速很蛋疼，最快也就下300多k的速度.网上查了一下,虽然百度盘限速，但是不限下载线程,也就是说其实用浏览器去下载,百度官方限制你的单个http请求的速度,破解方法就是开多个线程,一般10个就够了,带宽够大可以再高点，只要机器扛得住。

### 安装步骤
需要安装一个软件以及一个chrome的插件

#### aria2
这个是Linux下面一个非常优秀的多线程下载工具,安装方法:
* 安装aria2

```
sudo apt-get install aria2
```

* 安装BaiduExporter

装了这个之后需要配合一个浏览器插件,因为百度盘的文件链接是不能直接下载的，需要安装下Chrome插件[BaiduExporter](https://github.com/acgotaku/BaiduExporter/blob/master/chrome.crx),怎么安装Chrome插件就不细讲了，不懂的直接度娘。
**备注:** 如果是Mac用户可以参考这篇文章[Aria2 - 可能是现在下载百度云资料速度最快的方式](http://www.jianshu.com/p/e5e56a1d25a3),由于主要讲Ubuntu下面的配置,这个就不多讲了.

* 下载文件

安装好`aria2`以及`BaiduExporter`之后,我们以一个百度云里面的文件来做个测试:
![下载文件链接](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Ubuntu%2016.04%20aria2%20%E7%99%BE%E5%BA%A6%E7%9B%98%E4%B8%8B%E8%BD%BD01.png)
注意是勾选所需要下载的文件,然后会多出一个**导出下载**标签页,然后我们需要点列表中的`导出下载`,然后就会弹出一个框:
![aria2下载链接框](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/Ubuntu%2016.04%20aria2%20%E7%99%BE%E5%BA%A6%E7%9B%98%E4%B8%8B%E8%BD%BD02.png)
把框里面的内容全部复制出来,直接丢到终端中执行，注意文件会保存到当前路径,如果你想保存到指定位置，还是先cd到指定目录吧.
下载的命令如下:
```
aria2c -c -s10 -k1M -x16 --enable-rpc=false -o 'Gantz.O.2016.720p.WEB-DL.AAC2.0.H.264 [MultiSub(Eng,Ita,Por,Rum,Swe,Tur)]v2.mkv' --header "User-Agent: netdisk;5.3.4.5;PC;PC-Windows;5.1.2600;WindowsBaiduYunGuanJia" --header "Referer: http://pan.baidu.com/disk/home" --header "Cookie: BDUSS=FDVnRtbnVaclhoR2dMLWdScHY2TlNFN1h3b3Qyb0stcXgtcXpyS0gtSFpXT3RZSVFBQUFBJCQAAAAAAAAAAAEAAAA7KQQU0KHHv7Tz0KG08wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAANnLw1jZy8NYcD; pcsett=1489315049-753101e6547a834c2756fc96703e7839" "https://pcs.baidu.com/rest/2.0/pcs/file?method=download&app_id=250528&path=%2Fapps%2Fbypy%2FGantz.O.2016.720p.WEB-DL.AAC2.0.H.264%20%5BMultiSub(Eng%2CIta%2CPor%2CRum%2CSwe%2CTur)%5Dv2.mkv"
```
看看下载结果:
```
[#99b455 2.2GiB/2.2GiB(99%) CN:10 DL:148KiB ETA:1 *** Download Progress Summary as of Sat Mar 11 19:12:58 2017 *** 
================================================
[#99b455 2.2GiB/2.2GiB(99%) CN:10 DL:92KiB ETA:48s]
FILE: /home/anonymous/Gantz.O.2016.720p.WEB-DL.AAC2.0.H.264 [MultiSub(Eng,Ita,Por,Rum,Swe,Tur)]v2.mkv
------------------------------------------------

[#99b455 2.2GiB/2.2GiB(99%) CN:1 DL:19KiB ETA:13
03/11 19:13:57 [NOTICE] Download complete: /home/anonymous/Gantz.O.2016.720p.WEB-DL.AAC2.0.H.264 [MultiSub(Eng,Ita,Por,Rum,Swe,Tur)]v2.mkv

Download Results:
gid   |stat|avg speed  |path/URI
======+====+===========+=======================================================
99b455|OK  |   1.2MiB/s|/home/anonymous/Gantz.O.2016.720p.WEB-DL.AAC2.0.H.264 [MultiSub(Eng,Ita,Por,Rum,Swe,Tur)]v2.mkv

Status Legend:
(OK):download completed.
```
**备注:**简单说明一下命令行里面的命令`s10`指下载的线程数为10.开始速度均值为`2Mb/s`,后面快完成的时候会慢下来，均速大概`1.2Mb/s`.
