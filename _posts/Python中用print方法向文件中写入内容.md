title: Python中用print方法向文件中写入内容
id: 749
permalink: 749
categories:
  - Python
date: 2013-02-02 23:40:21
tags:
  - python
---

一个小功能，我就是想用print功能实现，不想用write
``` python
import os

os.chdir("/usr/tem")
char="my name is yangyanxing"
f = open("test.txt","w")
print >>f,char
```

但是Python3中还可以用以下的方式
``` python
import os

os.chdir("/usr/tem")
char="my name is yangyanxing"
f = open("test.txt","w")
print(char,file=f)
```
<!-- more -->
也可以考虑用with as结构，会简单与周全些
``` python
try:
    with("man.txt","w") as data:
        print >>data,char
except IOError as err:
    print("Write error"+ str(err))
```

这种可以不用考虑finally情况

不同版本真是麻烦。。。。