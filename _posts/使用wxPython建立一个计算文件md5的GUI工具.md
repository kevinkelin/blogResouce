title: 使用wxPython建立一个计算文件md5的GUI工具
id: 1350
permalink: 1350
comment: false
categories:
  - Python
date: 2015-06-26 01:13:43
tags:
  - python
---

最近在看wxPython，一开始觉得它的布局好乱，半天整不明白，这里通过写一个小工具来记录一下wxPython的一些基本使用

小工具最终是下面这个样子，将文件拖到上面会自动计算其md5与size

[![1](/image/2015/06/1_thumb.png "1")](/image/2015/06/1.png) 
 <!-- more -->
下面是全部的代码
  ```  python
#coding:gbk
import wx
import optparse
import time,hashlib
import threading
import os

def checkMD5(pefile):
    try:
        f = open(pefile,'rb')
        data = f.read()
        m = hashlib.md5()
        m.update(data)
        f.close()
        return m.hexdigest()
    except:
        return 'error'

def getFileSize(filename):
    try:
        size = int(os.path.getsize(filename))
        return size
    except:
        return 'error'

#线程函数
class FuncThread(threading.Thread):
    def __init__(self, func, *params, **paramMap):
        threading.Thread.__init__(self)
        self.func = func
        self.params = params
        self.paramMap = paramMap
        self.rst = None
        self.finished = False

    def run(self):
        self.rst = self.func(*self.params, **self.paramMap)
        self.finished = True

    def getResult(self):
        return self.rst

    def isFinished(self):
        return self.finished

def doInThread(func, *params, **paramMap):
    t_setDaemon = None
    if 't_setDaemon' in paramMap:
        t_setDaemon = paramMap['t_setDaemon']
        del paramMap['t_setDaemon']
    ft = FuncThread(func, *params, **paramMap)
    if t_setDaemon != None:
        ft.setDaemon(t_setDaemon)
    ft.start()
    return ft

class FileDropTarget(wx.FileDropTarget):
    def __init__(self, filetext,md5tx,filesizetx):
        wx.FileDropTarget.__init__(self)
        self.filepath = filetext
        self.md5tx = md5tx
        self.filesizetx = filesizetx

    def OnDropFiles(self,  x,  y, fileNames):
        filename = fileNames[0].encode('gbk')
        print filename
        print type(filename)
        self.filepath.SetValue(filename)
        md5 = doInThread(checkMD5,filename)
        filesize = doInThread(getFileSize,filename)
        while True:
            if not md5.isFinished():
                time.sleep(0.5)
            else:
                self.md5tx.SetValue(md5.getResult())
                break

        while True:
            if not filesize.isFinished():
                time.sleep(0.5)
            else:
                self.filesizetx.SetValue(str(filesize.getResult()))
                break

class Frame(wx.Frame): #Frame 进行初始化
    def __init__(self,title):
        wx.Frame.__init__(self,None,title=title,size = (400,300))
        boxSizer = wx.BoxSizer(wx.VERTICAL)

        self.panel = wx.Panel(self)

        # boxSizer.Add(self.panel,1,wx.EXPAND|wx.ALL) #wx.ALL 周围的距离，EXPAND扩充到全部

        filepath = wx.StaticText(self.panel,-1,"FileDir(请将文件拖到本对话框中)")
        filetext = wx.TextCtrl(self.panel,-1,"",size=(350,20))

        md5st = wx.StaticText(self.panel,-1,"MD5")
        md5tx = wx.TextCtrl(self.panel,-1,size=(250,20))

        filesizest = wx.StaticText(self.panel,-1,'FileSize')
        filesizetx = wx.TextCtrl(self.panel,-1,size=(250,20))

        # hashst = wx.StaticText(self.panel,-1,'Hash')
        # hashtx = wx.TextCtrl(self.panel,-1,size=(250,20))

        boxSizer.Add(filepath,0,wx.EXPAND|wx.LEFT|wx.TOP,border=10)
        boxSizer.Add(filetext,0,wx.LEFT|wx.TOP,border=10)
        boxSizer.Add((-1,20))
        boxSizer.Add(md5st,0,wx.LEFT|wx.TOP,border=10)
        boxSizer.Add(md5tx,0,wx.LEFT|wx.TOP,border=10)
        boxSizer.Add((-1,10))
        boxSizer.Add(filesizest,0,wx.LEFT|wx.TOP,border=10)
        boxSizer.Add(filesizetx,0,wx.LEFT|wx.TOP,border=10)
        # boxSizer.Add((-1,10))
        # boxSizer.Add(hashst,0,wx.LEFT|wx.TOP,border=10)
        # boxSizer.Add(hashtx,0,wx.LEFT|wx.TOP,border=10)

        dropTarget = FileDropTarget(filetext,md5tx,filesizetx)
        self.panel.SetDropTarget( dropTarget )

        self.panel.SetSizer(boxSizer)

class App(wx.App): ##继承wx.App
    def OnInit(self): ##还没有调起来的时候读取初始化
        self.frame = Frame('MD5&size信息')
        self.frame.Centre()
        self.frame.Show(True)
        return True

def killSelf(evt = None):
    os.system('taskkill /F /T /PID %d >NUL 2>NUL' % win32process.GetCurrentProcessId())

if __name__ == '__main__':
    parser = optparse.OptionParser()
    parser.add_option('-x', '--no-update', dest = 'test', action = 'store_true', help = 'start without update')
    parser.add_option('-t', '--no-update-test', dest = 'test2', action = 'store_true', help = 'start without update debug')
    options, args = parser.parse_args()
    if options.test:
        print("-x param")
    if options.test2:
        print("-t param")
```

一点点的解释

class App与App().MainLoop()是固定写法，在class App下有一个def OnInit方法来初始化主的Frame，将其居中并且Show()出来，没什么好说的，主要看一下Frame的定义

这个小工具使用的是boxSizer来布局，为了简单我只使用了一个boxSizer，将里面的所有控件采用VERTICAL(垂直)的方式来布局，如果想要将MD5与后面的文本框放在同一行，那么就需要添加一个水平的boxSizer，然后那将这个水平的boxSizer再放入主的boxSizer

boxSizer = wx.BoxSizer(wx.VERTICAL) #初始化一个垂直的boxSizer，也是整个框架的主Sizer

self.panel = wx.Panel(self) #初始化一个panel，这个panel是放了放之后的控件的

filepath = wx.StaticText(self.panel,-1,"FileDir(请将文件拖到本对话框中)")

filetext = wx.TextCtrl(self.panel,-1,"",size=(350,20))

md5st = wx.StaticText(self.panel,-1,"MD5")

md5tx = wx.TextCtrl(self.panel,-1,size=(250,20))

filesizest = wx.StaticText(self.panel,-1,'FileSize')

filesizetx = wx.TextCtrl(self.panel,-1,size=(250,20))

上面是初始化相应的静态文本与文本框，方法中的第一个参数是其所在的父类窗口，这里也就是self.panel，其实也可以不用panel，而是将其直接放入到boxSizer中

boxSizer.Add(filepath,0,wx.EXPAND|wx.LEFT|wx.TOP,border=10) 将filepath加入到主的boxSizer中，这里一开始我有一些困惑，一开始我一直以为先将所有的控件放入到panel中，然后再将panel放入到boxSizer中，但是这样是不对的，而应该是直接就入到boxSizer中，将该控件的父类设置为panel，之后就没有将panel放入boxSizer这一步操作，wx.LEFT|wx.TOP,border=10 这个参数表示的是该控件距离上来左各有10个像素的距离，再使用wx.EXPAND来使其充分的填充其所在的区域，我曾经想，可否设置成距离上10px,左20px，但是貌似不能这样设置，Add函数里只能有一个border参数，换句话说只能设置相同的数值，之后我再找找是否可以实现。

boxSizer.Add((-1,20)) #这个是添加一个空距离，距离上20px

dropTarget = FileDropTarget(filetext,md5tx,filesizetx)

self.panel.SetDropTarget( dropTarget )

这个是放该窗口类添加一个拖拽方法，也是比较固定的写法

上面的class FileDropTarget中的__init__与OnDropFiles方法也是固定的方法，只是里面的处理函数不同。

wxPython中的一些style与flag等参数在布局中使用需要一些经验，还有它的很多控件和与之绑定的方法，要想熟练掌握还需要下一些工夫，下面两个网站算是介绍比较详细，要多多查阅

[http://wxpython.org/Phoenix/docs/html/1classindex.html](http://wxpython.org/Phoenix/docs/html/1classindex.html "http://wxpython.org/Phoenix/docs/html/1classindex.html")

[http://docs.wxwidgets.org/2.8.9/wx_classref.html](http://docs.wxwidgets.org/2.8.9/wx_classref.html "http://docs.wxwidgets.org/2.8.9/wx_classref.html")