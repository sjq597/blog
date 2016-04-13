title: Permission denied (publickey)
date: 2015-11-21 21:41:09
tags: [Git,Linux]
categories: Git
---
最近在家里电脑的电脑上`git pull`的时候，报错。电脑系统是`Ubuntu 14.04 64bit`，发现项目无法更新，之前都可以把项目拉取下来，现在更新却不行了，具体报错如下：
```
Permission denied (publickey). 
fatal: Could not read from remote repository. 
Please make sure you have the correct access rights 
and the repository exists.
```
网上的说法是`SSH key`的缘故，但是我之前在gitcafe上都可以clone下来，后来也没改东西，突然就没权限了。没办法，把gitcafe上的`SSH key`删掉了，又重新生成了一遍，传上去，结果还是不行。后来我重新在另一个目录下拉取项目，发现可以，于是突然想到，会不会是目录的权限问题。于是给项目的目录赋上了最高权限
```bash
# 给docment目录赋最高权限
sudo chmod 777 docment/
```
然后重新更新项目，果然好使了。

但是后来电脑又出现了一个问题，突然就又连不上了，最后折腾了好久才搞定，这次是深入搞明白了，为啥会出现这个问题了，其实，说白了就是秘钥不对，但是网上的说法都太模糊，很多人其实都把密钥上传到了网站上，一开始是可以用的，后来确不行了，所以这里面应该不是密钥没上传导致的，而是本地的密钥出了问题，下面来看看具体的问题解决步骤。
一般报错信息是：
```
Cloning into '******'...
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
　　这个时候多半是你的密钥出了问题，而且出现这种问题一般是你使用了多个代码托管账户造成的，这个比较常见，现在很多公司都会搭建自己的代码托管仓库，例如**gitlab**，然后就是大名鼎鼎的**github**,作为程序员这个账户肯定是有的，然后呢，有时候吧，由于国内你懂得一些原因，**github**访问不是很稳定，所以会选择国内的一些代码托管仓库，例如**oschina**或者**gitcafe**。这么多账户一起使用，密钥的问题就来了。再加上我在家里还有个台式机，而且有时候为了能登公司的服务器，还要使用**Window**系统，这样下来，密钥一堆，所以密钥特别乱，慢慢的就产生了这个问题，偶尔就出现了上面这个问题。
首先要看看**~/.ssh**下面有些啥:一般就是会有一个**id_rsa**,**id_rsa.pub**,**config**这写文件,简单介绍一下这三个文件都是干啥的:
**id_rsa**是私钥，同理**id_rsa.pub**就是公钥，公钥是要传到服务器上的，每次你连接代码托管服务器，都是用服务器上的公钥去校验你本地的私钥，系统默认会读取你的**～/.ssh/**目录下的私钥文件，如果读取不到，或者读取到的私钥是错的，就会出现上面的问题，即**Permission denied (publickey).**，
这就要说道第三个文件**config**的作用了,先看看他长啥样吧：
```
Host github.com
    Hostname github.com
    IdentityFile ~/.ssh/id_rsa
    User github_username 

Host gitcafe.com
    Hostname gitcafe.com
    IdentityFile ~/.ssh/id_rsa.gitcafe
    User gitcafe_username
```
看到这个文件我想你们大概应该明白了，
1. `Host`后面的名字相当于一个别名，真正的Host是Hostname的值,也即你可以使用别名来访问你托管代码的网站,
2. `IdentityFile`就是你访问这个网站是，网站上的公钥会校验你的本地私钥的文件位置，这个填错了就无法正确校验了，所以会出错，
3. `User`就是你在代码托管网站上的用户名

另外说一个调试技巧，就是当你出问题，没权限`pull/push`代码的时候，如何确定是密钥出问题了呢？可以打开调试模式看看是用哪一个私钥文件在校验
```bash
ssh -vT git@gitcafe.com

OpenSSH_6.6.1, OpenSSL 1.0.2d 9 Jul 2015
debug1: Reading configuration data /home/zhangsan/.ssh/config
debug1: /home/zhangsan/.ssh/config line 6: Applying options for gitcafe.com
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: Applying options for *
debug1: Connecting to gitcafe.com [124.42.117.218] port 22.
debug1: identity file /home/zhangsan/.ssh/id_rsa.gitcafe type 1
debug1: identity file /home/zhangsan/.ssh/id_rsa.gitcafe-cert type -1
debug1: Remote protocol version 2.0, remote software version OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.3
debug1: Host 'gitcafe.com' is known and matches the RSA host key.
debug1: Found key in /home/zhangsan/.ssh/known_hosts:1
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /home/zhangsan/.ssh/id_rsa.gitcafe
debug1: Offering RSA public key: zhang.san@163.com
debug1: Authentications that can continue: publickey
debug1: Offering RSA public key: zhang.san@gmail.com
debug1: Server accepts key: pkalg ssh-rsa blen 279
debug1: Authentication succeeded (publickey).
Authenticated to gitcafe.com ([124.42.117.218]:22).
Hi zhangsan! You've successfully authenticated, but GitCafe does not provide shell access.
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
debug1: client_input_channel_req: channel 0 rtype eow@openssh.com reply 0
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 4632, received 3656 bytes, in 1.0 seconds
Bytes per second: sent 4642.3, received 3664.1
debug1: Exit status 1
```
这里输出了很多,看第4行，告诉我们加载配置文件的地方就是前面提到的**config**文件，然后第9行，他从配置文件里加载私钥文件，所以你想看你的**git**是加载的哪个文件，看这个就可以了，如果不对，那么按照上面的配置重新配置一下就OK了。

**备注：** 注意，如果在使用`sudo git ...`的时候仍然提示**Permission denied (publickey).**。记得去掉`sudo`,或者更改对应的私钥文件的权限。
