title: 在中文windows下使用pywinauto进行窗口操作（一）
id: 926
permalink: 926
categories:
  - Python
date: 2013-10-07 03:42:56
tags:
---

这两天开始接触pywinauto，听说百度的自动化QA也用这个模块，于是来了兴趣，但网上的教程很少，而且基本上都是拿官方的notepad来说，首先中文菜单的支持是问题，其次各种操作也没有写清楚，阅读官方的文档，发现这个东西使用起来还真是非常方便，下面我也以notepad为例来说明一下它的简单操作。

# 安装

## pywinauto;
 [https://sourceforge.net/project/showfiles.php?group_id=157379](https://sourceforge.net/project/showfiles.php?group_id=157379)

## Sendkeys 
[http://www.rutherfurd.net/python/sendkeys/index.html](http://www.rutherfurd.net/python/sendkeys/index.html)

## ctypes (如果你是python2.3或者2.4)

检测你是否安装正确
> from pywinauto import application
> app = application.Application.start("notepad.exe")
> app.notepad.TypeKeys("%FX")

<!-- more -->
都安装好了以后，我们来正式进入pywinauto的世界

# 启动程序
``` python
from pywinauto import application
app = application.Application.start('notepad.exe')
```

start() 函数里也可以接路径+程序名

使用spy++lite查看notepad的信息

[![1](/image/2013/10/1_thumb.png "1")](/image/2013/10/1.png) 

里面的窗口类名与标题文本相关重要，以后的查找窗口基本上都要用的到

现在我们来点击“帮助->关于记事本”操作
``` python
app.Notepad.MenuSelect('帮助->关于记事本'.decode('gb2312'))

```

这里的app是你刚才实例的对象，Notepad是类名，可以从spy++lite中看到,MenuSelect方法可以自动检索Notepad上的菜单选项，

decode(‘gb2312’)方法是把中文强制转换为unicode编码，对于非英文的操作系统都是要转换的，后面还有更简单的方法

# 查找“关于记事本”的窗口

还是使用spy++lite来查看“关于记事本”的信息

[![2](/image/2013/10/2_thumb.png "2")](/image/2013/10/2.png) 

窗口类名：#32770

标题文字：关于“记事本”

官方法文档中有以下两个方法

1\. 通过top_dlg = app.top_window_() 来获得最上面的window，但是官方并不推荐这种方式，目前来说这个“关于记事本”是最上面，但是也不能保证在测试的进程当中有什么意外的进程跑到了上面，一旦有新的进程，那么得到的就是一个错误的对象

2.通过find_dlg = app.window_(title_re = ‘’, class_name = ‘’) 方法获得，这也是为什么我上面说标题文本与窗口类名非常重要的原因，title_re和 class_name这两个可以单独使用也可以一块使用，因为有时没有标题文本，也有时一个窗口类名有多个对象，比如“Edit”有时当一个对话框中有多个输入框时会有多个Edit类名，对于“关于记事本”我们可以通过以下代码获得



中文要进行unicode编码，这里也可以通过decode(‘gb2312’)方法实现，但是不如输入一个U省事~ 呵呵

我们print一下得到的about_dlg

<pywinauto.application.WindowSpecification object at 0x01F0A530>

说明我们得到的是一个application.WindowSpecification对象

# 在”关于记事本”窗口上找到“确定”按钮(button)

在pywinauto中，对话框下面的是controller，button,checkbox,textbox等都是controller

我们可以使用print_control_identifiers() 方法来打印出该窗口中所有的controller



会得到以下的输出
``` html

Control Identifiers:
Static - ''   (L312, T265, R738, B267)
'' '0' '1' 'Static' 'Static0' 'Static1'
Static - ''   (L308, T280, R340, B313)
'2' 'Static2'
Static - 'Microsoft Windows'   (L350, T280, R695, B295)
'Microsoft Windows' 'Microsoft WindowsStatic' 'Static3'
Static - 'u7248u672c 6.1 (u5185u90e8u7248u672c 7601: Service Pack 1)'   (L350, T295, R748, B310)
'Static4' 'u7248u672c 6.1 (u5185u90e8u7248u672c 7601: Service Pack 1)' 'u7248u672c 6.1 (u5185u90e8u7248u672c 7601: Service Pack 1)Static'
Static - 'u7248u6743u6240u6709 xa9 2009 Microsoft Corporationu3002u4fddu7559u6240u6709u6743u5229u3002'   (L350, T310, R710, B325)
'Static5' 'u7248u6743u6240u6709 xa9 2009 Microsoft Corporationu3002u4fddu7559u6240u6709u6743u5229u3002' 'u7248u6743u6240u6709 xa9 2009 Microsoft Corporationu3002u4fddu7559u6240u6709u6743u5229u3002Static'
Static - 'Windows 7 u65d7u8230u7248 u64cdu4f5cu7cfbu7edfu53cau5176u7528u6237u754cu9762u53d7u7f8eu56fdu548cu5176u4ed6u56fdu5bb6/u5730u533au7684u5546u6807u6cd5u548cu5176u4ed6u5f85u9881u5e03u6216u5df2u9881u5e03u7684u77e5u8bc6u4ea7u6743u6cd5u4fddu62a4u3002'   (L350, T325, R710, B385)
'Static6' 'Windows 7 u65d7u8230u7248 u64cdu4f5cu7cfbu7edfu53cau5176u7528u6237u754cu9762u53d7u7f8eu56fdu548cu5176u4ed6u56fdu5bb6/u5730u533au7684u5546u6807u6cd5u548cu5176u4ed6u5f85u9881u5e03u6216u5df2u9881u5e03u7684u77e5u8bc6u4ea7u6743u6cd5u4fddu62a4u3002' 'Windows 7 u65d7u8230u7248 u64cdu4f5cu7cfbu7edfu53cau5176u7528u6237u754cu9762u53d7u7f8eu56fdu548cu5176u4ed6u56fdu5bb6/u5730u533au7684u5546u6807u6cd5u548cu5176u4ed6u5f85u9881u5e03u6216u5df2u9881u5e03u7684u77e5u8bc6u4ea7u6743u6cd5u4fddu62a4u3002Static' 'Windows 7 u65d7u8230u7248 u64cdu4f5cu7cfbu7edfu53cau5176u7528u6237u754cu9762u53d7u7f8eu56fdu548cu5176u4ed6u56fdu5bb6/u5730u533au7684u5546u6807u6cd5u548cu5176u4ed6u5f85u9881u5e03u6216u5df2u9881u5e03u7684u77e5u8bc6u4ea7u6743u6cd5u4fddu62a4u3002Static0' 'Windows 7 u65d7u8230u7248 u64cdu4f5cu7cfbu7edfu53cau5176u7528u6237u754cu9762u53d7u7f8eu56fdu548cu5176u4ed6u56fdu5bb6/u5730u533au7684u5546u6807u6cd5u548cu5176u4ed6u5f85u9881u5e03u6216u5df2u9881u5e03u7684u77e5u8bc6u4ea7u6743u6cd5u4fddu62a4u3002Static1'
Static - ''   (L350, T385, R665, B415)
'Static7' 'Windows 7 u65d7u8230u7248 u64cdu4f5cu7cfbu7edfu53cau5176u7528u6237u754cu9762u53d7u7f8eu56fdu548cu5176u4ed6u56fdu5bb6/u5730u533au7684u5546u6807u6cd5u548cu5176u4ed6u5f85u9881u5e03u6216u5df2u9881u5e03u7684u77e5u8bc6u4ea7u6743u6cd5u4fddu62a4u3002Static2'
SysLink - 'u6839u636e <A>Microsoft u8f6fu4ef6u8bb8u53efu6761u6b3e</A>uff0cu672cu4ea7u54c1u4f7fu7528u6743u5c5eu4e8e:'   (L350, T415, R665, B445)
'SysLink' 'u6839u636e <A>Microsoft u8f6fu4ef6u8bb8u53efu6761u6b3e</A>uff0cu672cu4ea7u54c1u4f7fu7528u6743u5c5eu4e8e:' 'u6839u636e <A>Microsoft u8f6fu4ef6u8bb8u53efu6761u6b3e</A>uff0cu672cu4ea7u54c1u4f7fu7528u6743u5c5eu4e8e:SysLink'
Static - 'kevin'   (L365, T445, R680, B460)
'Static8' 'kevin' 'kevinStatic' 'kevinStatic0' 'kevinStatic1'
Static - ''   (L365, T460, R680, B475)
'Static9' 'kevinStatic2'
Button - 'u786eu5b9a'   (L672, T503, R747, B524)
'Button' 'u786eu5b9a' 'u786eu5b9aButton'
```

static，SysLink,button等是它类型，后面接的是title，都是unicode的，这里面就有没有title的controller，再后面的（L,T,R,B）是这个控件的位置，分别对应着左上右下

在”关于记事本”窗口上找到“确定”按钮，可以通过app.window_()方法，传入的参数可以是title,也可以是class_name,所以我说这两个值相当重要，一直在用，这里的title支持正则表达式，非常方便

在app上先找到about_dlg,然后再about_dlg上找确定button

app.window_(title_re = u'关于“记事本”').window_(title_re = u'确定')，然后通过Click()方法来单击这个button

另外一种方法也是官方推荐的在非英文系统下的方法
``` python
OK = u'确定'
about_dlg[OK].Click()
```

这个的意思就是在about_dlg下找到u’确定’，看起来比上面要简练好理解，理解了这种方式，接下来还有更简单的，都不用找about_dlg

直接 app[u'关于“记事本”'][u'确定'].Click()

# 在记事本里写点东西

这个其实在校验pywinauto的时候已经做过了全用TypeKeys函数，但是这里如果要输入中文还是要u一下
``` python
app.notepad.TypeKeys(u"杨彦星")

```

# 一个比较恶心的问题

在MenuSelect函数中不支持正则，完全是全文匹配，如我输入

dig = app.Notepad.MenuSelect("编辑->替换".decode('gb2312')) 是找不到对象的

必须要

dig = app.Notepad.MenuSelect("编辑(E)->替换(R)".decode('gb2312')) 这样才行，得把(R) (E)写上才行，但是奇怪的是上面的“帮助->关于记事本”就不用输入，所以说是一个挺恶心的问题，我也不知道这是为什么…… 

最后把上面的函数合并一下，跑下来应该会很快
``` python
#! /usr/bin/env python
#coding=gbk


import time
from pywinauto import application
app = application.Application.start('notepad.exe')
app.Notepad.MenuSelect('帮助->关于记事本'.decode('gb2312'))
time.sleep(.5)

#这里有两种方法可以进行定位“关于记事本”的对话框
#top_dlg = app.top_window_() 不推荐这种方式，因为可能得到的并不是你想要的
about_dlg = app.window_(title_re = u"关于", class_name = "#32770")#这里可以进行正则匹配title
#about_dlg.print_control_identifiers()
app.window_(title_re = u'关于“记事本”').window_(title_re = u'确定').Click()
app.Notepad.MenuSelect('帮助->关于记事本'.decode('gb2312'))
time.sleep(.5) #停0.5s 否则你都看不出来它是否弹出来了！
ABOUT = u'关于“记事本”'
OK = u'确定'
#about_dlg[OK].Click()
#app[ABOUT][OK].Click()
app[u'关于“记事本”'][u'确定'].Click()

app.Notepad.TypeKeys(u"杨彦星")
dig = app.Notepad.MenuSelect("编辑(E)->替换(R)".decode('gb2312'))
Replace = u'替换'
Cancle = u'取消'
time.sleep(.5)
app[Replace][Cancle].Click()
dialogs = app.windows_()
```

先写这么多，以后再写点更多的应用~