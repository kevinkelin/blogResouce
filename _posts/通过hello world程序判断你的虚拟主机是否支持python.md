title: 通过hello world程序判断你的虚拟主机是否支持python
id: 897
permalink: 897
categories:
  - Python
date: 2013-09-25 01:37:52
tags:
---

今天在RC上买了台虚拟主机玩玩，看它上面说支持python语言，于是试了下

写一个hello world的小程序
``` shell
#!/usr/bin/python
print 'Content-Type: text/plainn'
```

以二进制方式上传到自已的空间上，访问 [http://static.yangyanxing.com/test.py](http://static.yangyanxing.com/test.py)


居然一次性成功了，完美地显示hello world
<!-- more -->

可是在网上查了一下，发现大部分还是出现500服务器错误，主要是以下几个原因

RC的python路径为/usr/bin/python 所以要在python文件首行加上

有时还要改.htaccess，在上面添加对python的解析

AddHandler cgi-script .py

没有以二进制方式上传

上传后有时要改下这个文件的权限为755，我的是744也可以执行

还是500服务器错误，请检查代码写是否有问题，别少一个引号多加一个分号什么的