title: 用于python的定时任务apscheduler的使用
comment: true
donate: true
categories:
  - Python
date: 2017-06-17 21:42:13
tags: 
  - python
permalink: use-apscheduler

---

最近在项目中有一个比较特殊的需求，要求在每个月第二个周二暂停任务，然后周三再开启
于是在网上查了一下，python中有一个apscheduler库可以实现，而且这个框架还挺强大的
这里记录一下它的使用


<!-- more -->

### 安装

我一开始在python2.6中安装的，但是安装有错误，估计是支持不好，在python2.7中是可以的
> pip install apscheduler -i https://pypi.douban.com/simple

安装过程中会额外安装一个pytz的库，是关于timezone的

### 基础概念 参考 [http://debugo.com/apscheduler/](http://debugo.com/apscheduler/)

> 触发器(trigger)包含调度逻辑，每一个作业有它自己的触发器，用于决定接下来哪一个作业会运行。除了他们自己初始配置意外，触发器完全是无状态的。
作业存储(job store)存储被调度的作业，默认的作业存储是简单地把作业保存在内存中，其他的作业存储是将作业保存在数据库中。一个作业的数据讲在保存在持久化作业存储时被序列化，并在加载时被反序列化。调度器不能分享同一个作业存储。
执行器(executor)处理作业的运行，他们通常通过在作业中提交制定的可调用对象到一个线程或者进城池来进行。当作业完成时，执行器将会通知调度器。
调度器(scheduler)是其他的组成部分。你通常在应用只有一个调度器，应用的开发者通常不会直接处理作业存储、调度器和触发器，相反，调度器提供了处理这些的合适的接口。配置作业存储和执行器可以在调度器中完成，例如添加、修改和移除作业。
你需要选择合适的调度器，这取决于你的应用环境和你使用APScheduler的目的。通常最常用的两个：
– BlockingScheduler: 当调度器是你应用中唯一要运行的东西时使用。
– BackgroundScheduler: 当你不运行任何其他框架时使用，并希望调度器在你应用的后台执行。

最主要用的就是scheduler与job store

### 作业存储

一般存储分为内存存储和持久化的存储，推荐使用持久化的存储，这样一旦主机挂了或者重启了，这样只要重新运行脚本就可以接着运行了，我这里使用的是mongodb来存储。

### 调度器

任务的执行调度工作由其来完成，主要用到的有BlockingScheduler（阻塞的），BackgroundScheduler（非阻塞的）

一个简单的例子，每隔5s钟输出‘hello world’，每天的13点50分输出‘i m blocking task’

``` python 
#coding:utf-8

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.schedulers.blocking import BlockingScheduler
import time
from datetime import datetime
from pymongo import MongoClient

mongoDBhost = 'vps.yangyanxing.com' #mongodb 服务器
mongoDBport = '29017' # 端口号
mongoDBuser = 'yangyanxing' # 用户名
mongoDBpwd = 'pwd' # 密码

mongoclient = MongoClient(host=['%s:%s'%(mongoDBhost,mongoDBport)])
mongoclient.admin.authenticate(mongoDBuser,mongoDBpwd)

dbjob = mongoclient.mac.jobs  # mongodb所用到的collection，这里是BackgroundScheduler的
dbjob_b = mongoclient.mac.jobs_block # mongodb所用到的collection，这里是blockingScheduler的

job_defaults = {
    'coalesce': False,
    'max_instances': 3,
    'misfire_grace_time': 30
}

def timetest():
    print 'hello world'

def timetestblock():
    print 'i m blocking task'

if __name__ == '__main__':
    scheduler = BackgroundScheduler(job_defaults=job_defaults)
    scheduler_b = BlockingScheduler(job_defaults=job_defaults)
    scheduler.add_jobstore('mongodb', client=dbjob)
    scheduler_b.add_jobstore('mongodb', client=dbjob_b)
    scheduler.add_job(timetest, 'interval', seconds=5)
    scheduler_b.add_job(timetestblock, 'cron', minute=50, hour=13, start_date=datetime.now())
    scheduler.start()
    scheduler_b.start()

```

运行脚本发现报错了。。。
> pytz.exceptions.UnknownTimeZoneError: u'Can not find timezone China Standard time'

![时区找不着的错误](/image/timezoneerror.png)

有的环境是不报这个错误的，如果报的话就手动指定一个时区，指定一个pytz库中定义了的时区，可以在site-packages中查看一下，具体原因没有细追。。。

```
timez = pytz.timezone('Asia/Shanghai')
# 初始化scheduler的时候加上timezone参数
scheduler = BackgroundScheduler(job_defaults=job_defaults,timezone=timez)
scheduler_b = BlockingScheduler(job_defaults=job_defaults,timezone=timez)

```

再次运行该脚本，则可以正常运行了，查看mongo数据库
![mongodb中该任务的记录](/image/mongorecord.png)

这里也记录着next_run_time

最终的代码

``` python 
#coding:utf-8

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.schedulers.blocking import BlockingScheduler
import time
from datetime import datetime
from pymongo import MongoClient
import pytz

'''
两个问题
1. 对于 interval，比如说每5秒钟进行一次的任务，当该脚本被停了，重新运行该脚本的时候，已经过了nexttime了，
   程序是怎么操作的
2. 报pytz.exceptions.UnknownTimeZoneError: u'Can not find timezone China Standard time' 的问题
'''

timez = pytz.timezone('Asia/Shanghai')
# 当有不识别的timezone的时候，初始化的时候可以加上timezone，最好也要加上，否则时间日期不对应会出问题
mongoDBhost = 'vps.yangyanxing.com'
mongoDBport = '29017'
mongoDBuser = 'yangyanxing'
mongoDBpwd = 'pwd'



mongoclient = MongoClient(host=['%s:%s'%(mongoDBhost,mongoDBport)])
mongoclient.admin.authenticate(mongoDBuser,mongoDBpwd)

dbjob = mongoclient.mac.jobs
dbjob_b = mongoclient.mac.jobs_block

job_defaults = {
    'coalesce': False,
    'max_instances': 3,
    'misfire_grace_time': 30
}

def timetest():
    print time.strftime('%Y%m%d--%H:%M:%S',time.localtime(time.time())),'hello world'

def timetestblock():
    print 'i m blocking task'


if __name__ == '__main__':
    scheduler = BackgroundScheduler(timezone=timez)
    scheduler_b = BlockingScheduler(timezone=timez)
    scheduler.add_jobstore('mongodb', client=dbjob)
    scheduler_b.add_jobstore('mongodb', client=dbjob_b)
    scheduler.add_job(timetest, 'interval', seconds=5)
    scheduler_b.add_job(timetestblock, 'cron', minute=50, hour=13, start_date=datetime.now())
    scheduler.start()
    scheduler_b.start()
    
```

### 几个问题

1、运行的时候会有一些误差，由于我这个mongodb在国外的VPS上，所以在操作的时候就有一些延迟，正常如果很快的话误差不会很大
2、关于timezone，如果有报错的话则要手工的指定，在中国境内可以定义为'Asia/Shanghai'
3、添加作业的时候，类型可以为cron,这个定义和linux中的crontab格式，比较灵活，而且它本身就可以定义第周几进行，第几个星期几等，推荐使用
4、添加作业的时候也可以使用装饰器
``` python
@scheduler_b.scheduled_job('cron',id='timetest_b',minute=50, hour=13)
def timetestblock():
    print 'i m blocking task'

```







