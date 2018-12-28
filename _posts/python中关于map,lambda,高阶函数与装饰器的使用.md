title: python中关于map,lambda,高阶函数与装饰器的使用
comment: true
categories:
  - Python
date: 2018-07-01 23:53:38
tags: python
permalink: higher-order-function

---

有一点python基础的朋友在看到一些老鸟写的一些函数的时候，他们会有意的写出一些高阶函数，要么带个map啊，要么带个lambda呀，再高一点的带个@装饰器啊，其它这些也没有什么难的，用法上高级一些，如果能捋顺它的逻辑，准确的在实际应用中使用，还是会给程序效率上带来一定的提高，至于可读性，当你习惯了这些用法时你就会觉得它的可读性也很好，之所以你不习惯，主要还是由于你不懂它的用法，说到可读性，这是另外的话题，以后再详细的聊聊，今天要说的主要是高阶函数与装饰器的使用。希望看完此篇文章的朋友有所收获。

<!-- more -->

# 高阶函数

高阶函数的定义：一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。
常规函数的参数一般是个变量或者是个对象，比如定义一个求和函数
``` python 
# coding:utf-8

def sum(x,y):
    return x + y

if __name__ == '__main__':
    print sum(3,4)

```

这里传入的参数，x,y 都是整数，返回的是它们的和。
变量可否是一个函数呢？

``` python 
>>> s = sum
>>> s
<function sum at 0x01F96B70>
>>> s.__name__
'sum'
>>> s(6,7)
13
```

把sum()函数 赋给变量s，s现在为一个函数，s的__name__为'sum',
可以调用s(6,7) 返回的就是sum(6,7)。

上面很简单，再来看一下高阶函数的定义，`一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数`,
既然函数也可以作为另外的一个函数的参数，那么我们再来定义另外一个函数
``` python 
# coding:utf-8

def sum(x,y):
    return x + y

def sum2(func,x,y):
    return func(x,y)

if __name__ == '__main__':
    print sum2(sum,5,6)

```

这里定义一个sum2函数，它的返回值是func(x,y)。

# 函数作为返回值

函数可以作为参数，也可以作为返回值
``` python 
def sum3(x,y):
    def sum4():
        return x+y
    return sum4

f3 =  sum3(1,2)
print f3
print f3()

```

`sum3` 函数的返回值是`sum4`,但直接打印f3时显示的是`<function sum4 at 0x02142DB0>`,要执行这个函数需要调用执行一个f3函数`f3()`

# map函数

上面的代码似乎没有什么具体的应用，只要为了说明，函数也可以作为参数传入另外的函数中，下面来看一下map函数
map()函数接收两个参数，一个是函数，一个是Iterable，map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回。
最简单的应用，给一个列表，返回该列表中每个元素的平方。
``` python 
def f(x):
    return x*x

l = range(1,10)
l2 = map(f,l)
print type(l2)
print l2
```

在python2中map返回的是一个list,在python3中返回的是一个map对象，如果要得到新的列表，需要执行list(l2)操作，以下代码都是在python2环境下操作。

# lambda 函数

说叫函数，其实它是一种没有名字的函数，一次性使用，适用于那种简单的表达式可以不用定义一个函数，上面的求和与求立方。
``` python 
l3 = map(lambda x:x*x,[1,2,3,4])
print l3

l4 = map(lambda x,y:x+y,[1,2,3,4],[5,6,7,8])
print l4

fl =  lambda x:x*3
print fl(10)

```
注意到lambda用在map函数中时，如果它的参数需要两个或者更多，那么在map的参数里也要传入相应个数的列表，且它的计算是列表中对应位置的表达式，比如求和，它是将列表1中的第一个元素和列表中第一个元素求和放到返回列表中的第一荐，列表的长度要一致，不能第一个列表有4个元素，第二个列表有3个或者5个，第二个列表也只能是4个。

# 装饰器

有了上面的一些基础，接下来咱们看看本文的重点----装饰器
首先看一个简单函数的定义，打印当时日期时间

``` python
from datetime import datetime

def showTime():
    print datetime.now()


showTime()
```

现在，假设我们要增强`showTime`函数的功能，比如，在函数调用前后自动打印日志，但又不希望修改`showTime`函数的定义，这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。

定义一个函数
``` python 
def log(func):
    def wrapper(*args, **kw):
        print 'call %s():' % func.__name__
        return func(*args, **kw)
    return wrapper

```
观察这个log函数，它的参数是一个函数，要执行传入参数的函数时，将其包了一层，做了一些别的事情，然后又将函数返回，`return wrapper` 所以本质上这个log就是一个返回函数的高阶函数。
参考上面定义sum2函数,再来回看一下上面说明的函数作为返回值，其实这里它已经执行了func函数，对于上面的代码就是`print datetime.now()`
装饰器的使用是在函数的定义处上面加上@符
``` python 
def log(func):
    def wrapper(*args, **kw):
        print 'call %s():' % func.__name__
        return func(*args, **kw)
    return wrapper

@log
def showTime(a,b):
    print a
    print datetime.now()
    print b

showTime('yang','fan')
```
加了装饰器以后，我们把调用showTime函数分解一下，它实际上是执行了`log(showTime("yang","fan"))`

如果decorator本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，要自定义log的文本：
``` python 
def log2(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator
    
@log2("yyx")
def showTime(a,b):
    print a
    print datetime.now()
    print b

showTime('yang','fan')
    
 ```
 
上面的执行是这样的：
首先执行`log2('yyx')`返回的是decorator函数，再调用返回的函数(decorator)，参数是showTime函数，返回值最终是wrapper函数,而wrapper函数将会执行showTime函数,所以在经过了log2装饰以后的调用分解开来应该是log2("yyx")(showTime("yang","fan"))。

所以在写一个装饰器的时候，要区别装饰器本身是否需要有参数，如果没有，包一层即可，有参数的话需要包两层。


# 参考文章
[廖雪峰-函数式编程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014317848428125ae6aa24068b4c50a7e71501ab275d52000)

 






