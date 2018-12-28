title: 使用web.py在BAE建立自已的电影网站
id: 953
permalink: 953
categories:
  - Python
date: 2013-11-01 00:06:43
tags:
  - python
  - webpy
---

最近在网上看了一篇文章使用web.py在BAE上建立电影网站，[http://www.51bigfool.com/%E6%88%91%E6%98%AF%E5%A6%82%E4%BD%95%E7%94%A8bae%E5%92%8Cweb-py%E6%89%93%E9%80%A0%E8%B1%86%E7%93%A3%E7%94%B5%E5%BD%B1top100%E7%9A%84.html](http://www.51bigfool.com/%E6%88%91%E6%98%AF%E5%A6%82%E4%BD%95%E7%94%A8bae%E5%92%8Cweb-py%E6%89%93%E9%80%A0%E8%B1%86%E7%93%A3%E7%94%B5%E5%BD%B1top100%E7%9A%84.html) 我自已也在此基础上做了一些改进，也在一点点的熟悉使用web.py这个框架，可以看一下我弄了一半的应用 [http://movie.yangyanxing.com](http://movie.yangyanxing.com)

准备 BAE web.py

# 在BAE上建立一个python的应用
快速创建即可，选择Iframe

[![create](/image/2013/11/create_thumb.jpg "create")](/image/2013/11/create.jpg) 
 <!-- more -->
之后进入云引擎将环境类型设置为python

[![python](/image/2013/11/python_thumb.png "python")](/image/2013/11/python.png) 

先简单看一下目录结构如下

│&#160; index.py    
│&#160; model.py     
│     
├─static     
│&#160; └─images     
│&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; lg_movie_a12_2.png     
│     
└─templates     
&#160;&#160;&#160; │&#160; add.html     
&#160;&#160;&#160; │&#160; base.html     
&#160;&#160;&#160; │&#160; index.html     
&#160;&#160;&#160; │&#160; play.html     
&#160;&#160;&#160; │&#160; show.html     
&#160;&#160;&#160; │&#160; top250style.css

新建一个Mysql，进入phpmyadmin导入所需的表

&#160;
``` html
    CREATE TABLE DoubanTop250(
    id INT AUTO_INCREMENT ,
    moviename TEXT,
    score FLOAT,
    url TEXT,
    PRIMARY KEY ( id )
```

我先按照原来的教程添加了相应的代码
model.py
``` python
# _*_ coding:utf-8 _*_
import web
import web.db
from bae.core import const

dbname = "数据库名字"

db = web.database(
    dbn='mysql',
    host=const.MYSQL_HOST,
    port=int(const.MYSQL_PORT),
    user=const.MYSQL_USER,
    passwd=const.MYSQL_PASS,
    db=dbname
)

def additem(name, rating, address):
    return db.insert('DoubanTop250', moviename=name, score=rating, url=address)

def get_items():
    return db.select('DoubanTop250', order='id')

def get_item(id):
    return db.select('DoubanTop250', where='id=$id', vars=locals())[0]
```

index.py

``` python
#_*_ coding:utf-8 _*_
import web
import model
import os
import re

urls = (
    "/", "Index",
    "/play/(d+)", "Play"
)

t_globals = {
    'datestr': web.datestr
}

app_root = os.path.dirname(__file__)
template_root = os.path.join(app_root, 'templates/')
render = web.template.render(template_root, base='base', globals=t_globals)

class Index:
    def GET(self):
        items = model.get_items()
#       return render.test(items)
        return render.index(items)

class Play:
    def POST(self, id):
        id = int(id)
        item = model.get_item(id)
        return render.play(item)

app = web.application(urls, globals()).wsgifunc()
from bae.core.wsgi import WSGIApplication
application = WSGIApplication(app)
```

base.html
``` html

$def with (page)
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html  xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title> 杨彦星的自娱自乐</title>
        <link type="text/css" rel="stylesheet" href="/templates/top250style.css" />
    </head>

    <body>
        <div id="header">
            <a target="_blank" href="http://movie.douban.com/chart" title="豆瓣电影排行旁"><img src="/static/images/lg_movie_a12_2.png" ></a>
        </div>

        <div id="main">
          <h1> 这是瞎写的我会乱说吗|<a target="_blank" href = "/addmovie/">添加电影</a></h1>
            $:page
        </div>

        <div id="footer">
            <div id="copyright">
                <p>本站纯属自娱自乐与学习python之用，电影啥的全是网上收集</p>
              <a href = "/">返回首页</a>
                <p><em>Proudly powered by Web.py</em></p>
            </div>
        </div>

    </body>
</html>
```

index.html
``` html
$def with(items)
        <table>
            <tr>
                <th>排名</th>
                <th>影片名</th>
                <th>得分</th>
                <th>百度影音播放</th>
            </tr>
            $for item in items:
            <tr>
                <td id="tdstyle">$item.id</td>
                <td id="tdstyle">$item.moviename</td>
                <td id="tdstyle">$item.score</td>
                <td id="tdstyle">
                    <form action="/play/$item.id" method="post">
                      <input type="submit" value="播放" />
                    </form>
                </td>
            </tr>
        </table>
```

play.html
``` html
$def with(item)
<h2>$item.id $item.moviename</h2>
<blockquote>
        <object classid="clsid:02E2D748-67F8-48B4-8AB4-0A085374BB99" width="600" height="400" id="BaiduPlayer" name="BaiduPlayer" onError=if(window.confirm('请您先安装百度影音软件,然后刷新本页才可以正常播放.')){window.open('http://player.baidu.com')}else{self.location='http://player.baidu.com'}>
        <PARAM NAME='URL' VALUE=$item.url>
        <PARAM NAME='Autoplay' VALUE='1'>
        </object>

<a title="返回首页" href="http://kanimdb.duapp.com"> 返回首页 </a>
</blockquote>
```

top250style.css
``` css
body{
  background-image: url(tw.jpg);

  background-position:right;
  background-color: #fff;
  margin-left:15%;
  margin-right:15%;
  border: 2px dotted gray;
  padding: 20px 20px 20px 20px;
  font-family: sans-serif;
}

h1 {
  font-family: sans-serif;
  font-weight: bolder;;
  color: green;
  border-bottom: 2px solid gray;
  text-align: center;
}

table {
  width: 500px;
  text-align: center;
  margin-top: 30px;
  margin-left: 20%;
  margin-right: 20%;
  border: thin solid black;
  border-collapse: collapse;
}

th, #tdstyle {
  border: thin dotted gray;
  padding: 5px;
  background-color: silver;
}
th {
  background-color: #fcba7a;
}

#main {
  padding-top: 20px;
  padding-bottom: 20px;
}

#bdplaer {
  padding-left: 10px;
  padding-right: 10px;
}

#header {
  text-align: center;
}

#footer {
  padding: 10px, 10px, 10px, 10px;

  text-align: center;
}

#copyright {
  padding: 10px, 10px, 10px, 10px;
  font-size: small;
}
```

修改app.conf
``` python
handlers:
  - url : /static/images/(.*)
    script: /static/images/$1
  - url : /templates/(.*)
    script: /templates/$1
  - url : /(.*)
    script: index.py

  - expire : .jpg modify 10 years
  - expire : .swf modify 10 years
  - expire : .png modify 10 years
  - expire : .gif modify 10 years
  - expire : .JPG modify 10 years
```

好了，现在这个应用基本上就可以看了，不过现在请忘了以上的代码。因为我们要为其添加一些自已的想要的东西

首先我们要添加“添加影片”功能

修改urls配置
``` python

urls = (
    "/","Index",
    "/play/(d+)","Play",
    "/add/","Add",
    "/addmovie/","addmovie"
)
```

这里的addmovie 其实只是想render到一个模板中

添加class addmovie实现

``` python
class addmovie:
  def GET(self):
    return render.add()
```

添加add.html
``` html
<form action="/add/" method="post">
电影名:<input type="text" name = "name"><br>
得分:<input type="text" name = "rating"><br>
播放地址:<input type="text" name = "address"><br>
<input type="submit" value="add">
</form>
```

很简单的表单，我前端不会，只会写简单的html

这里name值可以随便写，有意义或者你自已认识即可，反正后面的python代码会做处理

这里将movie的信息post到/add/中

转为urls里已经配置了

"/add/","Add",

下面来写class Add实现
``` python
class Add:
  def POST(self):
    movieinfo = web.data()
    #model.additem(movieinfo)
    #return render.show(movieinfo)
    postlist = movieinfo.split('&')
    movie = []
    for i in postlist:
      movie.append(i.split('=')[1])
    model.additem(urllib.unquote(movie[0]),movie[1],urllib.unquote(movie[2]))
    web.seeother("/")
```

这里的代码写的太ugly了，但是可以满足需要，如果有更好的实现可以和我说下

先将post来的数据使用&分割，将会得到name=xxx,rating=xxx,address=xxx的列表

接着来获取相应的值，也就是再将列表中的各个项再用‘=’进行分割，然后将值append到一个新的列表中

然后将新的列表中的各个值进行unquote，要不你插入到数据库中的将不是中文等，而是quote过的数据

插件结束后跳转到首页 web.seeother(“/”)

好了，现在我们完成了添加电影的操作

# 完成后你现在就去找些电影和百度影音地址来添加看看
但是我发现一个新的问题，百度影音在非IE浏览器上是不好用的，这还了得……

在网上找方法，有一个通过 javacsript先判断浏览器的appname来决定使用哪个，[http://pcbbs.baidu.com/thread-62871-1-1.html](http://pcbbs.baidu.com/thread-62871-1-1.html)

这个是我试过了，可以实现在chrome与FF下调用百度影音，但是我要传的$item.url不能传入javascrip中，传入后就是$item.url，在网上查找说可以要写两个$，我也试过了，但是还是不能播放，于是我决定放弃IE用户，让所有播放都走非IE的方法中，爱咋地咋地。

百度这点做的就不如qvod，人家官方都出了在非IE浏览器上调用的方法，而百度还在使用IE，真是不思进取，我也决定在网页上调用qvod播放接口

那么新的问题又来了，程序怎么知道你是要用qvod还是百度来播放？其实很简单，你在添加播放地址的时候，qvod的地址是以qvod开始的，百度影音是以bdhd开始的，只要判断一下$item.url是以哪个开头的就可以了

但是网上更多的还是thunder,edk等协议的影视资源，这些无论是百度还是qvod都无法播放，于是我想到了现在很流行的云点播，迅雷的云点播可以播放，但是非白金会员只能播放一段时间，随便找了几个提供在线云点播的，有弹出式广告的占大多数，于是找了一个没有广告的，okdvd，其接口调用也挺简单的

``` html
<iframe scrolling="no" frameborder="0" name="win_vod" id="win_vod" border="0" style="width: 950px; height: 550px" src="http://www.okdvd.com/api.php#!url=后面加上播放地址"></iframe>
```

所以最终修改我的play.html为
``` html

$def with(item)
<h2>$item.id $item.moviename</h2>
<blockquote>
<div align = "center">
$if item.url.startswith('qvod'):
    <object classid="clsid:F3D0D36F-23F8-4682-A195-74C92B03D4AF" width="860" height="460"
                    id="QvodPlayer" name="QvodPlayer" onerror="document.getElementById('QvodPlayer').style.display='none';document.getElementById('iframe_down').style.display='';document.getElementById('iframe_down').src='http://error2.qvod.com/error4.htm';">
                    <param name='Showcontrol' value='2'>
                    <param name='URL' value=$item.url>
                    <param name='Autoplay' value='1'>
                    <embed id="QvodPlayer2" name="QvodPlayer2" width="860" height="460" URL=$item.url type='application/qvod-plugin' Autoplay='1' Showcontrol='1' ></embed>
   </object>
$elif item.url.startswith('bdhd'):
<object id="BaiduPlayer" name="BaiduPlayer" type="application/player-activex" width="860" height="460" progid="Xbdyy.PlayCtrl.1" param_URL=$item.url param_OnPlay="onPlay" param_OnPause="onPause" param_OnFirstBufferingStart="onFirstBufferingStart" param_OnFirstBufferingEnd="onFirstBufferingEnd" param_OnPlayBufferingStart="onPlayBufferingStart" param_OnPlayBufferingEnd="onPlayBufferingEnd" param_OnComplete="onComplete" param_Autoplay="1"></object>
$else:
<iframe scrolling="no" frameborder="0" name="win_vod" id="win_vod" border="0" style="width: 860px; height: 460px" src="http://www.okdvd.com/okapi.php#!url=$item.url"></iframe>
</div>
</blockquote>
```

这里有一个问题困扰了我很长时间，原来这里的模板如果有python代码的话也是要遵循其缩进原则的，如$if 下面的<object>如果和$if同级，那么也会报错，BAE提供错误日志，可以很快的查看到哪里出错了

# 首页上播放按钮现在还都是“播放”字样
我想让它显示是百度还是qvod播放，其实方法和上面一样，判断$item.url的startswith()方法，另外我又有了新的需求，因为网上有些电影的资源实在不好找到百度或者qvod与thunder或者edk资源，但是优酷或者土豆上有，那我可以让其跳转到相应的网址上播放

所以最终的index.html修改成这样
``` html
$def with(items)
        <table>
            <tr>
                <th>排名</th>
                <th>影片名</th>
                <th>得分</th>
                <th>播放</th>
            </tr>
            $for item in items:
            <tr>
                <td id="tdstyle">$item.id</td>
                <td id="tdstyle">$item.moviename</td>
                <td id="tdstyle">$item.score</td>
                <td id="tdstyle">
                  $if item.url.startswith("http"):
                    <form action="$item.url" method="get" target="_blank">
                    <input type="submit" value="新页面播放" />
                    </form>
                  $elif item.url.startswith('bdhd'):
                    <form action="/play/$item.id" method="post">
                    <input type="submit" value="百度影音播放" />
                    </form>
                  $elif item.url.startswith('qvod'):
                    <form action="/play/$item.id" method="post">
                    <input type="submit" value="qvod播放" />
                    </form>
                  $else:
                    <form action="/play/$item.id" method="post">
                    <input type="submit" value="云点播播放" />
                    </form>
                </td>
            </tr>
        </table>
```

而最终的index.py修改成这样
``` python
#-*- coding:utf-8 -*-
import web
import model
import os
import re
import urllib

urls = (
    "/","Index",
    "/play/(d+)","Play",
    "/add/","Add",
    "/addmovie/","addmovie"
)

t_globals = {
    'datestr': web.datestr
}

app_root = os.path.dirname(__file__)
template_root = os.path.join(app_root, 'templates/')
render = web.template.render(template_root, base='base', globals=t_globals)

class Index:
    def GET(self):
        items = model.get_items()
#       return render.test(items)
        return render.index(items)

class Play:
    def POST(self, id):
        id = int(id)
        item = model.get_item(id)
        return render.play(item)

class Add:
  def POST(self):
    movieinfo = web.data()
    #model.additem(movieinfo)
    #return render.show(movieinfo)
    postlist = movieinfo.split('&')
    movie = []
    for i in postlist:
      movie.append(i.split('=')[1])
    model.additem(urllib.unquote(movie[0]),movie[1],urllib.unquote(movie[2]))
    web.seeother("/")

class addmovie:
  def GET(self):
    return render.add()

app = web.application(urls, globals()).wsgifunc()
from bae.core.wsgi import WSGIApplication
application = WSGIApplication(app)
```

# 遗留问题
现在没有权限限制，谁都可以添加，也没有修改与删除选项，所以后面我将添加相应的权限管理，另外总不能自已都去手工添加电影，考虑使用爬虫来自动添加电影

# 献给热爱的电影的同胞们
向[亚伦·斯沃茨](http://zh.wikipedia.org/zh-cn/%E4%BA%9A%E4%BC%A6%C2%B7%E6%96%AF%E6%B2%83%E8%8C%A8) 致敬

[![Aaron_Swartz_at_Boston_Wikipedia_Meetup,_2009-08-18_](/image/2013/11/Aaron_Swartz_at_Boston_Wikipedia_Meetup_20090818__thumb.jpg "Aaron_Swartz_at_Boston_Wikipedia_Meetup,_2009-08-18_")](/image/2013/11/Aaron_Swartz_at_Boston_Wikipedia_Meetup_20090818_.jpg) 

&#160; 向[汤唯](http://dianying.fm/category/key_%E6%B1%A4%E5%94%AF)致敬

[![357px-Tang_Wei2](/image/2013/11/357pxTang_Wei2_thumb.jpg "357px-Tang_Wei2")](/image/2013/11/357pxTang_Wei2.jpg)