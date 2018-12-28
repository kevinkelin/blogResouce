title: Python中的urllib2库的使用
id: 911
permalink: 911
categories:
  - Python
date: 2013-09-12 01:06:43
tags:
  - python
---

今天研究了下urllib2这个库的使用，才发现以前有很多不明白的东西，现在写下来也做个记录

# 最基础的应用
``` python
import urllib2
url = r'http://www.baidu.com'
html = urllib2.urlopen(url).read()

```

客户端与服务器端通过request与response来沟通，客户端先向服务端发送request，然后接收服务端返回的response

urllib2提供了request的类，可以让用户在发送请求前先构造一个request的对象，然后通过urllib2.urlopen方法来发送请求
``` python
import urllib2
url = r'http://www.baidu.com'
req = urllib2.Request(url)
html = urllib2.urlopen(req).read()
```
<!-- more -->
上例中先使用req = urllib2.Request(url)实例化一个resquest对象，接下来使用urllib2.urlopen(req)来打开这个网页

我们注意到在实例化Request对象的时候，队了url是必须的，还有几个默认的参数

[![attars](/image/2013/09/attars_thumb.png "attars")](/image/2013/09/attars.png) 

基中data与header也是使用的比较多的，一些需要登录的才能浏览的网站经常需要这两个参数
``` python
import urllib
import urllib2

url = 'http://www.baidu.com/'
values = {'name' : 'Michael Foord', 'location' : 'Northampton','language' : 'Python' }
data = urllib.urlencode(values)
req = urllib2.Request(url,data)
response = urllib2.urlopen(req)
the_page = response.read()

```

这个例子是向百度发送几个数据，这个例子是会返回一个错误页面，很正常，因为我们在访问百度的时候并不需要post什么信息，post了倒是会出错

百度是找不到相应的网页就会报错。

当然这个是POST数据，也可以用在GET方法，稍将上面的代码进行改造

百度是通过[http://www.baidu.com/s?wd=XXX](http://www.baidu.com/s?wd=XXX) 来进行查询的，这样我们需要将{‘wd’:’xxx’}这个字典进行urlencode
``` python
#coding:utf-8
import urllib
import urllib2

url = 'http://www.baidu.com/s'
values = {'wd':'杨彦星'}
data = urllib.urlencode(values)
print data
url2 = url+'?'+data
response = urllib2.urlopen(url2)
the_page = response.read()

```

以下以模拟登录人人网然后再显示首页内容为例来详细说明一下cookie的使用，以下是文档中给的例子，我们就通过改造这个例子来实现我们想要的功能
``` python
import cookielib, urllib2
cj = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
r = opener.open("http://example.com/")
```

&#160;
``` python
#coding:utf-8
import urllib2,urllib
import cookielib

url = r'http://www.renren.com/ajaxLogin'

#创建一个cj的cookie的容器
cj = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
#将要POST出去的数据进行编码
data = urllib.urlencode({"email":email,"password":pass})
r = opener.open(url,data)
```

当你看到有cj的时候,说明你已经访问了登录页面,是否正常登录你现在还看不出来,可以通过访问http://www.renren.com/home 来查看

上面的代码有两点要说明,我也是看了很长时间才明白

r = opener.open(url,data) 这句,为什么要使用opener这个对象来open,而不是用utllib2,urlopen?不光是例子里这么写,我们才这么写,通过改造我们也可以使用urllib2.urlopen,其实是因为opener是urllib2.bulid_opener创造出来的, 但是你可以这样理解,他build出来后,自已却并没有安装使用它,也没有它的属性与方法,如果想使urllib2也具有opener的属性与方法,可以先使用urllib2.install_opener(opener)来"安装"这个opener,安装完以后就可以使用urllib2来操作了
``` python
#coding:utf-8
import urllib2,urllib
import cookielib

url = r'http://www.renren.com/ajaxLogin'

#创建一个cj的cookie的容器
cj = cookielib.CookieJar()
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
urllib2.install_opener(opener)
#将要POST出去的数据进行编码
data = urllib.urlencode({"email":email,"password":pass})
#r = opener.open(url,data)如果没有上面的urllib2.install_opener方法，就必须这样写了
r = urllib2.urlopen(url,data)
html = urllib2.urlopen('http://www.renren.com/home').read()

```

同样urllib2还有proxy相关的handle,基本的思路和这个差不多