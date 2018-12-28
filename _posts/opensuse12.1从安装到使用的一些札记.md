title: opensuse12.1从安装到使用的一些札记
id: 210
permalink: 210
categories:
  - Linux相关
date: 2012-01-10 23:11:27
tags:
  - linux
---

opensuse也玩了很久了，但我还是觉得自己是一个初学者，写这篇文章主要是想记录一下自己使用中的问题，以后如果再次安装好做一个记录

本文主要参考网张上的文章

[http://cn.opensuse.org](http://cn.opensuse.org/) 中文wiki

[http://hi.baidu.com/winland0704/home](http://hi.baidu.com/winland0704/home)  winland的百度空间

[http://lug.ustc.edu.cn/sites/opensuse-guide/](http://lug.ustc.edu.cn/sites/opensuse-guide/)   Unofficial guide
<!-- more -->
一、安装

如果有条件还是选择光盘安装，如果没有条件或者想搞些新鲜的玩意，选择硬盘安装，我使用的是用Burg4DOS硬盘安装openSUSE

安装教程请到[http://www.vdisk.cn/down/index/9152371A9887](http://www.vdisk.cn/down/index/9152371A9887) 这里下载，这个教程很简单，也很好用，根着教程操作，但是到了分区这一步要小心，这里面如果出现什么磁盘空间不足之类的提示，在这个界面把 create LVM based proposal选项选中

[![](/image/2012/01/disk.png "分区")](/image/2012/01/disk.png)

硬盘安装之后有一个遗留问题，就是镜像所在的磁盘不能开机自动挂载上，要挂载的话还要输入一次ROOT密码，这个我找了半天也没有找到方法，但是这样却有一个好处，就是以后在安装这个镜像里的软件时，可以直接安装，我感觉这个可能就是问题所在

2、安装这后我认为最先要解决的是各种多媒体的codec

可以用一键安装

<center>[![ymp](http://lug.ustc.edu.cn/sites/opensuse-guide/images/oneclick/codecs.png) <span style="font-size: x-large;"> KDE </span>](http://opensuse-community.org/codecs-kde.ymp)</center><center>[![ymp](http://lug.ustc.edu.cn/sites/opensuse-guide/images/oneclick/codecsg.png) <span style="font-size: x-large;">GNOME</span>](http://opensuse-community.org/codecs-gnome.ymp)</center><center></center>我安装的KDE桌面，还是推荐使用KDE桌面

主要的codec有

libxine1-codecs（多媒体播放引擎，支持MPEG-4等）

k3b-codecs（K3b的MP3支持）

ffmpeg（有名的音视频编解码器，支持众多格式）

lame（MP3格式支持）

gstreamer-0_10-plugins-bad gstreamer-0_10-plugins-ugly gstreamer-0_10-ffmpeg（这三个是GStreamer多媒体播放支持） libdvdcss2（如果您不播放DVD可以跳过这条

<span style="color: #ff0000;">注意</span>：在刚刚安装完系统的时候，会有一个系统进程一直在用的软件，这时候你既不能安装软件，也不能做软件源的操作，等会吧， 做了一次更新检查后就好了。 在安装软件的时候，有时候会默认添加很多软件源，而且这些软件源默认也是做自动刷新的，所以注意如果不用的话要取消添加这些源， 即使添加了也要把自动刷新取消，可以定期做一次更新，要不然每次做软件的操作会有很长的等待时间，很烦人。 3.添加一些常用的软件源 parkman: 这个几乎是每一个linux用户必须添加的，很多多媒体的软件等都需要它，unofficial guide上推荐的台湾源我是怎么也添加不上去 所以我就不用那个源了，我用的是 http://packman.inode.at/suse/12.1/ 这个源 或者你可以找到很多别的好用的packman源， 软件源里一定要有的，OSS开源 non-oss非开源 update更新,我添加了国内的163镜像源， 使用帮助[http://mirrors.163.com/.help/opensuse.html](http://mirrors.163.com/.help/opensuse.html) 还有一些源都是些牛人建立的，也不错，里面有一些国人常用的软件

*   [http://download.opensuse.org/repositories/home:/hillwood/openSUSE_12.1/](http://download.opensuse.org/repositories/home:/hillwood/openSUSE_12.1/)
*   [http://download.opensuse.org/repositories/home:/swyear/openSUSE_12.1/](http://download.opensuse.org/repositories/home:/swyear/openSUSE_12.1/)
*   [http://download.opensuse.org/repositories/home:/stecue/openSUSE_12.1/](http://download.opensuse.org/repositories/home:/stecue/openSUSE_12.1/)
*   [http://download.opensuse.org/repositories/home:/opensuse_zh/openSUSE_12.1/](http://download.opensuse.org/repositories/home:/opensuse_zh/openSUSE_12.1/)
自己选择一些就可以了，以后你会增加很多的。

添加完软件源后要做的当然是安装软件了，其实常用的软件在安装opensuse的时候已经自动安装上了，但这里就安装的是自己喜欢 别的软件 如 Audaciou Smplayer 等

打开 yast 点击安装软件 搜索 MPlayer、SMPlayer、Audacious，AMSN安装，会自动提示安装必要的库

其他的常用的软件

Chrome 这个是我必须安装的 [https://www.google.com/chrome?platform=linux](https://www.google.com/chrome?platform=linux)

Skype: http://skype.tom.com/download/linux/skype-2.2.0.25-suse.i586.rpm

Fcitx: 这个是我后来才发现的非常好用的输入法，因为我使用的是五笔输入法，在linux上以前一直在ibus下使用万能五笔，但是不 知道12.1的怎么回事，还是按照原来的方法添加了万能五笔后，既没有添加成功，默认安装的输入法也丢失了，无奈之下，上网上搜索了下都说Fcitx这个输入法中的五笔相当好用，于是将系统中原来的ibus和ibus-table remove掉了，安装了Fcitx，这个输入法太惊艳了，我一下子就喜欢上了，爱不释手啊～～