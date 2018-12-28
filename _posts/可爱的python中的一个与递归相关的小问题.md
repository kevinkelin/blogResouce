title: 可爱的python中的一个与递归相关的小问题
id: 821
permalink: 821
categories:
  - Python
date: 2013-06-23 01:41:21
tags:
---

可爱的python一书中有一个练习题，是在一个目录中查找特定扩展名的文件，并且读取里面的内容，然后用户输入一个关键词，在这些文件中进行搜索，如果找到后就把这一行内容打印出来，他提出的问题是如果里面还有目录，目录里面还有更深的目录，解决这个问题，我想到的只能是递归。。。
``` python
#coding:utf-8
import os,sys

def cdGrep(keyword,filepath):
    filelist = os.listdir(filepath)

    for cdc in filelist:
        if os.path.isdir(cdc):
            filepath2 = os.path.join(filepath,cdc)
            cdGrep(keyword,filepath2)

        elif '.txt' in cdc:
#            print filepath+cdc
            f = open(filepath+''+ cdc)
            for line in f.readlines():
                linelower = line.lower()
                if keyword in linelower:

                    print '您查找的关键词在%s中找到'% (filepath+''+cdc)
                    print line
                f.close()

if __name__ == '__main__':
    keword = raw_input('请输入想要查询的关键字')
    pw = os.getcwd()
    cdGrep(keword,pw)
```

<!-- more -->
但是这里出现一个问题，就是目录只能进行底下一层，再深的目录就不进行遍历了，很是郁闷，于是在德问网上问了下，很快就有人指出了我的错误出在哪了
>if os.path.isdir(cdc): 
这里不应该写成cdc，如果是cdc的话如果他是一个目录的话，再到下一层系统是不能再识别的，应该是
>if os.path.isdir(os.path.join(filepath,cdc)):
将cdc与之前的目录进行结合，这样系统才能正常的识别目录

这样，程序就可以很好的遍历目录了