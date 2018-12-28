title: 利用logging模块轻松地进行Python日志记录(转)
comment: true
categories:
  - Python
date: 2018-06-23 23:53:38
tags: python
permalink: use-logging-in-python

---

在开发过程中，如果程序运行出现了问题，我们是可以使用我们自己的 Debug 工具来检测到到底是哪一步出现了问题，如果出现了问题的话，是很容易排查的。但程序开发完成之后，我们会将它部署到生产环境中去，这时候代码相当于是在一个黑盒环境下运行的，我们只能看到其运行的效果，是不能直接看到代码运行过程中每一步的状态的。在这个环境下，运行过程中难免会在某个地方出现问题，甚至这个问题可能是我们开发过程中未曾遇到的问题，碰到这种情况应该怎么办？
如果我们现在只能得知当前问题的现象，而没有其他任何信息的话，如果我们想要解决掉这个问题的话，那么只能根据问题的现象来试图复现一下，然后再一步步去调试，这恐怕是很难的，很大的概率上我们是无法精准地复现这个问题的，而且 Debug 的过程也会耗费巨多的时间，这样一旦生产环境上出现了问题，修复就会变得非常棘手。但这如果我们当时有做日志记录的话，不论是正常运行还是出现报错，都有相关的时间记录，状态记录，错误记录等，那么这样我们就可以方便地追踪到在当时的运行过程中出现了怎样的状况，从而可以快速排查问题。
因此，日志记录是非常有必要的，任何一款软件如果没有标准的日志记录，都不能算作一个合格的软件。作为开发者，我们需要重视并做好日志记录过程。


<!-- more -->

# 日志记录的流程框架
那么在 Python 中，怎样才能算作一个比较标准的日志记录过程呢？或许很多人会使用 print 语句输出一些运行信息，然后再在控制台观察，运行的时候再将输出重定向到文件输出流保存到文件中，这样其实是非常不规范的，在 Python 中有一个标准的 logging 模块，我们可以使用它来进行标注的日志记录，利用它我们可以更方便地进行日志记录，同时还可以做更方便的级别区分以及一些额外日志信息的记录，如时间、运行模块信息等。
接下来我们先了解一下日志记录流程的整体框架。
整个日志记录的框架可以分为这么几个部分：

- Logger：即 Logger Main Class，是我们进行日志记录时创建的对象，我们可以调用它的方法传入日志模板和信息，来生成一条条日志记录，称作 Log Record。
- Log Record：就代指生成的一条条日志记录。
- Handler：即用来处理日志记录的类，它可以将 Log Record 输出到我们指定的日志位置和存储形式等，如我们可以指定将日志通过 FTP 协议记录到远程的服务器上，Handler 就会帮我们完成这些事情。
- Formatter：实际上生成的 Log Record 也是一个个对象，那么我们想要把它们保存成一条条我们想要的日志文本的话，就需要有一个格式化的过程，那么这个过程就由 Formatter 来完成，返回的就是日志字符串，然后传回给 Handler 来处理。
- Filter：另外保存日志的时候我们可能不需要全部保存，我们可能只需要保存我们想要的部分就可以了，所以保存前还需要进行一下过滤，留下我们想要的日志，如只保存某个级别的日志，或只保存包含某个关键字的日志等，那么这个过滤过程就交给 Filter 来完成。
- Parent Handler：Handler 之间可以存在分层关系，以使得不同 Handler 之间共享相同功能的代码。

以上就是整个 logging 模块的基本架构和对象功能，了解了之后我们详细来了解一下 logging 模块的用法。

# 日志记录的相关用法

总的来说 logging 模块相比 print 有这么几个优点：

- 可以在 logging 模块中设置日志等级，在不同的版本（如开发环境、生产环境）上通过设置不同的输出等级来记录对应的日志，非常灵活。
- print 的输出信息都会输出到标准输出流中，而 logging 模块就更加灵活，可以设置输出到任意位置，如写入文件、写入远程服务器等。
- logging 模块具有灵活的配置和格式化功能，如配置输出当前模块信息、运行时间等，相比 print 的字符串格式化更加方便易用。

下面我们初步来了解下 logging 模块的基本用法，先用一个实例来感受一下：

``` python 
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

logger.info('This is a log info')
logger.debug('Debugging')
logger.warning('Warning exists')
logger.info('Finish')
``` 

在这里我们首先引入了 logging 模块，然后进行了一下基本的配置，这里通过 basicConfig 配置了 level 信息和 format 信息，这里 level 配置为 INFO 信息，即只输出 INFO 级别的信息，另外这里指定了 format 格式的字符串，包括 asctime、name、levelname、message 四个内容，分别代表运行时间、模块名称、日志级别、日志内容，这样输出内容便是这四者组合而成的内容了，这就是 logging 的全局配置。
接下来声明了一个 Logger 对象，它就是日志输出的主类，调用对象的 info() 方法就可以输出 INFO 级别的日志信息，调用 debug() 方法就可以输出 DEBUG 级别的日志信息，非常方便。在初始化的时候我们传入了模块的名称，这里直接使用 __name__ 来代替了，就是模块的名称，如果直接运行这个脚本的话就是 __main__，如果是 import 的模块的话就是被引入模块的名称，这个变量在不同的模块中的名字是不同的，所以一般使用 __name__ 来表示就好了，再接下来输出了四条日志信息，其中有两条 INFO、一条 WARNING、一条 DEBUG 信息，我们看下输出结果：

>2018-06-03 13:42:43,526 - __main__ - INFO - This is a log info
2018-06-03 13:42:43,526 - __main__ - WARNING - Warning exists
2018-06-03 13:42:43,526 - __main__ - INFO - Finish

可以看到输出结果一共有三条日志信息，每条日志都是对应了指定的格式化内容，另外我们发现 DEBUG 的信息是没有输出的，这是因为我们在全局配置的时候设置了输出为 INFO 级别，所以 DEBUG 级别的信息就被过滤掉了。
这时如果我们将输出的日志级别设置为 DEBUG，就可以看到 DEBUG 级别的日志输出了：
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
输出结果：

> 2018-06-03 13:49:22,770 - __main__ - INFO - This is a log info
2018-06-03 13:49:22,770 - __main__ - DEBUG - Debugging
2018-06-03 13:49:22,770 - __main__ - WARNING - Warning exists
2018-06-03 13:49:22,770 - __main__ - INFO - Finish

由此可见，相比 print 来说，通过刚才的代码，我们既可以输出时间、模块名称，又可以输出不同级别的日志信息作区分并加以过滤，是不是灵活多了？
当然这只是 logging 模块的一小部分功能，接下来我们首先来全面了解一下 basicConfig 的参数都有哪些：

- filename：即日志输出的文件名，如果指定了这个信息之后，实际上会启用 FileHandler，而不再是 StreamHandler，这样日志信息便会输出到文件中了。
- filemode：这个是指定日志文件的写入方式，有两种形式，一种是 w，一种是 a，分别代表清除后写入和追加写入。
- format：指定日志信息的输出格式，即上文示例所示的参数，详细参数可以参考：docs.python.org/3/library/l…，部分参数如下所示：

> %(levelno)s：打印日志级别的数值。
%(levelname)s：打印日志级别的名称。
%(pathname)s：打印当前执行程序的路径，其实就是sys.argv[0]。
%(filename)s：打印当前执行程序名。
%(funcName)s：打印日志的当前函数。
%(lineno)d：打印日志的当前行号。
%(asctime)s：打印日志的时间。
%(thread)d：打印线程ID。
%(threadName)s：打印线程名称。
%(process)d：打印进程ID。
%(processName)s：打印线程名称。
%(module)s：打印模块名称。
%(message)s：打印日志信息。


- datefmt：指定时间的输出格式。
- style：如果 format 参数指定了，这个参数就可以指定格式化时的占位符风格，如 %、{、$ 等。
- level：指定日志输出的类别，程序会输出大于等于此级别的信息。
- stream：在没有指定 filename 的时候会默认使用 StreamHandler，这时 stream 可以指定初始化的文件流。
- handlers：可以指定日志处理时所使用的 Handlers，必须是可迭代的。

下面我们再用一个实例来感受一下：

``` python

import logging
logging.basicConfig(level=logging.DEBUG,
                    filename='output.log',
                    datefmt='%Y/%m/%d %H:%M:%S',
                    format='%(asctime)s - %(name)s - %(levelname)s - %(lineno)d - %(module)s - %(message)s')
logger = logging.getLogger(__name__)

logger.info('This is a log info')
logger.debug('Debugging')
logger.warning('Warning exists')
logger.info('Finish')
```

这里我们指定了输出文件的名称为 output.log，另外指定了日期的输出格式，其中年月日的格式变成了 %Y/%m/%d，另外输出的 format 格式增加了 lineno、module 这两个信息，运行之后便会生成一个 output.log 的文件，内容如下：
> 2018/06/03 14:43:26 - __main__ - INFO - 9 - demo3 - This is a log info
2018/06/03 14:43:26 - __main__ - DEBUG - 10 - demo3 - Debugging
2018/06/03 14:43:26 - __main__ - WARNING - 11 - demo3 - Warning exists
2018/06/03 14:43:26 - __main__ - INFO - 12 - demo3 - Finish

可以看到日志便会输出到文件中，同时输出了行号、模块名称等信息。

以上我们通过 basicConfig 来进行了一些全局的配置，我们同样可以使用 Formatter、Handler 进行更灵活的处理，下面我们来了解一下。

## Level
首先我们来了解一下输出日志的等级信息，logging 模块共提供了如下等级，每个等级其实都对应了一个数值，列表如下：

|等级|数值|
| - | - |
|CRITICAL|50|
|FATAL|50|
|ERROR|40|
|WARNING|30|
|WARN|30|
|INFO|20|
|DEBUG|10|
|NOTSET|0|



这里最高的等级是 CRITICAL 和 FATAL，两个对应的数值都是 50，另外对于 WARNING 还提供了简写形式 WARN，两个对应的数值都是 30。
我们设置了输出 level，系统便只会输出 level 数值大于或等于该 level 的的日志结果，例如我们设置了输出日志 level 为 INFO，那么输出级别大于等于 INFO 的日志，如 WARNING、ERROR 等，DEBUG 和 NOSET 级别的不会输出。

``` python 
import logging

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.WARN)

# Log
logger.debug('Debugging')
logger.critical('Critical Something')
logger.error('Error Occurred')
logger.warning('Warning exists')
logger.info('Finished')
```

这里我们设置了输出级别为 WARN，然后对应输出了五种不同级别的日志信息，运行结果如下：

> Critical Something
Error Occurred
Warning exists

可以看到只有 CRITICAL、ERROR、WARNING 信息输出了，DEBUG、INFO 信息没有输出。

## Handler

下面我们先来了解一下 Handler 的用法，看下面的实例：

``` python 
import logging

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)
handler = logging.FileHandler('output.log')
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

logger.info('This is a log info')
logger.debug('Debugging')
logger.warning('Warning exists')
logger.info('Finish')
```

这里我们没有再使用 basicConfig 全局配置，而是先声明了一个 Logger 对象，然后指定了其对应的 Handler 为 FileHandler 对象，然后 Handler 对象还单独指定了 Formatter 对象单独配置输出格式，最后给 Logger 对象添加对应的 Handler 即可，最后可以发现日志就会被输出到 output.log 中，内容如下：
> 2018-06-03 14:53:36,467 - __main__ - INFO - This is a log info
2018-06-03 14:53:36,468 - __main__ - WARNING - Warning exists
2018-06-03 14:53:36,468 - __main__ - INFO - Finish

另外我们还可以使用其他的 Handler 进行日志的输出，logging 模块提供的 Handler 有：

- StreamHandler：logging.StreamHandler；日志输出到流，可以是 sys.stderr，sys.stdout 或者文件。
- FileHandler：logging.FileHandler；日志输出到文件。
- BaseRotatingHandler：logging.handlers.BaseRotatingHandler；基本的日志回滚方式。
- RotatingHandler：logging.handlers.RotatingHandler；日志回滚方式，支持日志文件最大数量和日志文件回滚。
- TimeRotatingHandler：logging.handlers.TimeRotatingHandler；日志回滚方式，在一定时间区域内回滚日志文件。
- SocketHandler：logging.handlers.SocketHandler；远程输出日志到TCP/IP sockets。
- DatagramHandler：logging.handlers.DatagramHandler；远程输出日志到UDP sockets。
- SMTPHandler：logging.handlers.SMTPHandler；远程输出日志到邮件地址。
- SysLogHandler：logging.handlers.SysLogHandler；日志输出到syslog。
- NTEventLogHandler：logging.handlers.NTEventLogHandler；远程输出日志到Windows NT/2000/XP的事件日志。
- MemoryHandler：logging.handlers.MemoryHandler；日志输出到内存中的指定buffer。
- HTTPHandler：logging.handlers.HTTPHandler；通过"GET"或者"POST"远程输出到HTTP服务器。

下面我们使用三个 Handler 来实现日志同时输出到控制台、文件、HTTP 服务器：

``` python 
import logging
from logging.handlers import HTTPHandler
import sys

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.DEBUG)

# StreamHandler
stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.setLevel(level=logging.DEBUG)
logger.addHandler(stream_handler)

# FileHandler
file_handler = logging.FileHandler('output.log')
file_handler.setLevel(level=logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)

# HTTPHandler
http_handler = HTTPHandler(host='localhost:8001', url='log', method='POST')
logger.addHandler(http_handler)

# Log
logger.info('This is a log info')
logger.debug('Debugging')
logger.warning('Warning exists')
logger.info('Finish')

```

运行之前我们需要先启动 HTTP Server，并运行在 8001 端口，其中 log 接口是用来接收日志的接口。
运行之后控制台输出会输出如下内容：

> This is a log info
Debugging
Warning exists
Finish

output.log 文件会写入如下内容：

> 2018-06-03 15:13:44,895 - __main__ - INFO - This is a log info
2018-06-03 15:13:44,947 - __main__ - WARNING - Warning exists
2018-06-03 15:13:44,949 - __main__ - INFO - Finish

HTTP Server 会收到控制台输出的信息。

这样一来，我们就通过设置多个 Handler 来控制了日志的多目标输出。

另外值得注意的是，在这里 StreamHandler 对象我们没有设置 Formatter，因此控制台只输出了日志的内容，而没有包含时间、模块等信息，而 FileHandler 我们通过 setFormatter() 方法设置了一个 Formatter 对象，因此输出的内容便是格式化后的日志信息。

另外每个 Handler 还可以设置 level 信息，最终输出结果的 level 信息会取 Logger 对象的 level 和 Handler 对象的 level 的交集。

## Formatter

在进行日志格式化输出的时候，我们可以不借助于 basicConfig 来全局配置格式化输出内容，可以借助于 Formatter 来完成，下面我们再来单独看下 Formatter 的用法：

``` python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.WARN)
formatter = logging.Formatter(fmt='%(asctime)s - %(name)s - %(levelname)s - %(message)s', datefmt='%Y/%m/%d %H:%M:%S')
handler = logging.StreamHandler()
handler.setFormatter(formatter)
logger.addHandler(handler)

# Log
logger.debug('Debugging')
logger.critical('Critical Something')
logger.error('Error Occurred')
logger.warning('Warning exists')
logger.info('Finished')
```

在这里我们指定了一个 Formatter，并传入了 fmt 和 datefmt 参数，这样就指定了日志结果的输出格式和时间格式，然后 handler 通过 setFormatter() 方法设置此 Formatter 对象即可，输出结果如下：

> 2018/06/03 15:47:15 - __main__ - CRITICAL - Critical Something
2018/06/03 15:47:15 - __main__ - ERROR - Error Occurred
2018/06/03 15:47:15 - __main__ - WARNING - Warning exists

这样我们可以每个 Handler 单独配置输出的格式，非常灵活。

## 捕获 Traceback

如果遇到错误，我们更希望报错时出现的详细 Traceback 信息，便于调试，利用 logging 模块我们可以非常方便地实现这个记录，我们用一个实例来感受一下：

``` python 
import logging

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.DEBUG)

# Formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# FileHandler
file_handler = logging.FileHandler('result.log')
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)

# StreamHandler
stream_handler = logging.StreamHandler()
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)

# Log
logger.info('Start')
logger.warning('Something maybe fail.')
try:
    result = 10 / 0
except Exception:
    logger.error('Faild to get result', exc_info=True)
logger.info('Finished')
```

这里我们在 error() 方法中添加了一个参数，将 exc_info 设置为了 True，这样我们就可以输出执行过程中的信息了，即完整的 Traceback 信息。
运行结果如下：

> 2018-06-03 16:00:15,382 - __main__ - INFO - Start print log
2018-06-03 16:00:15,382 - __main__ - DEBUG - Do something
2018-06-03 16:00:15,382 - __main__ - WARNING - Something maybe fail.
2018-06-03 16:00:15,382 - __main__ - ERROR - Faild to get result
Traceback (most recent call last):
  File "/private/var/books/aicodes/loggingtest/demo8.py", line 23, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero
2018-06-03 16:00:15,383 - __main__ - INFO - Finished
可以看到这样我们就非常方便地记录下来了报错的信息，一旦出现了错误，我们也能非常方便地排查。

# 配置共享

在写项目的时候，我们肯定会将许多配置放置在许多模块下面，这时如果我们每个文件都来配置 logging 配置那就太繁琐了，logging 模块提供了父子模块共享配置的机制，会根据 Logger 的名称来自动加载父模块的配置。
例如我们这里首先定义一个 main.py 文件：

``` python
import logging
import core

logger = logging.getLogger('main')
logger.setLevel(level=logging.DEBUG)

# Handler
handler = logging.FileHandler('result.log')
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)

logger.info('Main Info')
logger.debug('Main Debug')
logger.error('Main Error')
core.run()
```

这里我们配置了日志的输出格式和文件路径，同时定义了 Logger 的名称为 main，然后引入了另外一个模块 core，最后调用了 core 的 run() 方法。
接下来我们定义 core.py，内容如下：

``` python
import logging

logger = logging.getLogger('main.core')

def run():
    logger.info('Core Info')
    logger.debug('Core Debug')
    logger.error('Core Error')
```

这里我们定义了 Logger 的名称为 main.core，注意这里开头是 main，即刚才我们在 main.py 里面的 Logger 的名称，这样 core.py 里面的 Logger 就会复用 main.py 里面的 Logger 配置，而不用再去配置一次了。
运行之后会生成一个 result.log 文件，内容如下：


> 2018-06-03 16:55:56,259 - main - INFO - Main Info
2018-06-03 16:55:56,259 - main - ERROR - Main Error
2018-06-03 16:55:56,259 - main.core - INFO - Core Info
2018-06-03 16:55:56,259 - main.core - ERROR - Core Error

可以看到父子模块都使用了同样的输出配置。
如此一来，我们只要在入口文件里面定义好 logging 模块的输出配置，子模块只需要在定义 Logger 对象时名称使用父模块的名称开头即可共享配置，非常方便。
文件配置
在开发过程中，将配置在代码里面写死并不是一个好的习惯，更好的做法是将配置写在配置文件里面，我们可以将配置写入到配置文件，然后运行时读取配置文件里面的配置，这样是更方便管理和维护的，下面我们以一个实例来说明一下，首先我们定义一个 yaml 配置文件：

``` python 
version: 1
formatters:
  brief:
    format: "%(asctime)s - %(message)s"
  simple:
    format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
handlers:
  console:
    class : logging.StreamHandler
    formatter: brief
    level   : INFO
    stream  : ext://sys.stdout
  file:
    class : logging.FileHandler
    formatter: simple
    level: DEBUG
    filename: debug.log
  error:
    class: logging.handlers.RotatingFileHandler
    level: ERROR
    formatter: simple
    filename: error.log
    maxBytes: 10485760
    backupCount: 20
    encoding: utf8
loggers:
  main.core:
    level: DEBUG
    handlers: [console, file, error]
root:
  level: DEBUG
  handlers: [console]
```

这里我们定义了 formatters、handlers、loggers、root 等模块，实际上对应的就是各个 Formatter、Handler、Logger 的配置，参数和它们的构造方法都是相同的。
接下来我们定义一个主入口文件，main.py，内容如下：

``` python
import logging
import core
import yaml
import logging.config
import os


def setup_logging(default_path='config.yaml', default_level=logging.INFO):
    path = default_path
    if os.path.exists(path):
        with open(path, 'r', encoding='utf-8') as f:
            config = yaml.load(f)
            logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)


def log():
    logging.debug('Start')
    logging.info('Exec')
    logging.info('Finished')


if __name__ == '__main__':
    yaml_path = 'config.yaml'
    setup_logging(yaml_path)
    log()
    core.run()
```

这里我们定义了一个 setup_logging() 方法，里面读取了 yaml 文件的配置，然后通过 dictConfig() 方法将配置项传给了 logging 模块进行全局初始化。
另外这个模块还引入了另外一个模块 core，所以我们定义 core.py 如下：

``` python 
import logging

logger = logging.getLogger('main.core')

def run():
    logger.info('Core Info')
    logger.debug('Core Debug')
    logger.error('Core Error')
```

这个文件的内容和上文是没有什么变化的。
观察配置文件，主入口文件 main.py 实际上对应的是 root 一项配置，它指定了 handlers 是 console，即只输出到控制台。另外在 loggers 一项配置里面，我们定义了 main.core 模块，handlers 是 console、file、error 三项，即输出到控制台、输出到普通文件和回滚文件。
这样运行之后，我们便可以看到所有的运行结果输出到了控制台：

> 2018-06-03 17:07:12,727 - Exec
2018-06-03 17:07:12,727 - Finished
2018-06-03 17:07:12,727 - Core Info
2018-06-03 17:07:12,727 - Core Info
2018-06-03 17:07:12,728 - Core Error
2018-06-03 17:07:12,728 - Core Error

在 debug.log 文件中则包含了 core.py 的运行结果：

> 2018-06-03 17:07:12,727 - main.core - INFO - Core Info
2018-06-03 17:07:12,727 - main.core - DEBUG - Core Debug
2018-06-03 17:07:12,728 - main.core - ERROR - Core Error

可以看到，通过配置文件，我们可以非常灵活地定义 Handler、Formatter、Logger 等配置，同时也显得非常直观，也非常容易维护，在实际项目中，推荐使用此种方式进行配置。
以上便是 logging 模块的基本使用方法，有了它，我们可以方便地进行日志管理和维护，会给我们的工作带来极大的方便。

# 日志记录使用常见误区

在日志输出的时候经常我们会用到字符串拼接的形式，很多情况下我们可能会使用字符串的 format() 来构造一个字符串，但这其实并不是一个好的方法，因为还有更好的方法，下面我们对比两个例子：

``` python
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# bad
logging.debug('Hello {0}, {1}!'.format('World', 'Congratulations'))
# good
logging.debug('Hello %s, %s!', 'World', 'Congratulations')

```

这里有两种打印 Log 的方法，第一种使用了字符串的 format() 的方法进行构造，传给 logging 的只用到了第一个参数，实际上 logging 模块提供了字符串格式化的方法，我们只需要在第一个参数写上要打印输出的模板，占位符用 %s、%d 等表示即可，然后在后续参数添加对应的值就可以了，推荐使用这种方法。
运行结果如下：

> 2018-06-03 22:27:51,220 - root - DEBUG - Hello World, Congratulations!
2018-06-03 22:27:51,220 - root - DEBUG - Hello World, Congratulations!

另外在进行异常处理的时候，通常我们会直接将异常进行字符串格式化，但其实可以直接指定一个参数将 traceback 打印出来，示例如下：

``` python
import logging

logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')

try:
    result = 5 / 0
except Exception as e:
    # bad
    logging.error('Error: %s', e)
    # good
    logging.error('Error', exc_info=True)
    # good
    logging.exception('Error')
```

如果我们直接使用字符串格式化的方法将错误输出的话，是不会包含 Traceback 信息的，但如果我们加上 exc_info 参数或者直接使用 exception() 方法打印的话，那就会输出 Traceback 信息了。
运行结果如下：

> 2018-06-03 22:24:31,927 - root - ERROR - Error: division by zero
2018-06-03 22:24:31,927 - root - ERROR - Error
Traceback (most recent call last):
  File "/private/var/books/aicodes/loggingtest/demo9.py", line 6, in <module>
    result = 5 / 0
ZeroDivisionError: division by zero
2018-06-03 22:24:31,928 - root - ERROR - Error
Traceback (most recent call last):
  File "/private/var/books/aicodes/loggingtest/demo9.py", line 6, in <module>
    result = 5 / 0
ZeroDivisionError: division by zero



出处信息

作者：崔庆才丨静觅
链接：https://juejin.im/post/5b13fdd0f265da6e0b6ff3dd
来源：掘金



