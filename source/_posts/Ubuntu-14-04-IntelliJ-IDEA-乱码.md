title: Ubuntu 14.04下 IntelliJ IDEA 乱码
date: 2015-10-25 22:46:23
tags: [IDEA,Linux,开发工具]
categories: 开发环境
---
在Ubuntu下，由于缺少宋体，导致中文路径出现乱码，解决办法是先下载字体，然后在IDEA中设置使用宋体即可。
乱码有几种情况，

### 编辑器以及调试信息乱码
解决方案，选择`File-->settings-->Editor-->File Encoding`
![IDEA 中设置默认字符编码](http://7xn9y9.com1.z0.glb.clouddn.com/Ubuntu%2014.04下%20IntelliJ%20IDEA%20乱码01.png)
如上图所示，两个编码都选择`UTF-8`，这样可以保证以后新建的文件编码方式，防止乱码。

### IDEA 其他窗口乱码
如下图所示：
![IDEA 中窗口乱码](http://7xn9y9.com1.z0.glb.clouddn.com/Ubuntu%2014.04下%20IntelliJ%20IDEA%20乱码02.png)
出现这样的原因是linux系统提供的字体不支持中文的显示，在idea中，默认的是ubuntu字体，该字体并不支持中文显示。因此，还需要自己下载一个支持中文显示的字体。
* 下载simsun字体，[百度盘下载地址](http://pan.baidu.com/s/1pJ06y2B)
* 将下载好的字体添加到系统字体库中

```bash
sudo cp Downloads/simsun.ttf /usr/share/fonts
```

* 重启IDEA，设置字体
重启idea，然后选择`File->settings->Appearance & Behavior->Appearance`.如下所示：
![IDEA中设置simsun字体](http://7xn9y9.com1.z0.glb.clouddn.com/Ubuntu%2014.04下%20IntelliJ%20IDEA%20乱码03.png)
在右边勾选上`Override default fonts by(....)`,并在`Name`选项中选择刚刚添加的SimSun字体。
