title: 在MAC下加载NTFS移动硬盘或者U盘
comment: true
donate: true
categories:
  - mac
date: 2016-07-12 13:49:01
tags: 
  - 
permalink: use-mac-mount-ntfs-disk

---

在windows下使用的移动硬盘或者U盘大部分都是ntfs格式，在MAC下默认是不支持的，将一块硬盘分两个区一个给windows用一个给MAC用也不大方便，可以借用第三方软件来支持，其实仔细想想这样的功能苹果公司在技术上肯定是可以支持的，只是由于种种原因微软不让其默认支持，以下的方法不借用任何第三方软件，几条命令搞定在MAC下挂载nfts硬盘。
<!-- more -->

1. 打开终端查看磁盘的Volume Name

插件移动硬盘或者U盘，打开终端，输入`diskutil list` 命令来查看磁盘信息
![disk-info](http://ww3.sinaimg.cn/large/795ab47fgw1f5r3wnz6h8j20pm0i3wjm.jpg)

上面的是本机的磁盘，下面那个disk2是U盘，可以看到type是windows_NTFS,NAME是`奔波霸`
记住这个`奔波霸`

2. sudo打开/etc/fstab

`sudo vim /etc/fstab`

有的系统可能没有这个文件，那么就新建一个
输入`LABEL=奔波霸 none ntfs rw,auto,nobrowse`
LABEL为刚才记录的值，这里也可以是中文(有时中文会有问题，最好还是将磁盘设置为英文的)，如果有空格的话使用`\040`代替
![](http://ww1.sinaimg.cn/large/795ab47fgw1f5r4p8i7d0j20pm0iq40b.jpg)

后面的ntfs rw表示把这个分区挂载为可读写的ntfs格式，最后nobrowse非常重要，因为这个代表了在finder里不显示这个分区，这个选项非常重要，如果不打开的话挂载是不会成功的。编辑好以后重新插入磁盘，就能识别到了。

3. 建立一个软链接到Volumes文件夹下
> sudo ln -s /Volumes ~/Desktop/Volumes

这样就在桌面上创建了一个指向/Volumes的链接，以后接上磁盘的时候就可以方便的从桌面上进入磁盘了。

参考文章：http://www.tianwaihome.com/2014/07/mac-osx-ntfs.html



