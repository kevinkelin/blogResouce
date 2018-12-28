title: python处理网页中的cookie
id: 908
permalink: 908
categories:
  - Python
date: 2013-09-10 11:16:49
tags:
  - python
---

最近想要搭建个小黄鸡的微信应用，但是一路来发现，现在很多的方法都已经不能用了，我在本地试过，也试用过requests获取session的方法也不行，但是这经过这次折腾，倒是对cookie有了更多的了解，以下以python登录人人网的例子来介绍cookie的使用。
cookie的定义可以参考百度百科，[http://baike.baidu.com/subview/835/5062332.htm](http://baike.baidu.com/subview/835/5062332.htm) 但是里面说的乱七八糟的，而且好多重复，使用firefox或者fiddler等工具来抓取cookie
在firefox下使用httpFox插件来查到人人网的登录时需要POST的地址是[http://www.renren.com/ajaxLogin](http://www.renren.com/ajaxLogin "http://www.renren.com/ajaxLogin")&#160;
[![getUrl](/image/2013/09/getUrl_thumb.png "getUrl")](/image/2013/09/getUrl.png)
<!-- more -->
而且查看到需要POST的DATA有email和password 

python通过cookielib来处理cookie，以下是简单的代码
``` python
>>> import urllib
>>> import urllib2,cookielib
>>> login_page = "http://www.renren.com/ajaxLogin"
>>> cj = cookielib.CookieJar()
>>> opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
>>> opener.add_handler = [('User-agent','Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)')]
>>> data = urllib.urlencode({"email":'username',"password":'password'})
>>> opener.open(login_page,data)
<addinfourl at 53653216 whose fp = <socket._fileobject object at 0x03307B70>>
>>> if cj:
... for index,cookie in enumerate(cj):
... print index,':',cookie
...
0 : <Cookie _de=90D533AE20EB964CA96710977F452897 for .renren.com/>
1 : <Cookie anonymid=hlehtdzg-8359yw for .renren.com/>
2 : <Cookie first_login_flag=1 for .renren.com/>
3 : <Cookie id=224967207 for .renren.com/>
4 : <Cookie loginfrom=null for .renren.com/>
5 : <Cookie p=9beb60859c004bcaf0a32ff2c973473d7 for .renren.com/>
6 : <Cookie societyguester=86b6a6006002ab6316f708521ab50bfc7 for .renren.com/>
7 : <Cookie t=86b6a6006002ab6316f708521ab50bfc7 for .renren.com/>
8 : <Cookie xnsid=fa53da51 for .renren.com/>
9 : <Cookie t=30af9ffe774f4d6f242e92da1ccd6670 for .renren.com/xtalk/>
10 : <Cookie feedType=224967207_hot for .www.renren.com/>
11 : <Cookie JSESSIONID=abc3IP9kEhTExblxcRfeu for www.renren.com/>
>>> 
```

可以和firebug或者httpFox中得到的cookie进行对比，值可能不一致，但key基本上是一致的，你每次登录应该都不一致

[![getCookie](/image/2013/09/getCookie_thumb.png "getCookie")](/image/2013/09/getCookie.png) 

&#160;

我也尝试过使用fidder模拟发送没有cookie的POST数据，但是没有得到想要的返回值

[![withoutcookie](/image/2013/09/withoutcookie_thumb.png "withoutcookie")](/image/2013/09/withoutcookie.png) 

而加上cookie信息以后就可以正常的跳转到自己的主页了

[![withcookie](/image/2013/09/withcookie_thumb.png "withcookie")](/image/2013/09/withcookie.png) 

[![pg](/image/2013/09/pg_thumb.png "pg")](/image/2013/09/pg.png) 

好了，基本上了解了python中使用cookie来发送登录信息，现在我们来写一个小脚本来登录自己人人网。
``` python
#encoding=utf-8
import urllib2
import urllib
import cookielib
def renrenBrower(url,user,password):
    login_page = "http://www.renren.com/ajaxLogin"
    try:
        cj = cookielib.CookieJar()
        opener=urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
        opener.addheaders = [('User-agent','Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)')]
        data = urllib.urlencode({"email":'user',"password":'password'})
        opener.open(login_page,data)
        op=opener.open(url)
        data= op.read()
        return data
    except Exception,e:
        print str(e)
print renrenBrower("http://www.renren.com/home","用户名","密码")

```

&#160;

这样就可以将自己首页的信息显示出来了，其实在登录完以后，还可以接着写脚本来获取自己想要的信息，如朋友的新鲜事，

这里重点是使用SGMLParser来解析html代码

以下代码参考[http://www.oschina.net/code/snippet_148170_10661](http://www.oschina.net/code/snippet_148170_10661)&#160;
``` python
from sgmllib import SGMLParser
import sys,urllib2,urllib,cookielib
class spider(SGMLParser):
    def __init__(self,email,password):
        SGMLParser.__init__(self)
        self.h3=False
        self.h3_is_ready=False
        self.div=False
        self.h3_and_div=False
        self.a=False
        self.depth=0
        self.names=""
        self.dic={}

        self.email=email
        self.password=password
        self.domain='renren.com'
        try:
            cookie=cookielib.CookieJar()
            cookieProc=urllib2.HTTPCookieProcessor(cookie)
        except:
            raise
        else:
            opener=urllib2.build_opener(cookieProc)
            urllib2.install_opener(opener)

    def login(self):
        url='http://www.renren.com/PLogin.do'
        postdata={
                  'email':self.email,
                  'password':self.password,
                  'domain':self.domain
                  }
        req=urllib2.Request(
                            url,
                            urllib.urlencode(postdata)
                            )

        self.file=urllib2.urlopen(req).read()
        #print self.file
    def start_h3(self,attrs):
        self.h3 = True
    def end_h3(self):
        self.h3=False
        self.h3_is_ready=True

    def start_a(self,attrs):
        if self.h3 or self.div:
            self.a=True
    def end_a(self):
        self.a=False

    def start_div(self,attrs):
        if self.h3_is_ready == False:
            return
        if self.div==True:
            self.depth += 1

        for k,v in attrs:
            if k == 'class' and v == 'content':
                self.div=True;
                self.h3_and_div=True   #h3 and div is connected
    def end_div(self):
        if self.depth == 0:
            self.div=False
            self.h3_and_div=False
            self.h3_is_ready=False
            self.names=""
        if self.div == True:
            self.depth-=1
    def handle_data(self,text):
        #record the name
        if self.h3 and self.a:
            self.names+=text
        #record says
        if self.h3 and (self.a==False):
            if not text:pass
            else: self.dic.setdefault(self.names,[]).append(text)
            return
        if self.h3_and_div:
            self.dic.setdefault(self.names,[]).append(text)

    def show(self):
        type = sys.getfilesystemencoding()
        for key in self.dic:
            print ( (''.join(key)).replace(' ','')).decode('utf-8').encode(type),
                  ( (''.join(self.dic[key])).replace(' ','')).decode('utf-8').encode(type)

if __name__ =='__main__':
    renrenspider=spider('email','password')
    renrenspider.login()
    renrenspider.feed(renrenspider.file)
    renrenspider.show()
```
