title: Linux下Node.js安装
date: 2015-09-29 23:29:33
tags: [Linux,Web,Node.js,前端,开发环境]
categories: [博客搭建]
----
### 首先去官网下载Node.js
我的是`Ubuntu 14.04`,所以下载的是node-v4.1.1-linux-x64.tar.gz

#### 解压缩：
```
tar -xvf node-v4.1.1-linux-x64.tar.gz -C /usr/qunar/
cd /usr/qunar
sudo mv node-v4.1.1-linux-x64 node
export PATH=/usr/qunar/node/bin/:$PATH
node -v
```

**NOTE:** 记住`$PATH`前面是冒号，不是分号，之前搞错了，结果环境变量都被搞坏了，害得我又得去重新恢复环境变量，恢复方式：

```
export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
export JAVA_HOME=/usr/java/jdk1.7.0_40
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

#### 测试运行：
```
#--------------------测试----------------------------
#创建nodejs项目目录
mkdir -p /usr/qunar/nodejs/
#创建hello.js文件
vi /usr/qunar/nodejs/hello.js
#内容如下：
var http = require("http");
http.createServer(function(request, response) {	
    response.writeHead(200, {		
        "Content-Type" : "text/plain" // 输出类型	
    });	
response.write("Hello World");// 页面输出	
response.end();}).listen(8100); // 监听端口号
console.log("nodejs start listen 8100 port!");
#后台运行
node /usr/local/nodejs/hello.js &
#浏览器访问
http://localhost:8100/
```
如果看到浏览器界面输出Hello World!就表示成功了。
