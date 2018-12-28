title: python中的logging模块应用
id: 963
permalink: 963
categories:
  - Python
date: 2013-11-16 23:56:49
tags:
  - python
---

python中有系统自带的logging模块，使用起来非常方便，并且要在程序中经常要使用这个模块，这样出了问题可以通过日志很方便的查找在哪里出了问题，比直接找代码要方便些

我记录了一些经常用到的，不是很常用的可以到python文档中去查看 [http://docs.python.org/2/library/logging.html](http://docs.python.org/2/library/logging.html)
<!-- more -->
1\. 初始化 logger = logging.getLogger("endlesscode")，getLogger()方法后面最好加上所要日志记录的模块名字，后面的日志格式中的%(name)s 对应的是这里的模块名字

2\. 设置级别 logger.setLevel(logging.DEBUG),Logging中有NOTSET < DEBUG < INFO < WARNING < ERROR < CRITICAL这几种级别，日志会记录设置级别以上的日志

3\. Handler，常用的是StreamHandler和FileHandler，windows下你可以简单理解为一个是console和文件日志，一个打印在CMD窗口上，一个记录在一个文件上

4\. formatter，定义了最终log信息的顺序,结构和内容，我喜欢用这样的格式 '[%(asctime)s] [%(levelname)s] %(message)s', '%Y-%m-%d %H:%M:%S'，

**%(name)s Logger的名字**

**%(levelname)s 文本形式的日志级别**

**%(message)s 用户输出的消息**

**%(asctime)s 字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒**

%(levelno)s 数字形式的日志级别

%(pathname)s 调用日志输出函数的模块的完整路径名，可能没有

%(filename)s 调用日志输出函数的模块的文件名

%(module)s&#160; 调用日志输出函数的模块名

%(funcName)s 调用日志输出函数的函数名

%(lineno)d 调用日志输出函数的语句所在的代码行

%(created)f 当前时间，用UNIX标准的表示时间的浮 点数表示

%(relativeCreated)d 输出日志信息时的，自Logger创建以 来的毫秒数

%(thread)d 线程ID。可能没有

%(threadName)s 线程名。可能没有

%(process)d 进程ID。可能没有

5\. 记录 使用object.debug(message)来记录日志

下面来写一个实例，在CMD窗口上只打出error以上级别的日志，但是在日志中打出debug以上的信息
``` python
import logging
logger = logging.getLogger("simple_example")
logger.setLevel(logging.DEBUG)
# 建立一个filehandler来把日志记录在文件里，级别为debug以上
fh = logging.FileHandler("spam.log")
fh.setLevel(logging.DEBUG)
# 建立一个streamhandler来把日志打在CMD窗口上，级别为error以上
ch = logging.StreamHandler()
ch.setLevel(logging.ERROR)
# 设置日志格式
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
ch.setFormatter(formatter)
fh.setFormatter(formatter)
#将相应的handler添加在logger对象中
logger.addHandler(ch)
logger.addHandler(fh)
# 开始打日志
logger.debug("debug message")
logger.info("info message")
logger.warn("warn message")
logger.error("error message")
logger.critical("critical message")

```

运行一下将会看到CMD窗口只记录两条，spam.log中记录了五条日志

[![log](/image/2013/11/log_thumb.png "log")](/image/2013/11/log.png) 

&#160;

当一个项目比较大的时候，不同的文件中都要用到Log,可以考虑将其封装为一个类来使用
``` python
#! /usr/bin/env python
#coding=gbk
import logging,os

class Logger:
    def __init__(self, path,clevel = logging.DEBUG,Flevel = logging.DEBUG):
        self.logger = logging.getLogger(path)
        self.logger.setLevel(logging.DEBUG)
        fmt = logging.Formatter('[%(asctime)s] [%(levelname)s] %(message)s', '%Y-%m-%d %H:%M:%S')
        #设置CMD日志
        sh = logging.StreamHandler()
        sh.setFormatter(fmt)
        sh.setLevel(clevel)
        #设置文件日志
        fh = logging.FileHandler(path)
        fh.setFormatter(fmt)
        fh.setLevel(Flevel)
        self.logger.addHandler(sh)
        self.logger.addHandler(fh)

    def debug(self,message):
        self.logger.debug(message)

    def info(self,message):
        self.logger.info(message)

    def war(self,message):
        self.logger.warn(message)

    def error(self,message):
        self.logger.error(message)

    def cri(self,message):
        self.logger.critical(message)



if __name__ =='__main__':
    logyyx = Logger('yyx.log',logging.ERROR,logging.DEBUG)
    logyyx.debug('一个debug信息')
    logyyx.info('一个info信息')
    logyyx.war('一个warning信息')
    logyyx.error('一个error信息')
    logyyx.cri('一个致命critical信息')


```

这样每次使用的时候只要实例化一个对象就可以了

logobj = Logger(‘filename’,clevel,Flevel)

如果想在CMD窗口中对于error的日志标红，warning的日志标黄，那么可以使用ctypes模块

[![color](/image/2013/11/color_thumb.png "color")](/image/2013/11/color.png) 

``` python
#! /usr/bin/env python
#coding=gbk
import logging,os
import ctypes

FOREGROUND_WHITE = 0x0007
FOREGROUND_BLUE = 0x01 # text color contains blue.
FOREGROUND_GREEN= 0x02 # text color contains green.
FOREGROUND_RED  = 0x04 # text color contains red.
FOREGROUND_YELLOW = FOREGROUND_RED | FOREGROUND_GREEN

STD_OUTPUT_HANDLE= -11
std_out_handle = ctypes.windll.kernel32.GetStdHandle(STD_OUTPUT_HANDLE)
def set_color(color, handle=std_out_handle):
    bool = ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
    return bool


class Logger:
    def __init__(self, path,clevel = logging.DEBUG,Flevel = logging.DEBUG):
        self.logger = logging.getLogger(path)
        self.logger.setLevel(logging.DEBUG)
        fmt = logging.Formatter('[%(asctime)s] [%(levelname)s] %(message)s', '%Y-%m-%d %H:%M:%S')
        #设置CMD日志
        sh = logging.StreamHandler()
        sh.setFormatter(fmt)
        sh.setLevel(clevel)
        #设置文件日志
        fh = logging.FileHandler(path)
        fh.setFormatter(fmt)
        fh.setLevel(Flevel)
        self.logger.addHandler(sh)
        self.logger.addHandler(fh)

    def debug(self,message):
        self.logger.debug(message)

    def info(self,message):
        self.logger.info(message)

    def war(self,message,color=FOREGROUND_YELLOW):
        set_color(color)
        self.logger.warn(message)
        set_color(FOREGROUND_WHITE)

    def error(self,message,color=FOREGROUND_RED):
        set_color(color)
        self.logger.error(message)
        set_color(FOREGROUND_WHITE)

    def cri(self,message):
        self.logger.critical(message)



if __name__ =='__main__':
    logyyx = Logger('yyx.log',logging.WARNING,logging.DEBUG)
    logyyx.debug('一个debug信息')
    logyyx.info('一个info信息')
    logyyx.war('一个warning信息')
    logyyx.error('一个error信息')
    logyyx.cri('一个致命critical信息')

```
