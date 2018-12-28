title: python中的中文编码问题
id: 1050
permalink: 1050
categories:
  - Python
date: 2014-06-02 00:25:02
tags:
  - python
---

在编程领域中，中文编码问题一向是个很头疼的问题，今天写点总结一下解决的方法

unicode与utf-8和gbk之间的转换，涉及到的函数有decode与encode

首先明确一下，python中的字符串都是以某种编码来存储的，就中文来说，以gbk与utf-8的，你虽然可以这样定义一个变量 s = u’杨彦星’,但是你可以用type(s) 来看一下它的类型，此时这个s 不是字符串，而是unicode类型，当你定义s = ‘杨彦星’的时候，这时type(s)才是str类型，但是当你直接定义s = ‘杨彦星’时会在不同的场景下出现不同的问题，windows下的cmd还好，换到一个utf-8的解析器里就是乱码，所以最好的方式是在定义有中文（或者说非英文的）字符串时以unicode来定义，然后再去解码输出。

将字符串decode成unicode

s = ‘杨彦星’

s_uni = s.decode(‘gbk’) #将s转换成unicode，decode的时候就加上这个字符原来的编码类型

s_utf8 = s_uni.encode(‘utf-8’)

s_gbk = s_uni.encode(‘gbk’)

<!-- more --> 

decode() 函数需要字符本来的编码作为参数，encode()函数需要字符想要转换成的编码

当想要保存成特定编码的文件时，就需要使用codecs库

import codecs

s =u '杨彦星'

f = codecs.open('test.txt','w','utf-8')

f.write(s)

f.close()

这时可以打开test.txt查看文件的编码，这时就会是utf-8的，写入的中文要是unicode