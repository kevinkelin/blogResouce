title: (转)Python 参数知识（变量前加星号的意义）
id: 775
permalink: 775
donate: false
categories:
  - Python
date: 2013-03-16 20:24:01
tags:
  - python
---

原文地址：http://blog.csdn.net/qinyilang/article/details/5484415

**过量的参数

**在运行时知道一个函数有什么参数，通常是不可能的。另一个情况是一个函数能操作很多对象。更有甚者，调用自身的函数变成一种api提供给可用的应用。

对于这些情况，python提供了两种特别的方法来定义函数的参数，允许函数接受过量的参数，不用显式声明参数。这些“额外”的参数下一步再解释。

注意args和kwargs只是python的约定。任何函数参数，你可以自己喜欢的方式命名，但是最好和python标准的惯用法一致，以便你的代码，其他的程序员也能轻松读懂。
**
位置参数**

在参数名之前使用一个星号，就是让函数接受任意多的位置参数。

``` 
>>> def multiply(*args):
...     total = 1
...     for arg in args:
...         total *= arg
...     return total
...
>>> multiply(2, 3)
6
>>> multiply(2, 3, 4, 5, 6)
720
```

<!-- more -->
python把参数收集到一个元组中，作为变量args。显式声明的参数之外如果没有位置参数，这个参数就作为一个空元组。

**关键字参数

**python在参数名之前使用2个星号来支持任意多的关键字参数。

```
>>> def accept(**kwargs):
...     for keyword, value in kwargs.items():
...         print "%s => %r" % (keyword, value)
...
>>> accept(foo='bar', spam='eggs')
foo => 'bar'
spam => 'eggs'
```

注意：kwargs是一个正常的python字典类型，包含参数名和值。如果没有更多的关键字参数，kwargs就是一个空字典。

**混合参数类型**

任意的位置参数和关键字参数可以和其他标准的参数声明一起使用。混合使用时要加些小心，因为python中他们的次序是重要的。参数归为4类，不是所有的类别都需要。他们必须按下面的次序定义，不用的可以跳过。

1）必须的参数
2）可选的参数
3）过量的位置参数
4）过量的关键字参数

def complex_function(a, b=None, *c, **d):

这个次序是必须的，因为*args和**kwargs只接受那些没有放进来的其他任何参数。没有这个次序，当你调用一个带有位置参数的函数，python就不知道哪个值是已声明参数想要的，也不知道哪个被作为过量参数对待。

也要注意的是，当函数能接受许多必须的参数和可选的参数，那它只要定义一个过量的参数类型即可。

传递参数集合

除了函数能接受任意参数集合，python代码也可以调用带有任意多数量的函数，像前面说过的用星号。这种方式传递的参数由python扩展成为参数列表。以便被调用的函数
不需要为了这样调用而去使用过量参数。python中任何可调用的，都能用这种技法来调用。并且用相同的次序规则和标准参数一起使用。
```
>>> def add(a, b, c):
...     return a + b + c
...
>>> add(1, 2, 3)
6
>>> add(a=4, b=5, c=6)
15
>>> args = (2, 3)
>>> add(1, *args)
6
>>> kwargs={'b': 8, 'c': 9}
>>> add(a=7, **kwargs)
24
>>> add(a=7, *args)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: add() got multiple values for keyword argument 'a'
>>> add(1, 2, a=7)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: add() got multiple values for keyword argument 'a'

```