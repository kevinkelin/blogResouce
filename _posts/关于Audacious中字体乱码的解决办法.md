title: 关于Audacious中字体乱码的解决办法
id: 216
permalink: 216
categories:
  - Linux相关
date: 2012-01-10 23:30:03
tags:
---

安装完Audatious可以正常播放后，可以播放列表里确是惨不忍睹……

[![](/image/2012/01/audacious1.jpeg "audacious1")](/image/2012/01/audacious1.jpeg)

原因不说了，因为在windows下使用的不同标准的字体
 <!-- more -->
更改设置 文件-首选项- playlist

[![](/image/2012/01/audacious2.jpeg "audacious2")](/image/2012/01/audacious2.jpeg)

在Custom string 上输入 %f

在上面的Fallback character encodings里输入 GBK

关闭 - 点击 播放列表- 刷新

OK 一切正常了～