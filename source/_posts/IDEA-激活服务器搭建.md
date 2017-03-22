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

有了这个还不够,有时候vps访问并不是那么流畅，或者没联网，这可能会影响我们的idea启动速度,所以可以把激活服务器搭建在本机，然后设置开机启动即可,假设你的文件放在`/workspace/dev/IntelliJIDEALicenseServer_linux_amd64`,那么可以修改`/etc/rc.local`,添加:
```
sudo /workspace/dev/IntelliJIDEALicenseServer_linux_amd64 &
```
然后激活服务器可以填:
```
http://127.0.0.1:1017
```
然后就可以每次不用管是不是联网了。
