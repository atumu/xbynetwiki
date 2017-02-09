title: python学习笔记29 

#  Python学习笔记之日志logging模块 
建议结合http://python.jobbole.com/81666/一起
1.简单的将日志打印到屏幕
```

import logging
logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')

```
屏幕上打印:
WARNING:root:This is warning message
**默认情况下，logging将日志打印到屏幕，日志级别为WARNING；**
日志级别大小关系为：CRITICAL > ERROR > WARNING > INFO > DEBUG > NOTSET，当然也可以自己定义日志级别。
2.通过logging.basicConfig函数对日志的输出格式及方式做相关配置
```

import logging
logging.basicConfig(level=logging.DEBUG,
                format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                datefmt='%a, %d %b %Y %H:%M:%S',
                filename='myapp.log',
                filemode='w')
logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')

```
./myapp.log文件中内容为:
Sun, 24 May 2009 21:48:54 demo2.py[line:11] DEBUG This is debug message
Sun, 24 May 2009 21:48:54 demo2.py[line:12] INFO This is info message
Sun, 24 May 2009 21:48:54 demo2.py[line:13] WARNING This is warning message
##  logging.basicConfig函数各参数: 
filename: 指定日志文件名
filemode: 和file函数意义相同，指定日志文件的打开模式，'w'或'a'
format: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:
  *  %(levelno)s: 打印日志级别的数值
  *  %(levelname)s: 打印日志级别名称
  *  %(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
  *  %(filename)s: 打印当前执行程序名
  *  %(name)s:logger的名字
  *  %(funcName)s: 打印日志的当前函数
  *  %(lineno)d: 打印日志的当前行号
  *  %(asctime)s: 打印日志的时间
  *  %(thread)d: 打印线程ID
  *  %(threadName)s: 打印线程名称
  *  %(process)d: 打印进程ID
  *  %(message)s: 打印日志信息
datefmt: 指定时间格式，同time.strftime()
level: 设置日志级别，**默认为logging.WARNING**
stream: 指定将日志的输出流，可以指定输出到sys.stderr,sys.stdout或者文件，**默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略**
常用配置1："%(asctime)s - %(name)s- %(levelname)s-%(filename)s [line:%(lineno)d]-%(funcName)s - %(message)s"
##  3.将日志同时输出到文件和屏幕 
```

import logging
logging.basicConfig(level=logging.DEBUG,
                format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                datefmt='%a, %d %b %Y %H:%M:%S',
                filename='myapp.log',
                filemode='w')
#定义一个StreamHandler，将INFO级别或更高的日志信息打印到标准错误，并将其添加到当前的日志处理对象#
console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)
logging.getLogger('').addHandler(console)

logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')

```
屏幕上打印:
root        : INFO     This is info message
root        : WARNING  This is warning message
./myapp.log文件中内容为:
Sun, 24 May 2009 21:48:54 demo2.py[line:11] DEBUG This is debug message
Sun, 24 May 2009 21:48:54 demo2.py[line:12] INFO This is info message
Sun, 24 May 2009 21:48:54 demo2.py[line:13] WARNING This is warning message
##  4.logging之日志回滚 
```

import logging
from logging.handlers import RotatingFileHandler
#################################################################################################
#定义一个RotatingFileHandler，最多备份5个日志文件，每个日志文件最大10M
Rthandler = RotatingFileHandler('myapp.log', maxBytes=10*1024*1024,backupCount=5)
Rthandler.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
Rthandler.setFormatter(formatter)
logging.getLogger('').addHandler(Rthandler)
################################################################################################

```
从上例和本例可以看出，logging有一个日志处理的主对象，其它处理方式都是通过addHandler添加进去的。
**logging的几种handle方式如下：**
  * logging.StreamHandler: 日志输出到流，可以是sys.stderr、sys.stdout或者文件
  * logging.FileHandler: 日志输出到文件
**日志回滚方式**，实际使用时用RotatingFileHandler和TimedRotatingFileHandler
  * logging.handlers.BaseRotatingHandler
  * logging.handlers.RotatingFileHandler
  * logging.handlers.TimedRotatingFileHandler
  * logging.handlers.SocketHandler: 远程输出日志到TCP/IP sockets
  * logging.handlers.DatagramHandler:  远程输出日志到UDP sockets
  * logging.handlers.SMTPHandler:  远程输出日志到邮件地址
  * logging.handlers.SysLogHandler: 日志输出到syslog
  * logging.handlers.NTEventLogHandler: 远程输出日志到Windows NT/2000/XP的事件日志
  * logging.handlers.MemoryHandler: 日志输出到内存中的制定buffer
  * logging.handlers.HTTPHandler: 通过"GET"或"POST"远程输出到HTTP服务器
**由于StreamHandler和FileHandler是常用的日志处理方式，所以直接包含在logging模块中，而其他方式则包含在logging.handlers模块中，**
##  5.通过logging.config模块配置日志 
```

#logger.conf
###############################################
[loggers]
keys=root,example01,example02
[logger_root]
level=DEBUG
handlers=hand01,hand02
[logger_example01]
handlers=hand01,hand02
qualname=example01
propagate=0
[logger_example02]
handlers=hand01,hand03
qualname=example02
propagate=0
###############################################
[handlers]
keys=hand01,hand02,hand03
[handler_hand01]
class=StreamHandler
level=INFO
formatter=form02
args=(sys.stderr,)
[handler_hand02]
class=FileHandler
level=DEBUG
formatter=form01
args=('myapp.log', 'a')
[handler_hand03]
class=handlers.RotatingFileHandler
level=INFO
formatter=form02
args=('myapp.log', 'a', 10*1024*1024, 5)
###############################################
[formatters]
keys=form01,form02
[formatter_form01]
format=%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s
datefmt=%a, %d %b %Y %H:%M:%S
[formatter_form02]
format=%(name)-12s: %(levelname)-8s %(message)s
datefmt=

```
```

import logging
import logging.config

logging.config.fileConfig("logger.conf")
logger = logging.getLogger("example01")

logger.debug('This is debug message')
logger.info('This is info message')
logger.warning('This is warning message')

```
6.logging是线程安全的
参考:http://www.cnblogs.com/dkblog/archive/2011/08/26/2155018.html

##  使用json或yaml配置日志 
参考：http://python.jobbole.com/81666/
在 Python2.7 及之后的版本中，**你可以从字典中加载 logging 配置logging.config.dictConfig**。这也就意味着你可以从 JSON 或者 YAML 文件中加载日志的配置。尽管你还能用原来 .ini 文件来配置，但是它既很难读也很难写。
```

{
    "version": 1,
    "disable_existing_loggers": false,
    "formatters": {
        "simple": {
            "format": "%(asctime)s - %(name)s- %(levelname)s-%(filename)s [line:%(lineno)d]-%(funcName)s - %(message)s"
            "datefmt":"%Y-%m-%d %H-%M-%S"
        }
    },

    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "DEBUG",
            "formatter": "simple",
            "stream": "ext://sys.stdout"
        },

        "info_file_handler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "INFO",
            "formatter": "simple",
            "filename": "info.log",
            "maxBytes": 10485760,
            "backupCount": 20,
            "encoding": "utf-8"
        },

        "error_file_handler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "ERROR",
            "formatter": "simple",
            "filename": "errors.log",
            "maxBytes": 10485760,
            "backupCount": 20,
            "encoding": "utf-8"
        }
    },

    "loggers": {
        "app": {
            "level": "WARNING",
            "handlers": ["info_file_handler","error_file_handler"],
            "propagate": "no"
        }
    },

    "root": {
        "level": "DEBUG",
        "handlers": ["console", "info_file_handler", "error_file_handler"]
    }
}

```
```

import json
import logging.config
 
def setup_logging(
    default_path='logging.json', 
    default_level=logging.INFO,
    env_key='LOG_CFG'
):
    """Setup logging configuration
 
    """
    path = default_path
    value = os.getenv(env_key, None)
    if value:
        path = value
    if os.path.exists(path):
        with open(path, 'rt') as f:
            config = json.load(f)
        logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)

```