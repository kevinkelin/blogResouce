title: 使用国内镜像源来加速python pypi包的安装
id: 1389
permalink: 1389
comment: false
categories:
  - Python
date: 2015-10-10 22:57:44
tags:
  - python
---

在国内使用pip安装包的时候，很多时候由于各种原因无法正常使用pypi.python.org的源，还好有国内的良心网站提供了国内镜像

网上的方法都是在%HOMEPATH%中新建pip目录，再新建一个pip.ini，可是我试了以后还是不能用，也不知道原作者是否可以用

后来我看了一下它的文档 <!-- more -->

[![image](/image/2015/10/image1.png "image")](/image/2015/10/image1.png) 

这里应该是配制一个%HOME%的环境变量

于是我在环境变量里新建了一个HOME，值为%HOMEPATH%，然后在%HOMEPATH%目录里新建一个pip目录，在里面新建一个pip.ini

里面写入
  ```  python
[global]
index-url = http://pypi.douban.com/simple
```
或者

```  python
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host=pypi.douban.com
```

之后再使用pip install 就可以很顺利的安装了，我这里使用的是douban的安装源

# 20160113 更新

今天在我的Raspberry中无论怎么修改pip.conf都不能生效
无奈只能查看pip的源代码，把源代码中的默认下载地址改了
修改 /usr/lib/python2.7/dist-packages/pip目录下的cmdoptions.py
搜索 pypi.python.org
将其修改为
``` python
index_url = OptionMaker(
    '-i', '--index-url', '--pypi-url',
    dest='index_url',
    metavar='URL',
    default='http://pypi.douban.com/simple/',
    help='Base URL of Python Package Index (default %default).')
```

这里就直接使用http，而不是https，之后再使用sudo pip install 安装就默认走douban的了
还可以修改/usr/lib/python2.7/dist-packages/setuptools/command下面的 easy_install.py
也将里面的
> self.index_url = self.index_url or "http://pypi.python.org/simple"
改为
self.index_url = self.index_url or "http://pypi.douban.com/simple"

这样以后就可以通过 sudo easy_install 来安装，这个也是默认就走douban的源了