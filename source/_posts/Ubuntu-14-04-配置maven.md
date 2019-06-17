title: Ubuntu 14.04 配置maven
date: 2015-10-06 16:57:06
tags: [maven,开发环境,开发工具]
categories: [开发环境]
---
### 首先下载maven
* 去官网下载压缩包`apache-maven-3.0.5-bin.tar.gz`，[apache-maven官网下载页面](http://maven.apache.org/download.cgi)

* 解压压缩包

>tar -xvzf apache-maven-3.0.5-bin.tar.gz

把压缩包就放在当前解压目录，或者放到制定目录
>cp -r apache-maven-3.0.5 /usr/xxx/

现在已经创建好了一个`Maven`安装目录`apache-maven-3.0.5`，虽然直接使用该目录配置环境变量之后就能使用`Maven`了，但推荐做法是，在安装目录旁平行地创建一个符号链接，以方便日后的升级：
>sudo ln -s apache-maven-3.0.5 apache-maven

查看一下是否成功
![查看链接是否创建](https://blog-1254094716.cos.ap-chengdu.myqcloud.com/note_Ubuntu配置maven01.png)

* 编辑环境变量

>vi ~/.bashrc

在文件末尾加入以下内容
```bash
export M2_HOME=/usr/xxx/apache-maven
export PATH=$PATH:${M2_HOME}/bin
```

更新一下文件，然后再测试是否成功
```bash
​➜  xxx source ~/.bashrc
➜  xxx mvn -v
Apache Maven 3.0.5 (r01de14724cdef164cd33c7c8c2fe155faf9602da; 2013-02-19 21:51:28+0800)
Maven home: /usr/xxx/apache-maven
Java version: 1.7.0_40, vendor: Oracle Corporation
Java home: /usr/xxx/jdk1.7.0_40/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.13.0-32-generic", arch: "amd64", family: "unix"
```

在用户目录`~/.m2`目录下，添加`settings.xml`，内容如下：
其中最关键的看第29行和第44行，定义了两个仓库，分别是jar包仓库和插件仓库，这样在公司内网下载jar包就非常快
```xml
<?xml version="1.0"?>
<settings xmlns="http://maven.apache.org/POM/4.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                            http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>snapshots</id>
            <username>snapshots</username>
            <password>g2Dki6XCfhi48Bnj</password>
            <filePermissions>664</filePermissions>
            <directoryPermissions>775</directoryPermissions>
        </server>
        <server>
            <id>releases</id>
            <username>{username}</username>
            <password>{yourpwd}</password>
            <filePermissions>664</filePermissions>
            <directoryPermissions>775</directoryPermissions>
        </server>
    </servers>
    <profiles>
        <profile>
            <id>local</id>
            <repositories>
                <repository>
                    <id>QunarNexus</id>
                    <url>http://svn.xxx.xxx.com:8081/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <!-- always , daily (default), interval:X (where X is 
                            an integer in minutes) or never.-->
                        <updatePolicy>daily</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </releases>
                    <snapshots>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>QunarNexus</id>
                    <url>http://svn.xxx.xxx.com:8081/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <checksumPolicy>warn</checksumPolicy>
                    </releases>
                    <snapshots>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>local</activeProfile>
    </activeProfiles>
</settings>
```
