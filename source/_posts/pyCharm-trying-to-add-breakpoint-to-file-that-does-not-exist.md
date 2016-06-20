title: pyCharm trying to add breakpoint to file that does not exist
date: 2016-02-20 15:52:55
tags: [Python, 开发工具]
categories: Python笔记
---
pyCharm在调试程序的时候出现了这个问题,解决办法就是在`Run--> View Breakpoints`里面(或者直接按Ctrl+Shift+F8)
这里可以看到所有的断点，哪怕是其他工程项目里面的，这就会导致几个问题，当你删掉了某些文件的时候，断点记录还在，所以在运行的时候会报错：
> 
pydev debugger: warning: trying to add breakpoint to file that does not exist: /home/workspace/work/Python/upload_file/upload_file/views.py (will have no effect)

原因就是我把这个删掉了，但是之前在调试这个文件的时候设置过断点，所以记录还在，把不需要的断点记录勾选掉就可以了。
