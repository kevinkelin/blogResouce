title: python的requests初步使用
id: 1079
permalink: 1079
categories:
  - Python
date: 2014-06-15 23:20:53
tags:
  - python
  - requests
---

早就听说requests的库的强大，只是还没有接触，今天接触了一下，发现以前使用urllib，urllib2等方法真是太搓了……

这里写些简单的使用初步作为一个记录
<!-- more -->

# 安装 
[http://cn.python-requests.org/en/latest/user/install.html#install](http://cn.python-requests.org/en/latest/user/install.html#install)

# 发送无参数的get请求
``` python

r = requests.get('http://httpbin.org/get')
print r.text
'''
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Connection": "close",
    "Host": "httpbin.org",
    "User-Agent": "python-requests/2.3.0 CPython/2.6.6 Windows/7",
    "X-Request-Id": "8a28bbea-55cd-460b-bda3-f3427d66b700"
  },
  "origin": "124.192.129.84",
  "url": "http://httpbin.org/get"
'''

# 发送带参数的get请求,将key与value放入一个字典中，通过params参数来传递,其作用相当于urllib.urlencode
``` python
>>> import requests
>>> pqyload = {'q':'杨彦星'}
>>> r = requests.get('http://www.so.com/s',params = pqyload)
>>> r.url
u'http://www.so.com/s?q=%E6%9D%A8%E5%BD%A6%E6%98%9F'
```
# 发送post请求，通过data参数来传递,
``` html

>>> payload = {'a':'杨','b':'hello'}
>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print r.text
{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "a": "u6768",
    "b": "hello"
  },
  "headers": {
    "Accept": "*/*",
    "Accept-Encoding": "gzip, deflate",
    "Connection": "close",
    "Content-Length": "19",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "python-requests/2.3.0 CPython/2.6.6 Windows/7",
    "X-Request-Id": "c81cb937-04b8-4a2d-ba32-04b5c0b3ba98"
  },
  "json": null,
  "origin": "124.192.129.84",
  "url": "http://httpbin.org/post"
}
>>>
```
可以看到，post参数已经传到了form里,data不光可以接受字典类型的数据，还可以接受json等格式
``` html

>>> payload = {'a':'杨','b':'hello'}
>>> import json
>>> r = requests.post('http://httpbin.org/post', data=json.dumps(payload))
```
# 发送文件的post类型
这个相当于向网站上传一张图片，文档等操作，这时要使用files参数
``` html

>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('touxiang.png', 'rb')}
>>> r = requests.post(url, files=files)
```
## 定制headers，使用headers参数来传递
``` html

>>> import json
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> headers = {'content-type': 'application/json'}

>>> r = requests.post(url, data=json.dumps(payload), headers=headers)
```

# 响应内容

## 响应状态码
r = requests.get('http://httpbin.org/get')
print r.status_code
## 响应头
``` html

>>> print r.headers
{'content-length': '519', 'server': 'gunicorn/18.0', 'connection': 'keep-alive', 'date': 'Sun, 15 Jun 2014 14:19:52 GMT', 'access-control-allow-origin': '*', 'content-type': 'application/json'}
```
也可以取到这个个别的响应头用来做一些判断，这里的参数是不区分大小写的

r.headers[‘Content-Type’]

r.headers.get(‘Content-Type’)

## 响应内容，前面已经在应用了

r.text

r.content


# 获取响应中的cookies
``` html

>>> r = requests.get('http://www.baidu.com')
>>> r.cookies['BAIDUID']
'D5810267346AEFB0F25CB0D6D0E043E6:FG=1'
```

也可以自已定义请求的COOKIES
``` html

>>> url = 'http://httpbin.org/cookies'
>>> cookies = {'cookies_are':'working'}
>>> r = requests.get(url,cookies = cookies)
>>>
>>> print r.text
{
  "cookies": {
    "cookies_are": "working"
  }
}
>>>
```
cookies还有很多，因为目前我也还不是很多，以后再扩充吧

# 使用timeout参数设置超时时间

``` html
>>> requests.get('http://github.com', timeout=1)
<Response [200]>
```

如果将时间设置成非常小的数，如requests.get('[http://github.com'](http://github.com), timeout=0.001)，那么如果在timeout的时间内没有连接，那么将会抛出一个Timeout的异常

# 访问中使用session

先初始化一个session对象，s = requests.Session()

然后使用这个session对象来进行访问，r = s.post(url,data = user)

 

参考文章 [http://blog.csdn.net/iloveyin/article/details/21444613](http://blog.csdn.net/iloveyin/article/details/21444613) 基本上都是从这扒的代码

 

以下通过访问人人网来获取首页中的最近来访问，然后再访问查看更多的来访来读取更多的最近来访

20151029更新，和美女刘巍进行讨论，由于很久没有登录人人网，它的登录页面与获得最近来访的页面都有所变动，登录时的url是http://www.renren.com/ajaxLogin/login 获取最近来访是http://www.renren.com/myfoot/whoSeenMe

更多的来访就是以带session的访问http://www.renren.com/myfoot/whoSeenMe
``` python

#coding:utf-8
import requests
import re

url = r'http://www.renren.com/ajaxLogin/login'

user = {'email':'email','password':'pass'}
s = requests.Session()
r = s.post(url,data = user)

html = r.text
visit = []
first = re.compile(r'</span><span class="time-tip first-tip"><span class="tip-content">(.*?)</span>')
second = re.compile(r'</span><span class="time-tip"><span class="tip-content">(.*?)</span>')
third = re.compile(r'</span><span class="time-tip last-second-tip"><span class="tip-content">(.*?)</span>')
last = re.compile(r'</span><span class="time-tip last-tip"><span class="tip-content">(.*?)</span>')
visit.extend(first.findall(html))
visit.extend(second.findall(html))
visit.extend(third.findall(html))
visit.extend(last.findall(html))
for i in visit:
    print i

print '以下是更多的最近来访'
vm = s.get('http://www.renren.com/myfoot/whoSeenMe')
fm = re.compile(r'"name":"(.*?)"')
visitmore = fm.findall(vm.text)
for i in visitmore:
    print i
    
```
[![renren](/image/2014/06/renren_thumb.png "renren")](/image/2014/06/renren.png)