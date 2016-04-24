title: SSH 免密码登陆
date: 2016-04-24 11:07:12
tags: Linux
categories: Linux使用
---
开发工作中，经常需要使用ssh来登录服务器，密码一般都是机器生成的，特别难记，关键是机器多了这样也麻烦，所以配置一下ssh面密码登录，节省时间。

### 系统说明
本地机器：Ubuntu
服务器：CentOS

### 大概流程
#### 本地机器配置
1. 通过`ssh-keygen`产生RSA公私密钥对

```
$ ssh-keygen`
```
然后一路回车，不要输入任何密码和字符，最后在`~/.ssh/`文件夹中会生成两个文件`id_rsa`和`id_rsa.pub`两个文件，然后需要修改问年间权限
```
$ chmod 775 ~/.ssh
```
将`id_rsa.pub`上传到服务器的`~/.ssh/`文件夹下，其实也不用上传，直接把这个文件的内容复制然后在服务器的对应文件夹下创建文件，然后写入也是一样的。
2. 在`~/.ssh/`文件夹里创建配置文件`config`
```
Host my_server // 服务器别名
    HostName 192.168.1.120  // 服务器ip
    User root   //登录用户名
    Port 22     // ssh端口
```

#### 服务器机器配置
1. 修改`/etc/ssh/sshd_config`文件,将下面几行前面的注释去掉
```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      %h/.ssh/authorized_keys
```
如果已经注释掉了就不用配置则这里了，
2. 在用户目录下创建`.ssh`文件夹，如果有就不用创建了，具体路径为`~/.ssh/`
然后在`~/.ssh/`文件夹下面创建`authorized_keys`文件,并将之前上传到服务器上的`id_rsa.pub`文件里的内容拷贝到`authorized_keys`中,保存之后重启ssh服务

```
$ cd ~/.ssh/
$ sudo cat id_rsa.pub > authorized_keys
$ sudo chmod 644 authorized_keys
$ sudo service sshd restart
```

####　本机测试
通过终端连接服务器
```
$ ssh my_server
```
**注意:**如果这一步出现`bad owers`等错误，记得要修改本机的文件权限
```
$ chmod 700 ~/.ssh
$ chmod go+rwx ~/.ssh/*
```