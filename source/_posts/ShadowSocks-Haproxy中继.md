title: ShadowSocks+Haproxy中继
date: 2018-05-22 20:24:11
tags: [shadowsocks,haproxy]
categories: Linux使用
---
最近发现家里的网连我国外的服务器非常的慢,丢包已经基本上到了访问不了的地步了,所以搭的ss代理基本上只能在公司用.之前花几百快撸的腾讯云服务器放了好几个月都没管过了,正好拿来做中继代理.之前买了半年的阿里云服务器搞过，后来过期了没有续费了,这次撸了6年的应该够用了.

# 准备工作
其实就是运营商的国际出口比较堵,所以直接访问国外的服务器丢包严重，当然实测应该还有一个原因，应该是运营商主动干扰这些代理服务器.个人感觉后面的可能性要大一些,不过不管是哪种都不重要了,要实现低延迟链路，只能借助大厂的线路(阿里，腾讯),基本原理也很简单:
* 直连方案:
```
local(A) --> sserver(C)
```
* 中继方案:
```
local(A) --> aliyun/cloud tencent(B) --> sserver(C)
```

# 安装配置
主要是在中继机器B上操作,sserver(C)不用做任何变更,首先登录我们的中继机器(B):
```
sudo yum install haproxy -y
```
然后就是配置我们的机器了,编辑`sudo vim /etc/haproxy/haproxy.cfg`,直接改成下面这样:
```
global
    ulimit-n  51200

defaults
    log     global
    mode    tcp
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

frontend ss-in
    bind *: {B_local_listen_port}
    default_backend ss-out

backend ss-out
    server sserver_name {sserver_ip}:{sserver_port} maxconn 20480
```
这里需要设置的几个地方为:
```
{B_local_listen_port}:这个就是中继服务器监听外部请求的端口,为了你local改动小,这个端口可以设置和sserver(C)的端口一致
{sserver_ip}:这个就很好理解了,就是你部署sserver的机器的ip,后面那个端口也是ss服务器的端口
```
等这些都改完了，然后还需要做一件事,就是给你的中继服务器开安全组策略，阿里和腾讯的云主机基本上是一样的，开入站和出战规则即可,然后把设置好的安全组规则绑定到你的机器实例上.简单来说就是需要对`{B_local_listen_port}`开入站规则,对`{sserver_port}`开出战规则，如果你分不清觉得麻烦，就把这两个端口设置的一样,然后开一个端口的出站和入站规则即可.

# 服务启用
由于你改了安全组规则，所以需要重启你的服务器实例,重启好了之后,需要启动haproxy服务,不同的机器不一样，以我的腾讯云主机为例:
```
systemctl start haproxy.service
```
启动服务之后可以检测一下服务是否开启了,直接用`ps`命令即可.

然后就是本地`local`机器,即`sslocal`的配置,因为我们上面配置的中继监听端口和`sserver`的端口一致,所以我们的`sslocal`只用改一个ip，端口不用改,ip由原来的`sserver ip`改为中继服务器的ip就行了.延迟由原来的300ms直接降为40ms,收工.
