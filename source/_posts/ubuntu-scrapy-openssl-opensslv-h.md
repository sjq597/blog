title: ubuntu scrapy openssl/opensslv.h
date: 2017-04-30 12:53:08
tags: [Python]
categories: Python笔记
---
搞一下爬虫,装了一下scrapy报错:
```
build/temp.linux-x86_64-2.7/_openssl.c:434:30: fatal error: openssl/opensslv.h: No such file or di
rectory
  compilation terminated.
  error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
  
  ----------------------------------------
  Failed building wheel for cryptography
  Running setup.py clean for cryptography
```
具体报错就是上面那个:
> 
fatal error: openssl/opensslv.h

解决方法很简单:
```
sudo apt-get install libssl-dev
```
