title: 在SUSE中用Pidgin整合QQ与新浪微博
id: 227
permalink: 227
categories:
  - Linux相关
date: 2012-01-14 01:57:08
tags:
---

TX官方出的qq for linux实在是太恶心了，好友不全也就罢了，关键是还总崩溃，忍不了了，一开始我还总是用WEB qq还将就下，可以感觉这个实在是不方便，在网上找了些方法 ， lumqq很久之前就不更新也，不不支持现在的qq登录了，于是找到了传说中的Pidgin，下载了一个最新的2.10.0 软件源，默认的光盘里就有这个软件

下载安装，其实这个软件可以兼容MSN Gtalk facebook,我还额外的下载了一个新浪微博的插件 pidgin-microblog-sina

<!--StartFragment-->http://download.opensuse.org/repositories/home:/opensuse_zh/openSUSE_12.1/   软件源

但这个插件表现一般，比官方的air客户端差好多，当时只是好奇

第一次启动的时候会弹出添加账号窗口，这里选择新浪微博

[![](/image/2012/01/pidgin1.jpeg "pidgin add sina account ")](/image/2012/01/pidgin1.jpeg)

填写好后点击添加会弹出与新浪微博的授权页面，授权后会得到一串数字，填写到弹出的对话框中即可

添加QQ有点困难

首先下载libqq,<!-- more -->

[http://code.google.com/p/libqq-pidgin/downloads/list](http://code.google.com/p/libqq-pidgin/downloads/list)

下载后用7za 解压，当时我的系统中还没有安装7za，于是用
<pre>zypper install p7zip</pre>

命令安装

接着解压
<pre>7za x libqq.so_0.71_i386.7z</pre>

e是解压到当前路径

x是解压到压缩包命名的目录下

把解压出来的libqq.so复制到/usr/lib/purple-2 文件夹下

重启Pidgin，这时在添加账号里就有了QQ选项了

添加后界面虽说有点简单，但基本的功能都已经包括了，别奢望太多，毕竟不是100多M的QQ……

[![](/image/2012/01/pidgin2.jpeg "the qq UI in pidgin ")](/image/2012/01/pidgin2.jpeg)