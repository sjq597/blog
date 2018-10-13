title: Ubuntu 14.04使用shadowsocks 科学上网
date: 2015-10-06 18:22:53
tags: [shadowsocks,科学上网]
categories: [杂七杂八]
---
`shadowsocks`是目前本人和小伙伴们使用最多的一种，主要是因为`GoAgent`作者被请去喝茶了。准备工作，需要有配置好的`shadowsocks`服务端，这个去搜吧，你如果连一个`shadowsocks`账号也没有，那也没用，现在假定你有个
`shadowsocks`的服务器账号可以用来作代理。
看官方提供的安装`shadowsocks`方法
>sudo apt-get install python-pip
pip install shadowsocks

你会发现貌似装不上，确实，由于`shadowsocks`传播的太广，作者也被请去喝茶了，所以`github`上的原项目也被删了，相关服务器也没了，要装`shadowsocks`只能像下面这样，首先，你要有项目的源码，地址我的`github`上备份了一份：[shadowsocks github地址](git@github.com:sjq597/shadowsocks.git)
使用`git clone`下来即可，目前最新的就是`2.6.1`。

接下来，你需要将项目的代码打包安装，命令如下：
>sudo python setup.py install

![shadowsocks 目录结构](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu-14-04使用shadowsocks-科学上网01.png)
然后编辑`/etc/shadowsocks.json`。没有则创建一个这个文件。，编辑内容如下：
```json
{
    "server":"your server ip address",
    "server_port":server port,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"your account password",
    "timeout":600,
    "method":"rc4-md5",
    "workers":1
}
```

配置照着改，把你的`shadowsocks`的账号的地址和端口要填对，加密方式看你的账号是啥方式。

好了，安装成功并且编辑成功之后，你现在只需要启动终端中的服务进行端口监听就行(没有图形界面还省内存)。
>sslocal -c /etc/shadowsocks.json

成功启动的界面就是下面这样的：
![启动代理服务](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu-14-04使用shadowsocks-科学上网02.png)

然后你需要在浏览器里设置代理，注意`shadowsocks`是`socks 5`代理，所以和`vpn`还不太一样，默认不是全局代理的，也就是说你的应用
没有办法翻墙，但是浏览器可以。

3、设置浏览器代理，以`Chrome`为例，火狐也是一样。

由于`Chrome`的商店被墙了，上不了，你需要手动下载`SwitchySharp`，[百度盘地址](http://pan.baidu.com/s/1kTivhMf)。
手动安装过程：在`Chrome`浏览器地址栏输入
>chrome://extensions/ 

然后把插件拖到浏览器，安装即可。

安装好之后需要进行规则的设置，在工具栏点插件图标，选择`options`，如图`Proxy Profiles`：
![Proxy Profiles界面](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu-14-04使用shadowsocks-科学上网03.png)
注意`SOCKS Host`地址和端口设置分别为`127.0.0.1`和`1080`。特别注意要选`SOCKS v5`。不代理的地址可以加在下面`No Proxy for`里面。

还需要设置切换规则，也就是哪些特定的地址用代理，哪些不用，这样有个好处，比如你点开国内的不用代理的网站更快，而且有的号是需要流量的，这样还可以省流量,`Switch Rules`：
![Switch Rules界面](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu-14-04使用shadowsocks-科学上网04.png)
上面那些地址就是需要代理的，注意选你能用的代理，我这个插件由于开始使用了`goagent`，导入了`goagent`的配置文件，所以有三个代理。
但是正是`goagent`的`ip`都不好用，而且人多就卡，非常不稳定，我才决定折腾一下`shadowsocks`。代理规则可以自己填或者从配置文件导入.具体的配置文件[百度盘连接](http://pan.baidu.com/s/1jGpOLvg)
用法就是在插件的`Import/Export`选项里，`Switch Rules`里`Export Rules List`即可。
![导入代理规则](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu-14-04使用shadowsocks-科学上网05.png)
手机上的教程就不多讲了，只讲我的`android`手机配置吧。具体可以取网上搜，也是下一个手机的客户端，在配置文件里配置好你的
手机客户端，我在`google play`下载的，放心，绝对安全，[百度盘地址](http://pan.baidu.com/s/1ntF1drF)
安装之后，点左上角，添加配置文件，配置文件界面如下：
![安卓代理设置](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu-14-04使用shadowsocks-科学上网06.png)
把你的`shadowsocks`服务器地址，端口以及密码和加密算法填完就`ok`。然后点右上角就可以打开了。
