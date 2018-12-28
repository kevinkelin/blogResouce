title: python中的日志按天保存单独的文件
comment: true
categories:
  - Python
date: 2018-06-24 23:53:38
tags: python
permalink: python-logging-day-file

---

在运行web的python程序时,日志最好是使用按天的保存,这样查看起来会很方便,也不至于日志文件太大不好打开
python 自带的logging模块有着非常强大的作用

<!-- more -->

``` python 

import logging
import time,sys
from logging.handlers import TimedRotatingFileHandler


log = logging.getLogger('yyx')
log.setLevel(logging.DEBUG)
formatter = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")

log_file_handler = TimedRotatingFileHandler(filename="log", when="S", interval=2)
log_file_handler.setFormatter(formatter)
log_file_handler.setLevel(logging.DEBUG)
log.addHandler(log_file_handler)

stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.setFormatter(formatter)
log.addHandler(stream_handler)

log.debug('111')
log.error('11error')
log.info('11info')
time.sleep(2)
log.error('222')
time.sleep(2)
log.info('333')

```

TimedRotatingFileHandler类的参数意义如下:

- filename：日志文件名的prefix；
- when：是一个字符串，用于描述滚动周期的基本单位，字符串的值及意义如下： 
&nbsp;“S”: Seconds 
&nbsp;“M”: Minutes 
&nbsp;“H”: Hours 
&nbsp;“D”: Days 
&nbsp;“W”: Week day (0=Monday) 
&nbsp;“midnight”: Roll over at midnight
- interval: 滚动周期，单位有when指定，比如：when=’D’,interval=1，表示每天产生一个日志文件；
- backupCount: 表示日志文件的保留个数； 不写则全保存

除了上述参数之外，TimedRotatingFileHandler还有两个比较重要的成员变量，它们分别是suffix和extMatch。suffix是指日志文件名的后缀,suffix中通常带有格式化的时间字符串，filename和suffix由“.”连接构成文件名（例如：filename=“runtime”， suffix=“%Y-%m-%d.log”,生成的文件名为runtime.2015-07-06.log）