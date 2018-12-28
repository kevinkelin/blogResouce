title: python中的异常处理
comment: true
categories:
  - Python
date: 2017-08-19 23:53:38
tags: python
permalink: try_in_python

---
写程序肯定要接触异常处理，还好python中的异常处理比较简单，但是有些细节还是需要好好看一下
这里记录一下try在python中的使用。

<!-- more -->
#### 最简单的，try...except
先看一下以下的代码执行情况
``` python
def test():
    print 'yangyanxing'
    print 5/0
    print 'end script....'

if __name__ == '__main__':
    test()

```

程序运行到 print 'yangyanxing' 时，并没有出错，可以正常的输出
但是当走到5/0的时候，会触发除0的错误，我们都知道，0是不能做为除数的，所以程序会在此处崩溃，
下面的 `print 'end script... '` 也就不会执行了
执行程序得到以下的输出
```
yangyanxing
Traceback (most recent call last):
  File "try.py", line 11, in <module>
    test()
  File "try.py", line 7, in test
    print 5/0
ZeroDivisionError: integer division or modulo by zero

```
如何让程序可以正常的运行呢？很简单，加个try呗，把可能出现异常的地方进行捕获

``` python
def test():
    print 'yangyanxing'
    try:
        print 5/0
    except Exception as e:
        print e
    
    print 'end script....'

if __name__ == '__main__':
    test()

```

将得到以下的输出
```
yangyanxing
integer division or modulo by zero
end script....
```

这样，当程序运行到出错的那行代码时，将执行except处定义代码，即 `print e`，然后程序接着就执行`print 'end script....'`

可是，你不可能在执行每一步操作的时候都对其加上单独的try语句，那样程序看上去会很诡异，有时你也不知道会出什么错误，所以try的范围可以加大一些
``` python
def test():
    try:
        print 'yangyanxing'
        print 5/0
        print 'end script....'
    except Exception as e:
        print e

if __name__ == '__main__':
    test()

```

这时得到的输出是
```
yangyanxing
integer division or modulo by zero
[Finished in 0.0s]

```

嗯，很简单是不是，你也不知道是哪行出的错误？你是不是想要得到更加详细的信息？
python标准库已经为你准备好了，可以使用traceback来打印更加详细的异常信息

``` python

#coding:utf-8

import traceback

def test():
    try:
        print 'yangyanxing'
        print 5/0
        print 'end script....'
    except Exception as e:
        print e
        print 1111
        print traceback.format_exc()

if __name__ == '__main__':
    test()

```

它的输出结果为
```
yangyanxing
integer division or modulo by zero
1111
Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 8, in test
    print 5/0
ZeroDivisionError: integer division or modulo by zero

```

请看1111下面的那个trackback信息,这里标记的错误在哪一行，什么错误，一定要善用trackback

#### try...except...finally

这个比之前那个try...except多了一个finally，功能上也强大一些，这个finally不是必须的，使用的时候也根据实际情况自己来定。
而且加了finally以后也有很多注意的地方。
finally的意思是，无论程序是否有异常，最后都要执行它的代码。

看这样的例子
``` python 
#coding:utf-8

import traceback

def test():
    try:
        print 'yangyanxing'
        print 5/0
        print 'hello world'
    except Exception as e:
        print traceback.format_exc()
    finally:
        print 'end script....'

if __name__ == '__main__':
    test()

```

得到的输出为
```
yangyanxing
Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 8, in test
    print 5/0
ZeroDivisionError: integer division or modulo by zero

end script....

```

注意，这里`print 'end script....'`被执行了，程序在print 5/0的时候发生的错误，在此中断，开始执行except代码，执行完except代码以后，就会执行finally代码，而出异常处后面的代码就不会被执行了

关于返回值的问题，看以下的代码 

``` python
#coding:utf-8

import traceback

def test():
    try:
        if 2>1:
            return 1
        else:
            return 0

    except Exception as e:
        print traceback.format_exc()
        return 2
    finally:
        print 'end script....'
        return 3

if __name__ == '__main__':
    print test()

```

它的返回值是什么呢？0？1？2？3？

看一下程序运行的结果
```
end script....
3
```
结果是3

也就是说，即使在try里定义了返回值，也是没有用的，程序最终返回的是finally中执行的返回值，所以如果使用finally的话，那么在try与except中最好不要定义返回值，可以将其保存到变量中，然后在finally中返回.
``` python
#coding:utf-8

import traceback

def test():
    try:
        if 2>1:
            result = 1
        else:
            result =  0

    except Exception as e:
        print traceback.format_exc()
        result =  2
    finally:
        print 'end script....'
        return result

if __name__ == '__main__':
    print test()

```
这样程序返回值就是1了

#### 函数中异常的嵌套使用

这里不是try 里再加了个try，而是函数间的嵌套，看如下代码
``` python
#coding:utf-8

import traceback

def test():
    print 'start test...'
    test2()
    print 'end script...'

def test2():
    try:
        print 'start test2...'
        print 5/0
        print 'end test2...'
    except Exception as e:
        print traceback.format_exc()

if __name__ == '__main__':
    test()


```

它会有什么结果呢？

```
start test...
start test2...
Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 13, in test2
    print 5/0
ZeroDivisionError: integer division or modulo by zero

end script...
```
注意，这里在test2()函数中，已经将除0错误给逮到了，并且做了处理，所以在test函数中，不会受到什么影响，end script也可以正常的运行。
但是，对于某些复杂的程序，当test2函数中发生异常的时候，调用它的函数以接下来的操作中得是在test2运行正常才会执行正常，这种就比较尴尬。
怎么处理呢？

答案是在test2中使用raise将这个异常抛出，让调用者知道，它出错了。
``` python 

#coding:utf-8

import traceback

def test():
    try:
        print 'start test...'
        test2()
        print 'end script...'
    except Exception as e:
        print traceback.format_exc()

def test2():
    try:
        print 'start test2...'
        print 5/0
        print 'end test2...'
    except Exception as e:
        print traceback.format_exc()
        raise e

if __name__ == '__main__':
    test()
```

得到的输出为
``` 
start test...
start test2...
Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 16, in test2
    print 5/0
ZeroDivisionError: integer division or modulo by zero

Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 8, in test
    test2()
  File "/Users/yangyanxing/Desktop/try.py", line 20, in test2
    raise e
ZeroDivisionError: integer division or modulo by zero

```

上面的输出为test2()中打印的，下面的是test()函数打印的，这里test()函数中的`print 'end script...'`因为抓到了异常而没有执行。

这里也是根据业务流程来灵活的使用raise，有时某个异常会导致大流程失败的时候则要将其抛出，比如最近在做的一个项目，虚拟机启动并且执行相应的测试代码，有时经常会遇到虚拟机没有启动成功，但是程序还是会执行接下来的代码，检查原因就是虚拟机的会滚操作，它抓了异常，但是并没有抛出，然后上层在调用的时候就没有判断它的会滚是否成功，这样就会造成程序一直错误的执行。

#### 自定义异常

有时候系统定义的这些异常不能满足需求时就需要自己定义一些异常，这个用的也比较多，
比如要设计以下的场景，当循环到5次的时候，抛出一个'老子不干了，太累了'的异常
``` python 

#coding:utf-8

import traceback
import time
def test():
    try:
        print 'start test...'
        test2()
        print 'end script...'
    except Exception as e:
        print traceback.format_exc()

def test2():
    try:
        times = 10
        while times > 0:
            print times
            if times == 5:
                raise TypeError('i don\'t want to work....')
            times -=1
            time.sleep(1)
    except Exception as e:
        print traceback.format_exc()
        raise e

if __name__ == '__main__':
    test()


```

得到以下输出 

```
Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 19, in test2
    raise TypeError('i don\'t want to work....')
TypeError: i don't want to work....

Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 8, in test
    test2()
  File "/Users/yangyanxing/Desktop/try.py", line 24, in test2
    raise e
TypeError: i don't want to work....

```


其实这里还不是自定义的异常，这里只是定义了异常信息，异常的类型还是TypeError，自定义的异常是写一个类继承自Exception
``` python 
#coding:utf-8

import traceback
import time

class IdontWantWork(Exception):
    def __init__(self,err=''):
        Exception.__init__(self,err)


def test2():
    try:
        times = 10
        while times > 0:
            print times
            if times == 5:
                raise IdontWantWork('老子不干了，太累了')
            times -=1
            time.sleep(1)
    except Exception as e:
        print traceback.format_exc()

if __name__ == '__main__':
    test2()

```

输出如下

```
10
9
8
7
6
5
Traceback (most recent call last):
  File "/Users/yangyanxing/Desktop/try.py", line 25, in test2
    raise IdontWantWork('老子不干了，太累了')
IdontWantWork: 老子不干了，太累了
```

简单的异常就记录这些，以后再补充。

参考文章
[python 自定义异常和异常捕捉](http://blog.csdn.net/flyingshuai/article/details/73482177)
[Python中的异常处理](http://www.cnblogs.com/jessonluo/p/4743574.html)







