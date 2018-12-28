title: 通过小程序练习Python中关于os模块的方法
id: 397
permalink: 397
categories:
  - Python
date: 2012-11-16 01:14:18
tags:
  - python
---

通过一个小程序来熟悉一下os模块中的方法，OS模块在以后会经常的使用，操作目录与文件等
<!-- more -->

```python
import os
print os.getcwd()
for tmpdir in ('/tmp',r'c:temp',r'd:temp'):
    if os.path.isdir(tmpdir):  ##判断上面元组中的元素是否存在并且为一个目录
        break
else:
    print 'No temp dir available'
    tmpdir = ''
    os.chdir('d:')    ##我这里直接转到d分区了
    os.makedirs('temp1')  ##建立temp1文件夹
    print os.getcwd()  ## 获得当前目录路径

if tmpdir:
    os.chdir(tmpdir)  ##切换文件夹
    cwd = os.getcwd()
    print 'Current temp dir is %s ' % cwd
    print 'Now it will create an dir...'
    dirname = os.listdir(cwd)  ##显示当前目录下的文件，放到一个列表中
    print dirname
    os.makedirs('example1')  ##建立一个新的文件夹，example
    os.chdir('example1')
    cwd = os.getcwd()
    print 'Now the current dir is %s ' % cwd
    print 'The original dir listing is:'
    print os.listdir(cwd)

    print 'Now it will create a test file'
    fobj = file('test.txt','w')
    fobj.write('foon')
    fobj.write('barn')
    fobj.close()
    print 'Now after create a test file the listing is :'
    print os.listdir(cwd)  ##这里会创建一个文件，里面有foo bar 这两行字符

    print 'Now it will change the test.tex to test.ini'
    os.rename('test.txt','test.ini')  ##rename
    print os.listdir(cwd)

    path = os.path.join(cwd,os.listdir(cwd)[0])  ##path 此时为 d:tempexample1test.ini
    print 'The full file pathname is %s' % path
    print os.path.split(path)  ## ['d:tempexample1', 'test.ini']  split() 函数返回 dirname(目录名) 与 basename(文件名+扩展名)的一个元组
    print os.path.splitext(os.path.basename(path))  ##('test', '.ini') splittext() 函数返回filename(文件名)
```

[![1352999163_9540](/image/2012/11/1352999163_9540.png)](/image/2012/11/1352999163_9540.png)