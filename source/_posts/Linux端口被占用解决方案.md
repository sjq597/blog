title: Linux端口被占用解决方案
date: 2016-06-26 10:52:53
tags: [Linux, 开发环境]
categories: Linux使用
---
之前一直使用ss代理上google，但是最近不知道怎么回事儿，本地运行
```
sslocal -c /etc/config.json
```
报错:
```
INFO: loading config from doc/config.json
2016-06-26 12:34:53 INFO     loading libcrypto from libcrypto.so.1.0.0
2016-06-26 12:34:53 INFO     starting local at 127.0.0.1:1080
2016-06-26 12:34:53 ERROR    [Errno 98] Address already in use
```
主要是找不到占用这个端口的进程，也不知道怎么把占用端口的进程杀掉,网上查了相关解决方法，总算找到解决方案:
首先得找到是哪个进程占用了这个端口,即找出进程的pid:
```
➜  ~ netstat -anp | grep 1080             
（并非所有进程都能被检测到，所有非本用户的进程信息将不会显示，如果想看到所有信息，则必须切换到 root 用户）
tcp        0      0 0.0.0.0:1080            0.0.0.0:*               LISTEN      1955/EmbedThunderMa
```
看到没，那个`1955`就是罪魁祸首,于是再通过pid查看具体是哪个进程:
```
➜  ~ ps -ef | grep 1955
anonymo+  1955  1871  0 09:47 ?        00:00:18 /opt/xware-desktop/xware/lib/EmbedThunderManager *********
anonymo+  5621  2847  0 10:41 pts/0    00:00:00 grep --color=auto 1955
```
终于找到原因,原来是前一阵子为了下东西，装了一个Linux版的迅雷，虽然最后还是没法下载，但是这东西有个服务开机自动启动，占用了我的1080端口，导致我想用没法用
果断卸载了坑爹的迅雷,然后把这个进程杀了
```
sudo kill -9 1955
```
然后果然1080端口可以用了。


