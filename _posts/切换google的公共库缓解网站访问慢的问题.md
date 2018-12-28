title: 切换google的公共库缓解网站访问慢的问题
id: 1052
permalink: 1052
categories:
  - Python
date: 2014-06-07 22:58:32
tags:
---

近期谷(zheng)歌(fu)抽疯，很多google的公共库文件访问缓慢或者根本就是无法访问，很多网站前端以前都是直接引用google的地址，这会或多或少的影响网站打开速度。本人博客也“不幸”引入了一些google的公共库，换了几个服务器，无论是BAE还是SAE或者国外的服务器再或者是国内的服务器，都很慢，今天查了查，将其换为360网站卫士提供的公共库资源（[http://libs.useso.com/](http://libs.useso.com/)），效果果然提高不少。

这个是由360网站卫士CDN驱动的常用前端公共库以及和谐使用Google公共库&字体库的调用方法。

打开Chrome开发者工具(F12),打开网站，查看到一些引用都是error，其中一个是 <!-- more -->

[http://ajax.googleapis.com/ajax/libs/jquery/1.4.3/jquery.min.js](http://ajax.googleapis.com/ajax/libs/jquery/1.4.3/jquery.min.js)&#160;

这个我检查了下，发现是我主题模板里的footer.php里的一句引用，于是替换成了以下。

[http://ajax.useso.com/ajax/libs/jquery/1.4.3/jquery.min.js](http://ajax.useso.com/ajax/libs/jquery/1.4.3/jquery.min.js)&#160;

之后还有一个fonts.googleapis.com/css?family=Open+Sans 没有找到，找这个文件可以费了点劲，我在主题模板里没有找到这句话，查找见面源代码根据位置也找不到，在用notepad++搜了几个wordpree的文件夹以后还是没有发现，无奈文件太多，没想到藏这么深，于自已写个脚本来搜索wordpress所有文件。。。
``` python  
#coding:utf-8
import os,sys

def listFile(path, isDeep=True):
    _list = []
    if isDeep:
        try:
            for root, dirs, files in os.walk(path):
                for fl in files:
                    _list.append('%s%s' % (root, fl))
        except:
            pass
    else:
        for fn in glob.glob( path + os.sep + '*' ):
            if not os.path.isdir(fn):
                _list.append('%s' % path + os.sep + fn[fn.rfind('')+1:])
    return _list

pwd = os.getcwd()
print pwd
cut = listFile(pwd)
flag = 1

for i in cut:
    filename = os.path.split(i)[1]
    ext = os.path.splitext(i)[1]
    if ext == '.php' or ext == '.js':
        f = open(i)
        for j in f.readlines():
            if 'fonts.googleapis.com' in j:
                print i
                print j
                flag = 0
if flag:
    print 'TT'
```

将它放到wordpress目录下运行，不一会就找到了原来是wp-includes目录下的script-loader.php，而且只有这一个文件，大概在602行左右

$open_sans_font_url = "//fonts.googleapis.com/css?family=Open+Sans:300italic,400italic,600italic,300,400,600&subset=$subsets";

将其改为
$open_sans_font_url = "//fonts.useso.com/css?family=Open+Sans:300italic,400italic,600italic,300,400,600&subset=$subsets";

```

保存之后上传，这两个文件，再打开网站试下，果真比原来快了不少！希望对于有类似问题的朋友有帮助。