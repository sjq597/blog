title: IDEA中配置MySQL
date: 2015-10-06 12:58:04
tags: [IDEA,开发环境,开发工具,SQL]
categories: [开发环境]
---
### 首先在终端中安装
```bash
sudo apt-get install mysql-server
```

安装中途会要求输入密码，直接输入就可以了。
等待安装完成就可以登录了
```bash
~$ mysql -uroot -p+yourpassword
```
**NOTE:** 假设你的密码是`123456`，那么最后就是`-p123456`。

### 在IDEA中配置MySQL
在`View->Tool Windows->DataBase`，打开数据库配置窗口
![数据库配置窗口](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_IDE和MySQL环境配置01.png)
点`+`号，`Data Source`选择`MySQL`配置添加，填上自己的用户名和密码，注意第一次添加需要下载驱动
![数据库连接配置](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_IDE和MySQL环境配置02.png)即`Driver files`里会提示你下载。
在`0 tables 0 procedures下`面有红色的提示，点击`download`就可以下载了，下载好了，点`Test Connection`
![测试数据库连接](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_IDE和MySQL环境配置03.png)
出现这个就说明成功了，记得上面没有配置`Database`。
![数据库控制台](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_IDE和MySQL环境配置04.png)
最终效果就是这样，可以直接写`SQL`语句，然后执行。
