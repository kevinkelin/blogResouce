title: 使用python进行文件夹对比
categories: 
  - Python
date: 2015-12-29 23:09:30
tags: 
  - python
permalink: use-python-dircompare

---
最近写一个小脚本，在使用系统急救箱扫描并且处理完后，要将处理以后的样本和提供的标准进行对比
已经提供了样本与处理以后的文件，我要写的脚本就是启急救箱并且将两个文件夹进行比较
启动并且扫描比较好实现，但是在进行文件夹对比的时候走了一些弯路
上网查到python的标准库里有一个filecmp类，这个类可以对文件夹或者文件进行对比，使用起来还是比较方便的
[filecmp官方文档/](http://python.usyiyi.cn/python_278/library/filecmp.html "filecmp官方文档")

使用最下面的例子程序，我又对其进行了扩充
我想达到的目的就是先对两个文件夹进行比较，输出不同的文件，然后再输出各自文件夹中独有的文件
 <!-- more -->
``` python
#coding:gbk
from filecmp import dircmp


def show_diff_files(dcmp):
    for name in dcmp.diff_files:
        print "diff_file %s found in %s and %s" % (name, dcmp.left,dcmp.right)
    for sub_dcmp in dcmp.subdirs.values():
        show_diff_files(sub_dcmp)
        
def show_only(dcmp):
    if dcmp.left_only:
        ave_rst = 1
        for i in dcmp.left_only:
            print "%s只存在于%s中"%(i,dcmp.left)
    if dcmp.right_only:
        for i in dcmp.right_only:
            print "%s只存在于%s中"%(i,dcmp.right)
    for sub_dcmp in dcmp.subdirs.values():
        show_only(sub_dcmp)

def compare(dir1,dir2):
    dcmp = dircmp(dir1,dir2)
    show_diff_files(dcmp)
    show_only(dcmp)
```