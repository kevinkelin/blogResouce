title: Python解析XML文档
id: 872
permalink: 872
categories:
  - Python
date: 2013-08-28 00:32:24
tags:
  - python
  - xml
---

解析XML主要用到pytohn自带的XML库，其次还是lxml库

# XML结构
先以一个相对简单但功能比较全的XML文档为例
``` xml
<?xml version='1.0' encoding='utf-8'?>
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'>
  <title>dive into mark</title>
  <subtitle>currently between addictions</subtitle>
  <id>tag:diveintomark.org,2001-07-29:/</id>
  <updated>2009-03-27T21:56:07Z</updated>
  <link rel='alternate' type='text/html' href='http://diveintomark.org/'/>
  <entry>
    <author>
      <name>Mark</name>
      <uri>http://diveintomark.org/</uri>
    </author>
    <title>Dive into history, 2009 edition</title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition'/>
    <id>tag:diveintomark.org,2009-03-27:/archives/20090327172042</id>
    <updated>2009-03-27T21:56:07Z</updated>
    <published>2009-03-27T17:20:42Z</published>
    <category scheme='http://diveintomark.org' term='diveintopython'/>
    <category scheme='http://diveintomark.org' term='docbook'/>
    <category scheme='http://diveintomark.org' term='html'/>
    <summary type='html'>Putting an entire chapter on one page sounds
      bloated, but consider this &mdash; my longest chapter so far
      would be 75 printed pages, and it loads in under 5 seconds&hellip;
      On dialup.</summary>
  </entry>
  <entry>
    <author>
      <name>Mark</name>
      <uri>http://diveintomark.org/</uri>
    </author>
    <title>Accessibility is a harsh mistress</title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2009/03/21/accessibility-is-a-harsh-mistress'/>
    <id>tag:diveintomark.org,2009-03-21:/archives/20090321200928</id>
    <updated>2009-03-22T01:05:37Z</updated>
    <published>2009-03-21T20:09:28Z</published>
    <category scheme='http://diveintomark.org' term='accessibility'/>
    <summary type='html'>The accessibility orthodoxy does not permit people to
      question the value of features that are rarely useful and rarely used.</summary>
  </entry>
  <entry>
    <author>
      <name>Mark</name>
    </author>
    <title>A gentle introduction to video encoding, part 1: container formats</title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2008/12/18/give-part-1-container-formats'/>
    <id>tag:diveintomark.org,2008-12-18:/archives/20081218155422</id>
    <updated>2009-01-11T19:39:22Z</updated>
    <published>2008-12-18T15:54:22Z</published>
    <category scheme='http://diveintomark.org' term='asf'/>
    <category scheme='http://diveintomark.org' term='avi'/>
    <category scheme='http://diveintomark.org' term='encoding'/>
    <category scheme='http://diveintomark.org' term='flv'/>
    <category scheme='http://diveintomark.org' term='GIVE'/>
    <category scheme='http://diveintomark.org' term='mp4'/>
    <category scheme='http://diveintomark.org' term='ogg'/>
    <category scheme='http://diveintomark.org' term='video'/>
    <summary type='html'>These notes will eventually become part of a
      tech talk on video encoding.</summary>
  </entry>
</feed>
```

先简单的看一下这个XML的结构<!-- more -->
``` xml
<feed xmlns='http://www.w3.org/2005/Atom' xml:lang='en'> #这里定义了命名空间(namespace) http://www.w3.org/2005/Atom
  <title></title>
  <subtitle></subtitle>
  <id></id>
  <updated></updated>
  <link rel='alternate' type='text/html' href='http://diveintomark.org/'/> #这里的<link>没有text，但是里面有相应的属性
  <entry>
    <author>
      <name></name>
      <uri></uri>
    </author>
    <title></title>
    <link rel='alternate' type='text/html'
      href='http://diveintomark.org/archives/2009/03/27/dive-into-history-2009-edition'/>
    <id></id>
    <updated></updated>
    <published></published>
    <category scheme='http://diveintomark.org' term='diveintopython'/>
    <summary type='html'></summary>
  </entry>
</feed>
```
首先有一个全局的根元素<feed></feed>

在根元素下面有title,subtitle,id,update,link,entry子元素

在entry元素下面还有author,title,link,id,updated,published,category,summary子元素 （姑且称为孙元素）

在author元素下面还有name,uri子元素（这该称为曾孙元素了吧~ 哈哈）

结构还是挺清晰的

下面我们用python的方法来一步步的取出在元素<></>这间的content以为元素内的属性

使用的方法主要有

tree = etree.parse() 解析XML

root = tree.getroot() 得到根元素

root.tag 根元素名称

root.attrib 显示元素的属性

root.findall() 查找元素

# 下面请看代码，都已经将注释与结果写在里面
``` python
import xml.etree.ElementTree as etree #将xml.etree.ElementTree引入
tree = etree.parse('feed.xml') #解析XML
root = tree.getroot()
print root
# <Element {http://www.w3.org/2005/Atom}feed at cd1eb0>

#元素即列表
print root.tag
#{http://www.w3.org/2005/Atom}feed
# ElementTree使用{namespace}localname来表达xml元素

for child in root:
    print child

    # <Element {http://www.w3.org/2005/Atom}title at e2b5d0>
    # <Element {http://www.w3.org/2005/Atom}subtitle at e2b4e0>
    # <Element {http://www.w3.org/2005/Atom}id at e2b6c0>
    # <Element {http://www.w3.org/2005/Atom}updated at e2b6f0>
    # <Element {http://www.w3.org/2005/Atom}link at e2b4b0>
    # <Element {http://www.w3.org/2005/Atom}entry at e2b720>
    # <Element {http://www.w3.org/2005/Atom}entry at e2b510>
    # <Element {http://www.w3.org/2005/Atom}entry at e2b750>
    # 这里只显示一级子元素，而子元素的子元素将不会被遍历

#属性即字典
print root.attrib
#{'{http://www.w3.org/XML/1998/namespace}lang': 'en'}
#我们注意到feed下面的link这个元素有属性
print root[4].attrib
#{'href': 'http://diveintomark.org/', 'type': 'text/html', 'rel': 'alternate'}
print root[3].attrib
#{} 将会得到一个空字典，因为updated元素内没有属性值

#查找元素
entrylist = root.findall('{http://www.w3.org/2005/Atom}entry')
print entrylist
# [<Element {http://www.w3.org/2005/Atom}entry at 18423a0>, <Element {http://www.w
# 3.org/2005/Atom}entry at 18425d0>, <Element {http://www.w3.org/2005/Atom}entry a
# t 1842968>]
print root.findall('{http://www.w3.org/2005/Atom}author')
# 这里将得到一个空列表，因为author不是feed的直接子元素

#查找子元素
entries = tree.findall('{http://www.w3.org/2005/Atom}entry') #先找到entry元素·
title = entries[0].find('{http://www.w3.org/2005/Atom}title')#接着再找title元素
print title.text
#'Dive into history, 2009 edition'

all_links = tree.findall('//{http://www.w3.org/2005/Atom}link') #在元素前面加'//' 则可以在所有元素里查找包括子元素和孙元素
# [<Element {http://www.w3.org/2005/Atom}link at e181b0>,
 # <Element {http://www.w3.org/2005/Atom}link at e2b570>,
 # <Element {http://www.w3.org/2005/Atom}link at e2b480>,
 # <Element {http://www.w3.org/2005/Atom}link at e2b5a0>]

print all_links[0].attrib #将会得到这个Link的属性字典
 # {'href': 'http://diveintomark.org/',
 # 'type': 'text/html',
 # 'rel': 'alternate'}
 
```
关于XML库解析与查找XML文档基本的方法就这些了，现在通过一个实例来学以至用下

# 还是回到微信的XML解析上
微信将用户的信息POST到你的服务器上，基本形式如下
``` xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>1348831860</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[this is a test]]></Content>
<MsgId>1234567890123456</MsgId>
</xml>
```
现在我们来通过上面介绍的方法来获得<Content>元素中的‘this is a test’字段
``` python
import xml.etree.ElementTree as etree

weixinxml = etree.parse('weixinpost.xml')
wroot = weixinxml.getroot()
print wroot.tag

for child in wroot:
    print child.tag

if wroot.find('Content') is not None:
    print wroot.find('Content').text
else:
    print 'Nothing found'
    
```

这样简单几步就可以把想要的内容取出来了

参考文章 [http://woodpecker.org.cn/diveintopython3/xml.html](http://woodpecker.org.cn/diveintopython3/xml.html)