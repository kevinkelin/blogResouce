title: 美化opensuse中的字体
id: 221
permalink: 221
categories:
  - Linux相关
date: 2012-01-10 23:37:57
tags:
---

原文地址

[http://forums.opensuse.org/ae-ae-chinese/aes-aeoe-e-e-ae-zae-aeoe/c-ae-zae-aeoe/470464-opensuse-c-aeoe-oec-zcs-ae-ae-az-e-c-i-oec-ae-mactype.html](http://forums.opensuse.org/ae-ae-chinese/aes-aeoe-e-e-ae-zae-aeoe/c-ae-zae-aeoe/470464-opensuse-c-aeoe-oec-zcs-ae-ae-az-e-c-i-oec-ae-mactype.html)

openSUSE 目前最完美的中文字型设置，类似 Mactype
<!-- more -->
1.添加linuxsir中文官方源

http://download.opensuse.org/repositories/home:/opensuse_zh/openSUSE_12.1/

安装里面的freetype2

安装，然后 Konsole 输入 infctl settyle 「styles」，默认的 style 有 infinality / linux / osx / osx2 / win7 / win98 / winxp 建议大家用 osx。

这个命令我不知道我使用的对不对，我是这么输入的，infctl settyle osx 结果就是字体变了，还挺好看的