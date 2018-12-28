title: Ubuntu从安装到设置的一点记录
id: 1032
permalink: 1032
categories:
  - Linux相关
date: 2014-03-06 23:43:25
tags:
  - linux
  - Ubuntu
---

1.硬盘安装：http://www.linuxidc.com/Linux/2013-10/91565.htm

2.Nvidia显卡的安装，http://blog.csdn.net/wzcqr0501/article/details/8498659 曾尝试安装，但第一次失败且改不回来，于是不准备安装了，自带的Nvidia显卡已经很好了

3.修改hosts改变google的解析 sudo gedit /etc/hosts

4.默认启动ubuntu后自动挂载windows分区 http://www.linuxidc.com/Linux/2013-02/79679.htm

创建挂载目录 sudo mkdir  /media/windows 等等

查看磁盘分区的UUID sudo  blkid  记录UUID

编辑fstab sudo gedit  /etc/fstab 写入如下内容

UUID=D06ABBA96ABB8AAC /media/windows ntfs default 0 0

5.解决中文乱码问题

sudo gedit /etc/profile
<!-- more -->
在最后添加：

export PATH=$PATH GST_ID3_TAG_ENCODING=GBK:UTF-8:GB18030
export PATH=$PATH GST_ID3V2_TAG_ENCODING=GBK:UTF-8:GB18030

6.导入windows字体 http://www.cnblogs.com/zhj5chengfeng/p/3251009.html

> sudo mkdir /usr/share/fonts/winfonts
sudo cp ~/WinFonts/* /usr/share/fonts/winfonts
cd /usr/share/fonts/winfonts
sudo chmod 744 *
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -f -v

重启电脑

7.安装fcitx小企鹅输入法 http://www.cnblogs.com/yuemengke/archive/2013/04/09/3010207.html
> sudo apt-get install fcitx fcitx-config-gtk fcitx-sunpinyin fcitx-table-wbpy

8.安装qq http://www.ubuntusoft.com/thread-225-1-1.html

