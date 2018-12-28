title: webpy的常规使用与踩过的坑
comment: true
donate: true
categories:
  - Python
date: 2016-12-21 21:42:13
tags: 
  - python
  - webpy
permalink: use-webpy

---
在使用python做网站的时候首先接触的就是webpy了，这个框架小巧简单，对于小型的网络应用功能足以了，在使用的过程中也遇到过一些总是，在这里也记录总结一下


# 基础的框架搭建

### 安装

```
pip install web.py -i https://pypi.douban.com/simple
```

<!-- more -->

### hello world

``` python
import web

urls = ("/.*", "hello")
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, world!'

if __name__ == "__main__":
    app.run()

```

return 可以返回字符串也可以返回字典，不过直接返回字符串不是一个很好的网页习惯，如果是对外提供的api是很好的，但是如果是做的网页，最好还是将数据render（渲染）到模板中

### 使用GET或者POST接收数据

`web.input()` 既可以接收GET的数据，也可以接收POST过来的数据

`web.data()` 通过这个方法可以取到数据

``` python
import web

urls = ("/.*", "hello")
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, world!'
    def POST(self):
        username = web.input().get('username','')

if __name__ == "__main__":
    app.run()

```

### 模板的定义使用

``` python
import web

urls = ("/.*", "hello")
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, world!'
        
app_root = os.path.dirname(__file__) #定义根路径
templates_root = os.path.join(app_root, 'templates') #定义模板所在的路径
render = web.template.render(templates_root)

if __name__ == "__main__":
    app.run()

```

使用的时候用return render
如： return rende.index() 则需要在templates目录下创建一个index.html 这样访问

### 模板中使用渲染的数据
当有以下代码的时候
``` python
name = 'yangyanxing'
return render.index(name)
```

此时可以在index.html中使用渲染过来的name值

``` html
$def with(name_t)

<html>
  <body>
  hello $name_t
  </body>
</html>
```

$def with 必须写在首行，with里面有多少个参数，那个在之前的py文件中就需要render多少个参数，多了少了都不行，with里可以使用默认参数，这个和一般的函数定义是一样的。

# 中级使用

### session的使用

- session 可以使用文件的形式也可以存储在数据库中

``` python 
import web

urls = ("/.*", "hello")
app = web.application(urls, globals())
store = web.session.DiskStore('sessions') #定义session使用文件存储方式

#定义session格式，使用本地存储，还可以使用数据库存储方式
if web.config.get('_session') is None:
    session = web.session.Session(app, store,)
    web.config._session = session
else:
    session = web.config._session

class hello:
    def GET(self):
        return 'Hello, world!'
        
app_root = os.path.dirname(__file__) #定义根路径
templates_root = os.path.join(app_root, 'templates') #定义模板所在的路径
render = web.template.render(templates_root,globals={'session': session}) #将session应用到模板中

if __name__ == "__main__":
    app.run()

```

- session的赋值

``` python
username = 'yangyanxing'
session.realname = username
```
- session的读取
  > print session.realname

- session在模板中的使用
``` html
$def with (something)

$session.realname

```

上面在定义
> render = web.template.render(templates_root,globals={'session': session})

时`globals={'session': session}` 这里如果定义成`globals={'session_t': session}` 那么在模板中使用session中也要使用`$session_t

### 模板布局的使用

有时候对于一个小型网站，它的header与footer是一样的，只是中间的内容不一样，那么可以定义一个模板，base.html包括了header与footer，其它模板在定义的时候只要引用这个模板就行了，这样做的好处也在于当修改base.html，不用修改每一个模板。
首先定义一个基础的模板，我这里就叫做base.html
``` html
$def with (content)

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name=”renderer” content=”webkit” />
<title>欢迎来到我们的世界</title>

</head>
</head>
<body>
    <div class="header">
		something here
	</div>
	<div class="container">
    	$:content
    	<footer>
    	    <p>&copy; yangyanxing.com 2016</p>
    	</footer>
	</div>
    
</body>
</html>

```

这里有两个要点 `$def with (content)` 这里的content可以随意写，而下面的`$:content` 则是别的模板文件内容，可以这样理解，当你再使用`return render.index(name)` 时，其实它是将index.html里的内容当成这里的`content` 这样也就实现了一个基础的模板，只更新里面的内容的效果。

定义完基础模板还不行，如果要使其生效还得在py文件中修改以下定义render的代码
`render = web.template.render(templates_root,globals={'session': session},base='base')`
这样定义render才能将base.html设置为基础模板

此时如果index.html中是以下的样子
``` html
$def with (name)
hello $name
```

那么当你有以下代码的时候
``` python
myname = 'yangyanxing'
return render.index(myname)

```

那么最终生成的html应该是
``` html 
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name=”renderer” content=”webkit” />
<title>欢迎来到我们的世界</title>

</head>
</head>
<body>
    <div class="header">
		something here
	</div>
	<div class="container">
    	hello yangyanxing
    	<footer>
    	    <p>&copy; yangyanxing.com 2016</p>
    	</footer>
	</div>
    
</body>
</html>
```

# 高级使用(踩过的坑)

### checkbox传过来多个值的时候数据的处理

有这种情况，有一个表单它的name值下面是checkbox，如以下的html
``` html 
<form action='/groupaddcase' method='POST'>
<input type="checkbox" name="caseids"  value="1" checked="true">1
<input type="checkbox" name="caseids"  value="2" >2
<input type="checkbox" name="caseids"  value="3" >3
<input type="checkbox" name="caseids"  value="4" >4
<input type="checkbox" name="caseids"  value="5" >5

<input class="btn btn-success" type="submit" value="修改">
</form>

```
此时如果在py中处理如下
``` python 
class Groupaddcase:
    def POST(self):
        caseids = web.input().get('caseids')
        print caseids

```

这样处理的话那么传过来的caseids只有一个，当你勾选了多个checkbox时也只有一个数据传过来，解决的办法是在接收的时候将其定义成数组
``` python 
class Groupaddcase:
    def POST(self):
        groupinfo  =  web.input(caseids=[])
        groupid = groupinfo.get('groupid','')

```

这样groupid就以数组的形式保存了传过来的checkbox的值,多个checkbox可以定义多个数组
`groupinfo  =  web.input(caseids=[],casenames=[])`


### 添加一个hook来处理在所有请求前做的处理

有时会有这样的需求，需要屏蔽某一个IP的客户端访问或者在访问前需要检查一下是否登录了，这种情况下可以使用hook方法来控制
``` python 
def __redirect():
    rejectIP = ['192.168.0.101']
    if web.ctx.ip in rejectIP:
        raise web.seeother(r'http://so.com')
        
if __name__ == "__main__":
    app.add_processor(web.loadhook(__redirect))
    app.run()  

```

### 模板中使用python的逻辑处理方法

- 判断
``` python
$if condition:
    <td>yes</td>
$else:
    <td>no</td>
```
- 循环
``` python 
$for item in some_array:
    <td>$item['title']</td>
```

判断和循环的使用都要在前面加一个$,以`:` 结尾，且下面的语句也要遵守缩进规则