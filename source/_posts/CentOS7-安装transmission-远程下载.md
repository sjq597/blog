title: CentOS7 安装transmission 远程下载
date: 2019-12-30 23:41:56
tags: [Linux]
categories: 开发环境
---

买的VPS三年了，一不小心忘了续费过期了，数据环境全给清空了，又得重新配置环境。因为有ipv6地址，所以想把上学那会儿的PT站用起来，重新装下`transmission`,这个装好了可以直接从网页上添加任务挂种，比较方便，安装过程如下 

### 安装Transmission

```
# 安装源
yum install epel-release
# 直接安装相关的包
yum install transmission-*
# 启动服务创建配置文件
service transmission-daemon start
# 停止文件修改配置文件
service transmission-daemon stop
```
修改配置文件`/var/lib/transmission/.config/transmission-daemon/settings.json`,需要改动的几个地方如下:
```
"download-dir": "/root/pt",
"incomplete-dir": "/root/pt",
"rpc-authentication-required": true,
"rpc-port": 12345,
"rpc-enabled": true,
"rpc-password": "the fuck password",
"rpc-username": "zhangsan",
"rpc-whitelist": "0.0.0.0",
"rpc-whitelist-enabled": false,
```
**NOTE:** 下载的目录文件夹你得手动创建一下，我这里直接就在`pt`文件夹下面了,端口最好也改下，不要用默认的。

然后重启服务
```
systemctl start transmission-daemon.service
```

最后网页验证一下,访问`http://123.4.5.6:12345/transmission/web/`,如果可以访问那就说明可以了。

### 百度云盘
其实用PT下载挺快的，一般可以达到30Mb/s这种速度，但是怎么把文件搞回来是个问题，直接在服务器上起一个python的httpServer也可以，但是直连非常的慢，还不稳定，所以我采取了一种比较简单的方案：把下载下来的文件同步到百度云盘，然后再通过百度云下载。
