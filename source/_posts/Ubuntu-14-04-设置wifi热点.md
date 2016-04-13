title: Ubuntu 14.04 设置wifi热点
date: 2015-10-07 23:35:49
tags: [Linux]
categories: Linux使用
---
查了一下资料主要有两种方法:
### 第一种:network manager设置
这种方法配置比较复杂.并且不能给`Android`手机和`iPhone`共享.果断放弃(其实我是尝试过的,真的没有用).

### 第二种:使用ap-hotspot,亲测可用
这种方法配置简单,用起来也方便。

#### 第一步:安装ap-hotspot
```bash
$ sudo add-apt-repository ppa:nilarimogard/webupd8
$ sudo apt-get update
$ sudo apt-get install hostapd
$ sudo apt-get install ap-hotspot
```

####  第二步:配置ap-hotspot
```bash
$ sudo ap-hotspot configure
Detecting configuration...
Detected eth0 as the network interface connected to the Internet. Press ENTER if this is correct or enter the desired interface below (e.g.- eth0, ppp0 etc.):
# 确认eth0作为连接网络的网卡(有多个有线网卡可能不同，自己选择)，一般就默认回车确认选择
Detected wlan0 as your WiFi interface. Press ENTER if this is correct or enter the desired interface (e.g.- wlan1):
# 确认wlan0作为发射wifi信号的网卡，一般就默认回车确认选择
Enter the desired Access Point name or press ENTER to use the default one (myhotspot):
# 输入你想建的wifi的名字
Enter the desired WPA Passphrase below or press ENTER to use the default one (qwerty0987):
#输入wifi的密码
```

#### 第三步:启动wifi
```bash
$ sudo ap-hotspot start
Starting Wireless Hotspot...
Wireless Hotspot active
```
正常情况下，`Ubuntu 14.04`是会一直在`Starting Wireless Hotspot...`状态的，即新建`wifi`失败，失败解决方案有下面两种，第一个不行换第二个。

### 第一次配置无法启动解决方案
* 无法出现`Wireless Hotspot active`，并一直保持`Starting Wireless Hotspot...`

`hostapd`默认版本有bug
解决方法：移除hostapd:
```bash
sudo apt-get remove hostapd
```

然后
- 64 bit

```bash
cd /tmp wget http://archive.ubuntu.com/ubuntu/pool/universe/w/wpa/hostapd_2.1-0ubuntu1.3_amd64.deb
sudo dpkg -i hostapd*.deb 
sudo apt-mark hold hostapd
```
- 32bit：

```bash
cd /tmp wget http://archive.ubuntu.com/ubuntu/pool/universe/w/wpa/hostapd_2.1-0ubuntu1.3_i386.deb
sudo dpkg -i hostapd*.deb 
sudo apt-mark hold hostapd
```
 之后需要重新安装ap-hotspot.
如果下载包一直404 not found就去找这里找[2.x的deb包下载地址](http://archive.ubuntu.com/ubuntu/pool/universe/w/wpa/)

* 下载了2.x的包还是不行，直接去Ubuntu官网[Ubuntu官网1.0版本下载](http://old-releases.ubuntu.com/ubuntu/pool/universe/w/wpa/hostapd_1.0-3ubuntu2.1_amd64.deb)

* 重新安装第二次启动出现问题

```bash
出现如下问题：
$ sudo ap-hotspot start
Another process is already running
$ sudo ap-hotspot stop
Wireless Hotspot is not active
```
解决办法：`sudo rm /tmp/hotspot.pid`

**NOTE:**另外附上几个常用的命令选项:
```bash
Usage:    ap-hotspot [argument]

    start          start wireless hotspot        // 打开wifi
    stop           stop wireless hotspot         // 停止wifi
    restart        restart wireless hotspot      // 重启wifi
    configure      configure hotspot             // 配置wifi
    debug          start with detailed messages  // dubug模式打开wifi,会显示一些详细的信息
```

### 隐藏SSID
上面讲了这么多,比较复杂,而且这个建的热点还不能隐藏,下面介绍一个比较简单的另一个工具,`create_ap`项目的github地址:[oblique/create_ap](https://github.com/oblique/create_ap#start-service-immediately)
比较强大,主要是看中了他无线可以隐藏的功能,因为公司不让私自建wifi.

#### 安装create_ap
```bash
git clone https://github.com/oblique/create_ap
cd create_ap
make install
```

#### create_ap使用

1. 无密码
```bash
create_ap wlan0 eth0 MyAccessPoint
```
2. WPA + WPA2加密
```bash
create_ap wlan0 eth0 MyAccessPoint MyPassPhrase
```
3. 创建隐藏wifi
```bash
create_ap --hidden wlan0 eth0 MyAccessPoint MyPassPhrase
```
