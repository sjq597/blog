title: Ubuntu 14.04 Sublime Text 3 手动安装插件
date: 2015-10-02 00:15:57
tags: [Linux,Sublime Text,插件,Markdown,开发工具]
categories: Linux使用
---
### Ubuntu 14.04 Sublime Text 3 手动安装插件
首先声明，这种方式只针对于在线安装失败的情况下，例如公司封端口，或者远端仓库不稳定的情况，`github`在国内有时候确实不太稳定。

#### 先安装Package Control
* Sublime Text 3 终端安装
先按ctrl+\`调出终端，或者直接 `View > Show Console`
然后在终端中输入下面的代码：
```python
import urllib.request,os,hashlib; 
h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; 
pf = 'Package Control.sublime-package'; 
ipp = sublime.installed_packages_path(); 
urllib.request.install_opener( 
urllib.request.build_opener( 
urllib.request.ProxyHandler()
) 
); 
by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); 
dh = hashlib.sha256(by).hexdigest(); 
print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
**NOTE:** 如果在线安装失败，可以离线安装，具体方法是：

* 下载`Package Control`安装包[Package Control.sublime-package](https://packagecontrol.io/Package%20Control.sublime-package)
* 找到`Preferences > Browse Packages`,打开的主目录为`Packages`，到上一级目录`sublime-text-3`，目录结构如下：
```
➜  sublime-text-3  tree -L 1
.
├── Cache
├── Installed Packages
├── Local
└── Packages
```

* 把下载的`Package Control.sublime-package`离线压缩包拷到`Installed Packages`目录下(不用解压)，然后重启`sublime text 3`，然后在`Preferences`菜单下就可以看到多了两个选项`Package Settings`和`Package Control`，这就说明安装成功了。

#### 安装M`arkdownEditing`插件
* 首先下载插件安装包,并将安装包拷贝到`Sublime Text 3`的安装目录下对应的目录
```
git clone https://github.com/SublimeText-Markdown/MarkdownEditing.git
sudo cp -r MarkdownEditing/ ~/.config/sublime-text-3/Packages
```
打开一个`.md`文件就可以看到效果了。

#### 安装`Markdown Preview`插件
安装上面的插件看到的当然看到的不是最终效果，如果需要预览，还可以装一个`Markdown Preview`的插件，同样的我的还是不能通过包控制器安装，还是说说如何手动安装吧。
**NOTE:** 以下方法来自`Markdown Preview`的[github官方地址](https://github.com/revolunet/sublimetext-markdown-preview)

* 首先下载插件安装包,并将安装包拷贝到`Sublime Text 3`的安装目录下对应的目录
```
git clone git@github.com:revolunet/sublimetext-markdown-preview.git
sudo cp -r sublimetext-markdown-preview ~/.config/sublime-text-3/Packages
```
* 如果需要预览，可以按输入`Shift+Ctrl+p`，输入`Markdown Preview`，可以选择以下几个选项
 - Markdown Preview: Preview in Browser
 - Markdown Preview: Export HTML in Sublime Text
 - Markdown Preview: Copy to Clipboard
 - Markdown Preview: Open Markdown Cheat sheet

默认就选第一个了，最后选`github`或者`markdown`然后就可以在浏览器中查看预览内容了。
**NOTE:** 但是这样太麻烦，可以设置一个快捷键,点击`Preferences->`选择`Key Bindings User`，将其内容改为如下代码，以后直接按`alt+m`就可以在浏览器中预览了。
```
[
    {"keys":["alt+m"], "command": "markdown_preview", "args": { "target": "browser"} }
]
```
