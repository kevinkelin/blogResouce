title: 使用Redis作为消息队列实现生产消费与发布订阅
comment: true
categories:
  - Python
date: 2018-12-26 23:53:38
tags: python
permalink: use-redis-for-mq-in-python

---
日常的工作中，经常会用到队列(Queue)，在python中有原生的队列，但是由于原生的队列是存在于内存当中，当系统重启后队列里的消息就没有，且无法进行分步式，Redis中的List数据有四种原语，LPUSH,LPOP,RPUSH,RPOP，配合使用可以实现简单的生产消费模型。

<!-- more -->

# 原语说明
push 是向列表中添加信息，pop是从列表中读取信息，l与r 则是左和右或者说列表头和列表尾，lpush将消息放到列表头，rpush将消息放到列表尾。

``` python
>>> import redis
>>> pool = redis.ConnectionPool(host='192.168.99.100', port=6739, db=0)
>>> r=redis.Redis(connection_pool=pool)
>>> r.lpush('task',"task1")
1L
>>> r.lpush('task',"task2")
2L
>>> r.lpush('task',"task10")
3L
>>> r.lindex('task',0)
'task10'
>>> r.lindex('task',1)
'task2'
>>> r.lindex('task',2)
'task1'
>>> r.lindex('task',3)
>>>

```

可以看出每次调用lpush时，数据都添加到列表的最前面.

lpop是从列表的头开始出数据,现在task中有三条数据,['task10','task2','task1'],现在调用三次lpop
``` python 
>>> r.lpop('task')
'task10'
>>> r.lpop('task')
'task2'
>>> r.lpop('task')
'task1'
>>> r.lpop('task')
>>> r.lpop('task')

```
可以看到lpop是从列表头中开始弹出数据的，当列表中没有数据的时候，则返回空。

rpush与rpop与其相反，从列表尾部进行操作
``` python 
>>> r.rpush('task',1)
1L
>>> r.rpush('task',2)
2L
>>> r.rpush('task',3)
3L
>>> r.rpush('task',10)
4L
>>> for i in range(4):
...     print r.lindex('task',i)
...
1
2
3
10
>>> r.rpop('task')
'10'
>>> r.rpop('task')
'3'
>>> r.rpop('task')
'2'
>>> r.rpop('task')
'1'
>>> r.rpop('task')

```

# 实现生产消费模型

那么利用列表的push与pop命令就可以实现简单的生产消费了

``` python 
#coding:gbk
import redis
import time,threading

pool = redis.ConnectionPool(host='192.168.99.100', port=6739, db=0)
r=redis.Redis(connection_pool=pool)


def ping(r):
    # 一直执行ping命令，防止连接丢失
    while 1:
        try:
            r.ping()
        except:
            pass
        finally:
            time.sleep(1)


def producer(r,l):
        times = 10
        while times >0 :
            l.acquire()
            try:
                r.lpush('task','%s produce data'%threading.current_thread().name)
            finally:
                l.release()
                time.sleep(2)
                times -=1

def consumer(r,l):
    while 1:
        l.acquire()
        task = None
        try:
            task = r.lpop('task')
            if task:
                print "%s get a task %s "%(threading.current_thread().name,task)
        finally:
            l.release()
            time.sleep(1)

if __name__ == '__main__':
    thlist = []
    t1 = threading.Thread(target=ping,args=(r,))
    t1.setDaemon(True)
    thlist.append(t1)
    lock_c = threading.Lock()
    lock_p = threading.Lock()

    for i in range(10): #10个生产者与消费者
        t_p = threading.Thread(target=producer,args=(r,lock_p),name="Producer:%s"%i)
        t_c = threading.Thread(target=consumer, args=(r, lock_c), name="Consumer:%s" % i)
        thlist.append(t_p)
        thlist.append(t_c)
        t_p.setDaemon(True)
        t_c.setDaemon(True)
    for th in thlist:
        th.start()
        # th.join()

    print 2222
    time.sleep(10)

```

# 阻塞pop
lpop与rpop还有一个阻塞的版本,当给定列表内没有任何元素可供弹出的时候，连接将被 BLPOP 命令阻塞，直到等待超时或发现可弹出元素为止。有时候，为了等待一个新元素到达数据中，需要使用轮询的方式对数据进行探查。

另一种更好的方式是，使用系统提供的阻塞原语，在新元素到达时立即进行处理，而新元素还没到达时，就一直阻塞住，避免轮询占用资源。


# 发布订阅

可以配合使用`publish` 和 `pubsub` 来实现发布订阅

### 发布

``` python 
pool = redis.ConnectionPool(host='192.168.99.100', port=6739, db=0)
r=redis.Redis(connection_pool=pool)
r.publish('mychanel','hello everyone')
```

该方法返回的是订阅者的数量


### 订阅
``` python
pool = redis.ConnectionPool(host='192.168.99.100', port=6739, db=0)
r=redis.Redis(connection_pool=pool)
p = r.pubsub() #打开订阅功能
p.subscribe(['mychanel']) # 订阅关注的频道，可以是多个
p.parse_response()
```
订阅者能收到的信息只能是自它开始订阅之后的消息，之前已经发布的就不能收到了。

`parse_response()` 是阻塞的，只有当收到消息时才结束，可以写一个while 循环，但还有一个更好的方法，是调用它的`listen()` 方法

``` python
>>> for i in p.listen():
...     print i
```


redis可以作为简单的消息队列来用，但是它毕竟不是专业的消息队列，如果对于有很大的消息队列需求的系统还是考虑使用专业的MQ如kafka等。







