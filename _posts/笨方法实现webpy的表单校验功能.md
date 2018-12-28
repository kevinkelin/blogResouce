title: 笨方法实现webpy的表单校验功能
id: 1340
permalink: 1340
categories:
  - Python
date: 2015-05-02 17:43:01
tags:
  - webpy
---

最近在写一个小的demo，简单的用户注册与登录，检查在注册的时候要先检测用户名是否已经存在了，还要检查一下密码长度要不得小于3个字符

检查用户是否存在我是写在了一个单独的model.py里
``` python
def check(uname):
info = db.select('pytest',where="username=$uname",vars=locals())
if len(info)>0:
return False
else:
return True
```

<!-- more -->
这里的return True与False与实际的是相反的，这里后面做校验操作的时候做说明

官方的[文档](http://webpy.org/cookbook/forms.zh-cn)中有写校验的说明，但写的比较笼统，但是有一点写到了表单的校验
``` python
vpass = form.regexp(r".{3,20}$", 'must be between 3 and 20 characters')
vemail = form.regexp(r".*@.*", "must be a valid email address")

validators = [
        form.Validator("Passwords did't match", lambda i: i.password == i.password2)]
```
vpass 是通过正则表达式来校验密码长度在3-20位，

vmail是校验邮箱格式的正确性，要有一个@符

而下面的validators则是由一个form.Validator生成的列表类

我通过这里的lambda函数想到了对表单的校验
``` python
validators=[
        form.Validator(u'用户名不能为空',lambda i:len(i.username)>0),
        form.Validator(u'用户名已经存在',lambda i:model.check(i.username)),
        form.Validator(u'两次输入的密码不一致',lambda i:i.pwd==i.pwd2),
    ]
```

这里的model.check就是调用上面写的函数，如果有，则返回False，触发'用户名已经存在'条件

另外，我看form.py里有一个变量 notnull = Validator("Required", bool)，也可以直接调用这个变量，但是它的提示文字是Required，这里可以初始化一个中文的提示
<pre class="brush:python">form.Textbox('username',form.Validator(u"必填", bool),description=u'用户名(*)')</pre>
 

最终我写的这个表单是这个样子的
<pre class="brush:python">##表单校验
vpass = form.regexp(r".{3,20}$", u'密码长度不得小于3位')

reg_form = form.Form(
    # form.Textbox('username',form.Validator(u"必填", bool),description=u'用户名(*)'),
    form.Textbox('username',description=u'用户名(*)'),
    form.Password('pwd',vpass,description=u'密码'),
    form.Password('pwd2',description=u'重复密码'),
    form.Button('submit',type='submit',html=u'注册'),
    validators=[
        form.Validator(u'用户名不能为空',lambda i:len(i.username)>0),
        form.Validator(u'用户名已经存在',lambda i:model.check(i.username)),
        form.Validator(u'两次输入的密码不一致',lambda i:i.pwd==i.pwd2),
    ]
    )</pre>
 

validators里我不知道还是否有别的调用接收表单值的方法，我尝试过
<pre class="brush:python">form.Validator(u'用户名不能为空',len(web.input().get('username'))>0)

</pre>
但是这个在模板form.render()中就会报错，所以现在还是先以这种笨方式简单的调用表单校验吧！