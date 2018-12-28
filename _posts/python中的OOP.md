title: python中的OOP
id: 813
permalink: 813
categories:
  - Python
date: 2013-06-01 22:17:55
tags:
  - python
---

直接上代码，已经在后面有注释了
``` python
#coding:utf8

name = 'yangyanxing'
class Test():
    class kevin():
        var1 = '我是内部类'

    name = 'kvein'
    gae = '26'

    def fun1(self):
        print self.name
        print '我是公共方法'
        self.__fun2() #可以通过公有就去来调用私有方法，在调用的过程中可以进行更改

    def __fun2(self):
        print '我是私有方法'

    @classmethod
    def fun3(self): #可以不通过实例来访问这个类方法
        print '#'*40
        print self.name
        print '我是类方法'

    @staticmethod #静态方法，也是可以不通过实例对象就可以访问的方法但是在定义的时候不用加self
    def fun4():
        print Test.name
        print name #这里的name是全局变量
        Test.fun3()
        print '我是静态方法'

print Test.name #公有属性可以直接方法，不用实例化对象
yang = Test() #实例化一个类
interyang = Test.kevin() #实例化一个内部类
yang.fun1() #方法类里面的公共属性
print interyang.var1 # 访问内部类里的属性
Test.fun3()#访问类方法
Test.fun4()
```
``` python
#coding:utf8

class Test():
    var1 = '类的公有属性'
    __var2 = '类的私有属性'

    def fun(self):
        self.var2 = '对象的公有属性' # 这里定义了一个对象的公有属性
        self.__var3 = '对象的私有属性'# 这里定义了一个对象的私有属性
        var4 = '函数的局部变量' #这里定义了一个函数的局部变量，这里面的var4只有在函数内部使用

kevin = Test() #实例了一个对象
yang = Test() #又实例了另外一个对象
print kevin.var1
##print kevin.__var2 #这里将无法访问
kevin.fun()
print kevin.var2 #在没有调用fun函数之前是没有var2的
##print kevin.__var3 对象的私有属性是无法调用的
##print yang.var2 #这里因为没有调用yang的fun方法，所以还是无法访问yang里的var2
```