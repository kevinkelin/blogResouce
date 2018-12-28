title: web.py中关于url的总结
id: 932
permalink: 932
categories:
  - Python
date: 2013-10-22 01:42:41
tags: 
  - webpy
  - python
---

网页中的数据在传递的时候有GET和POST两种方式，GET是以网址的形式传参数，在web.py中有着很好的匹配，如果我们配置以下的urls
``` python
  urls =(
    '/','index',
    '/weixin/(.*?)','WeixinInterface'

```

先不考虑/weixin/后面的东西，现在我们来写index的类
``` python
class index:
    def GET(self):
        i = web.input(name = 'kevinkelin',age = 100)
```
<!-- more -->
随便写一个index.html模板文件
``` html
$def with(name,age)

$if name:
I just want to say <em>hello</em> to $name, he is $age years old
$else:
<em>hello</em>,world!
```

当访问[http://127.0.0.1:8080/](http://127.0.0.1:8080/) 此时没有传递name与age的值，由于我的GET函数里定义了默认的name与age的值，所以程序会将kevinkelin与26传递到模板中去得到以下的输出

I just want to say _hello_ to kevinkelin, he is 100 years old

当访问[http://127.0.0.1:8080/?name=yyx&age=26](http://127.0.0.1:8080/?name=yyx&age=26) 即向GET函数中传递name = yyx and age = 26的时候得到以下的输出

I just want to say _hello_ to yyx, he is 26 years old

&#160;

我们也可以不定义默认的的参数，即定义为空

i = web.input(name = None,age = None)

当访问[http://127.0.0.1:8080/](http://127.0.0.1:8080/) 的时候将会得到 _hello_,world!的输出即模板中的else

但是如果你不定义name和age将会出错

i = web.input()

这是因为后面你将i.name与i.age分配到模板当中去，但是全局变量里又没有这两个变量，所以会报错

但有时我们想这样传递参数，不想加那个“？”这时我们得要更改urls规则
``` python
urls =(
    '/name=(.*)&age=(.*)','index',
    '/weixin/(.*?)','WeixinInterface'
```

重新写class index
``` python
class index:
    def GET(self,name,age):
```

这里是将url的参数通过正则匹配然后传递到index类中的GET的参数中

当访问[http://127.0.0.1:8080/name=yyx&age=26](http://127.0.0.1:8080/name=yyx&age=26) 时将得到

I just want to say _hello_ to yyx, he is 26 years old

第二种方法看似简单，但其实不好控制，要求写的正则工作量加大了

如果我想知道到底有多少参数通过GET方式传递过来，我可以直接return 来看一下到底有哪些传递过来了

&#160;

接下来看一下post来的数据

我们可以制作一个简单的表单或者直接使用fiddler来构造数据进行POST传值
``` python
def POST(self):
        data = web.data()
```


&#160;

我想看一下得到的数据类型

return type(data)

得到的是<type 'str'>，也就是说web.py已经将post的数据转换成了str类型

那么我来试一下传递xml
``` xml

<xml>
<ToUserName>yanxingyang</ToUserName>
<FromUserName>study_python</FromUserName>
<CreateTime>123456</CreateTime>
<MsgType>text</MsgType>
<Content>Just a test</Content>
</xml>
```

其实这个微信的XML格式做了一些更改，我来试着使用lxml对它进行解析

from lxml import etree 

data = web.data()

xml = etree.fromstring(data)

content = xml.find(‘Content’).text

return content

得到的结果很好



关于python解析XML以前也有总结过（[http://my.oschina.net/yangyanxing/blog/159212](http://my.oschina.net/yangyanxing/blog/159212)），在我的脑子里总感觉解析XML是件头痛的事情……

今天先总结到这，明天接着搞LXML进行解析xml操作。