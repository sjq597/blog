title: 'Linux 本地和服务器互传文件命令:scp'
date: 2015-10-27 18:58:07
tags: [Linux,开发工具,Web]
categories: Linux使用
---
在日常工作开发中,都是登陆跳板机然后再登陆开发机或者在线服务器,有时候需要把一个本地文件上传到服务器,例如服务器缺少某个字体,通常是本地机下载字体然后上传到服务器.也有时候需要把服务器上的某个文件拷贝到本地,例如某个日志或者结果等.这个时候`scp`命令登场了.

### scp命令使用
#### 准备条件
> 假设你的用户名是zhang.san
> 跳板机地址:redirector.machine.com
> 目标机器地址:target.machine.com
> 本机文件:/home/zhang.san/1.txt
> 目标机器文件:/home/dev/www/log/test.log

**注意:** 由于是跳板机,所以会有很多人登陆,每个人在跳板机上都有一个以自己名字命名的目录,例如`/home/zhang.san/`

#### 本地上传到服务器 
* 上传文件到跳板机

```bash
➜  ~  scp 1.txt zhang.san@redirector.machine.com:~
Enter PASSCODE:"此处可能要输入密码"
1.txt                                              100%   10     0.0KB/s   00:00    
```

* 登陆跳板机查看

```bash
ssh zhang.san@redirector.machine.com
Enter PASSCODE:"输入密码处"
Last login: Tue Oct 27 19:16:47 2015 from 10.86.108.97
[zhang.san@redirector.machine.com ~]$ ls
1.txt
```
 再在跳板机上进行同样的操作上传到服务器即可.

#### 服务器下载到本地
在本地机器上执行下面命令
```bash
scp zhang.san@redirector.machine.com:1.txt ./ 
```
注意这个命令不是在跳板机,而是在本地机器上执行.

其实前面的是比较复杂的，如果单纯的只是想从服务器上上传东西，可以使用一个简单的命令：
```python
python -m SimpleHTTPServer 端口号
```
这个命令可以把执行这个命令的目录变成一个服务器的目录，可以直接用http下载,pythons服务器上一般是安装了的，没有的话就不能这么用了。
