title: IDEA 激活服务器搭建
date: 2017-03-19 12:45:07
tags: [Linux]
categories: 开发环境
---
记一下,老是忘记了，之前的激活服务器搭建在公司的机器上，所以必须得等vpn才能正常启动，打算在自己的vps上搭建一个，记录一下:

首先下载激活软件,这放一个下载地址[百度盘IDEA激活服务器下载地址](https://pan.baidu.com/s/1c2pFhIS),密码:uh36.
然后把这个下载的软件上传到你的vps上就行了:
```
scp IntelliJIDEALicenseServer_linux_amd64 your_name@you_vps_id:~
```
然后登录到你的vps上，找到当前目录，执行:
```
chmod +x IntelliJIDEALicenseServer_linux_amd64
./IntelliJIDEALicenseServer_linux_amd64 &
```
然后在你的IDEA激活那选`License Server`,地址填:
```
http://<your_vps_id>:1017
```
