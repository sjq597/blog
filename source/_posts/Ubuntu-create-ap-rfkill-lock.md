title: Ubuntu create_ap rfkill lock
date: 2016-02-20 16:14:59
tags: [Linux]
categories: Linux使用
---
在[Ubuntu 14.04设置wifi热点](../../../2015/10/07)这篇文章中我们使用create_ap来在Ubuntu上建立wifi,主要是看中了它可以建立隐藏wifi的功能，但是在使用的时候老是出问题，例如
> 
WARN: Your adapter does not fully support AP virtual interface, enabling --no-virt
Config dir: /tmp/create_ap.wlan0.conf.A912iinp
PID: 20324
command failed: Operation not supported (-95)
Access Point's SSID is hidden!
RTNETLINK answers: Operation not possible due to RF-kill

解决办法：
```bash
$ sudo rfkill list all
0: hci0: Bluetooth
        Soft blocked: no
        Hard blocked: no
1: dell-wifi: Wireless LAN
        Soft blocked: yes
        Hard blocked: no
2: dell-bluetooth: Bluetooth
        Soft blocked: no
        Hard blocked: no
3: phy0: Wireless LAN
        Soft blocked: yes
        Hard blocked: no
```
注意看1和3的`Soft blocked`那里都是yes,在终端里输入：
```bash
$ sudo rfkill unblock wifi
$ sudo rfkill unblock all
```
然后再看看
```bash
$ sudo rfkill list
0: hci0: Bluetooth
        Soft blocked: no
        Hard blocked: no
1: dell-wifi: Wireless LAN
        Soft blocked: no
        Hard blocked: no
2: dell-bluetooth: Bluetooth
        Soft blocked: no
        Hard blocked: no
3: phy0: Wireless LAN
        Soft blocked: no
        Hard blocked: no
```
yes那里变成no了，然后就可以再次使用create_ap来建立wifi了。引用子stackoverflow上网友的原话：
> 
Soft-blocking
The output to sudo rfkill list shows that your network card is "soft-blocked".
This could happen when the wireless card has been signalled to switch-off via the kernel.
Try the following steps:
run in a terminal:
sudo rfkill unblock wifi; sudo rfkill unblock all
rerun sudo rfkill list to confirm that the card has been unblocked.
reboot
rerun sudo rfkill list again to confirm unblocking as been retained.
rerun sudo lshw -class network - you should now see that the kernel has recognised (or not) the wireless card.
If the wireless kernel module has been recognised (it should not say "unclaimed"), Network Manager should now be able to see wireless networks that are available in your vacinity.

