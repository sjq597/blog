title: Ubuntu 14.04 使用VPN
date: 2015-10-25 20:10:03
tags: [Linux,开发工具]
categories: 开发环境
---
Linux下使用VPN十分简单，只需要安装一软件即可。

* 安装网络挂件`vpnc`

```bash
sudo apt-get install vpnc
```

* 找到公司的默认配置文件`qunar_common.pcf`,使用命令将其转化成对应的配置文件

```bash
sudo pcf2vpnc Documents/qunar_common.pcf default.conf
```

* 替换配置文件

```bash
sudo chmod 777 /etc/vpnc 
sudo cp default.conf /etc/vpnc
```

* 更改配置文件中的账户Xauth username

```bash
sudo vim /etc/vpnc/default.conf
```

* 连接VPN

```bash
sudo vpnc-connect
```
**备注:** 关闭连接用`sudo vpnc-disconnect`。
