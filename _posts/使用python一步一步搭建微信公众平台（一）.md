title: 使用python一步一步搭建微信公众平台（一）
id: 871
permalink: 871
categories:
  - Python
date: 2013-08-24 19:03:05
tags:
  - python
---

最近无聊，想玩玩微信的公众平台，后来发现乐趣无穷啊~

使用的工具，python 新浪SAE平台，微信的公众平台

你需要先在微信的公众平台与新浪SAE平台上各种注册，微信平台注册的时候需要你拍张手持身份证的照片，还有几天的审核期

微信公众平台：[http://mp.weixin.qq.com](http://mp.weixin.qq.com/ "微信公众平台")
 <!-- more -->
新浪SAE：[http://sae.sina.com.cn/](http://sae.sina.com.cn/)

等待微信公众审核通过后，登录公众平台后，点击高级功能。将会看到需要提供一个接入信息：

![微信接口配置](http://static.oschina.net/uploads/img/201309/04223229_FNSJ.jpg) 微信接口配置[/caption]

那么我们需要一个网址作为接口（这时就需要SAE上搭建Python的一个应用），Token呢，就是相当于我们和微信之间约定的“密码”，这里可以随便填写英文或者数字，但实测输入纯数字有时会有问题，所以还是字符串比较靠谱。

# 第一步，在SAE上搭建python的应用
在下图的应用里选择python应用。

[![iELK05K8pJTk2](/image/2013/08/iELK05K8pJTk2.jpg)](/image/2013/08/iELK05K8pJTk2.jpg)

 

填好二级域名和应用名称等，选择好语言。这里我们使用Python开发选择web应用。创建好应用之后，在代码管理中创建一个新的版本。而后我们可以选择编辑代码。能够实现在线编辑，根本用不着配置本地环境，SVN等等。当然像这种轻量级的应用在线编辑器就可以了，SVN的话还不如在线编辑好用

# 第二步，编写index.wsgi

因为我们使用的是web.py框架，因为其良好的xml解析，想了解web.py的童鞋可以移步 [http://webpy.org/docs/0.3/tutorial.zh-cn](http://webpy.org/docs/0.3/tutorial.zh-cn)

首先编写config.yaml
``` python
name: yangyanxing
version: 1

libraries:
- name: webpy
  version: "0.36"

- name: lxml
  version: "2.3.4"
```
注意严格的缩进，差一个空格你就废了！而且调试的时候很不好发现问题。。。

 

接着我们继续编写index.wgsi
``` python
# coding: UTF-8
import os

import sae
import web

from weixinInterface import WeixinInterface

urls = (
'/weixin','WeixinInterface'
)

app_root = os.path.dirname(__file__)
templates_root = os.path.join(app_root, 'templates')
render = web.template.render(templates_root)

app = web.application(urls, globals()).wsgifunc()
application = sae.create_wsgi_app(app)</pre>
```
简单解释一下，
>from weixinInterface import WeixinInterface
这里我们需要再创建一个weixinIterface的py文件，你也可以将这个类写在index.wgsi文件中，只是这样看起来会乱乱的

新建一个weixinIterface.py文件，注意大小写，写入以下代码
``` python
# -*- coding: utf-8 -*-
import hashlib
import web
import lxml
import time
import os
import urllib2,json
from lxml import etree

class WeixinInterface:

    def __init__(self):
        self.app_root = os.path.dirname(__file__)
        self.templates_root = os.path.join(self.app_root, 'templates')
        self.render = web.template.render(self.templates_root)

    def GET(self):
        #获取输入参数
        data = web.input()
        signature=data.signature
        timestamp=data.timestamp
        nonce=data.nonce
        echostr=data.echostr
        #自己的token
        token="yangyanxing" #这里改写你在微信公众平台里输入的token
        #字典序排序
        list=[token,timestamp,nonce]
        list.sort()
        sha1=hashlib.sha1()
        map(sha1.update,list)
        hashcode=sha1.hexdigest()
        #sha1加密算法

        #如果是来自微信的请求，则回复echostr
        if hashcode == signature:
            return echostr
```
这里定义了一个GET方法，是根据微信公众平台的要求，进行的token验证，因为这里我们定义了templates_root为根目录下的templates，所以还要在根目录下创建一个目录teplatest的目录
开发者通过检验signature对请求进行校验（下面有校验方式）。若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，否则接入失败。

signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。

加密/校验流程：
1\. 将token、timestamp、nonce三个参数进行字典序排序
2\. 将三个参数字符串拼接成一个字符串进行sha1加密
3\. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信</pre>
因为微信是将验证信息GET发出去的，所以这里使用了GET方法来取得值并且返回相应用值

保存全部，现在回到微信的公众平台高级管理界面

![微信接口配置](http://static.oschina.net/uploads/img/201309/04223229_FNSJ.jpg)

在url里面填写你在新浪SAE里应用名称并且加上/weixin，如：http://XXXX.sinaapp.com/weixin token随便输入，只要注意更改weixinInterface.py中的token就行了，输入好了以后点击提交，如果没有什么问题的话就会通过验证！

 

# 第三步，新建一个简单的自动回复的方法
鹦鹉学舌，就是用户说什么，它也回复什么，没什么用，只是随便玩玩！

在weixinInterface.py里继续添加代码
``` python
def POST(self):
    str_xml = web.data() #获得post来的数据
    xml = etree.fromstring(str_xml)#进行XML解析
    content=xml.find("Content").text#获得用户所输入的内容
    msgType=xml.find("MsgType").text
    fromUser=xml.find("FromUserName").text
    toUser=xml.find("ToUserName").text
    return self.render.reply_text(fromUser,toUser,int(time.time()),u"我现在还在开发中，还没有什么功能，您刚才说的是："+content)
```
这个def 是和上一个GET同级的，注意缩进

接着我们在templates目录下创建reply_text.xml模板文件，写入以下代码
``` xml
$def with (toUser,fromUser,createTime,content)
<xml>
<ToUserName><![CDATA[$toUser]]></ToUserName>
<FromUserName><![CDATA[$fromUser]]></FromUserName>
<CreateTime>$createTime</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[$content]]></Content>
</xml>
```

注意这里的toUser与fromUser是刚才post的是相反的，因为这里的toUser也就是POST函数里的fromUser，这里的fromUser也就是POST函数里的toUser,msgType是text

 

全部保存，现在就在用你的个人微信关注一下你创建的公众微信号，然后随便输入些内容，如果没有什么问题，你将会收到一条鹦鹉学舌的回复内容！

 

参考文章：[http://www.nilday.com/%E5%88%A9%E7%94%A8sae%E6%90%AD%E5%BB%BA%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%B9%B3%E5%8F%B0%EF%BC%88%E4%B8%80%EF%BC%89web-py%E5%AE%9E%E7%8E%B0%E7%9A%84sae-hello-world/](http://www.nilday.com/%E5%88%A9%E7%94%A8sae%E6%90%AD%E5%BB%BA%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%B9%B3%E5%8F%B0%EF%BC%88%E4%B8%80%EF%BC%89web-py%E5%AE%9E%E7%8E%B0%E7%9A%84sae-hello-world/)