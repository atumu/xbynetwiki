author:xbynet
title:APScheduler任务调度利器
modifyAt:2016-12-12 03:16:42
location:python/APScheduler任务调度利器
createAt:2016-12-12 03:16:42

Java中任务调度一般用Quartz,Python中的任务调度工具也有不少:Celery,RQ,APScheduler等。
Celery：非常强大的分布式任务调度框架
[RQ][1]：基于Redis的作业队列工具
[APScheduler][2]：一款强大的任务调度工具

RQ参考Celery，据说要比Celery轻量级(Really?)
APScheduler感觉更像Quartz。
本人小小的建议是一般项目用APScheduler，因为不用像Celery那样再单独启动worker、beat进程，而且API也很简洁。
对于大点分布式项目用Celery

官网：http://apscheduler.readthedocs.io/en/v3.3.0/index.html
API：http://apscheduler.readthedocs.io/en/v3.3.0/py-modindex.html
当前版本：3.3.0
安装：$ pip install apscheduler
例子：https://github.com/agronholm/apscheduler/tree/master/examples/?at=master

# 特性
Advanced Python Scheduler (APScheduler) 一款强大的任务调度工具.
内置了三种调度模式：

 - Cron风格
 - 间隔性(Interval-based)执行
 - 仅在某个时间执行一次

作业存储支持以下几种方式：
Memory
SQLAlchemy (any RDBMS supported by SQLAlchemy works)
MongoDB
Redis
RethinkDB
ZooKeeper
除了Memory方式不需要序列化之外(一个例外是使用ProcessPoolExecutor)，其余都需要作业函数参数可序列化。

支持与以下框架集成:
asyncio (PEP 3156)
gevent
Tornado
Twisted
Qt (using either PyQt or PySide)

# 基本概念
四大组件：
triggers
job stores
executors
schedulers

## schedulers
内置以下几种调度器实现：
BlockingScheduler: 
BackgroundScheduler: 默认采用ThreadPoolExecutor池(默认10)，可以配置ProcessPoolExecutor，或同时使用
AsyncIOScheduler: 使用asyncio模块
GeventScheduler: 使用gevent
TornadoScheduler: use if you’re building a Tornado application
TwistedScheduler: use if you’re building a Twisted application
QtScheduler: use if you’re building a Qt application

基类：BaseScheduler，可以通过此类查询相关[配置选项][3]。
### 调度器配置示例：

方式一、
```
from pytz import utc

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.jobstores.mongodb import MongoDBJobStore
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor

jobstores = {
    'mongo': MongoDBJobStore(),
    'default': SQLAlchemyJobStore(url='sqlite:///jobs.sqlite')
}
executors = {
    'default': ThreadPoolExecutor(20),
    'processpool': ProcessPoolExecutor(5)
}
job_defaults = {
    'coalesce': False,
    'max_instances': 3
}
scheduler = BackgroundScheduler(jobstores=jobstores, executors=executors, job_defaults=job_defaults, timezone=utc)
```
方式二、三：略。请看官网

## 启动与关闭调度器：
通过调用start()方法来启动

    scheduler.start()
    scheduler.shutdown()
    scheduler.shutdown(wait=False)

## triggers
内置以下三种触发器实现：
date: 对已仅执行一次的情绪，指定某个之间点。请参考[这里][4]
interval: 指定时间间隔(fixed intervals)周期性执行。请参考[这里][5]
cron: 使用cron风格表达式周期性执行。请参考[这里][6]

# job管理
**添加jobs:**

 - 调用scheduler.add_job()方法，会返回`apscheduler.job.Job`实例(可用于job修改、移除等)
 - 使用装饰器scheduled_job()

**作业存储注意事项：**

除了Memory方式不需要序列化之外(一个例外是使用ProcessPoolExecutor)，其余都需要作业函数参数可序列化。
如果需要存储作业，而且每次启动时你的应用都会重新添加一遍作业，那么请在添加job时指定一个`唯一的ID`，以及指定`replace_existing=True`，否则每次启动应用都会添加一次job的副本。
如果需要立即启动该任务，请在添加job时指定trigger参数。

**移除job:**
调用scheduler.remove_job()放到，参数为 job’s ID and job store alias
调用job实例的remove()方法 on the Job instance you got from add_job()

注意：如果任务已经调度完毕，并且之后也不会再被执行的情况下，会被自动移除。

    job = scheduler.add_job(myfunc, 'interval', minutes=2)
    job.remove()
    
    scheduler.add_job(myfunc, 'interval', minutes=2, id='my_job_id')
    scheduler.remove_job('my_job_id')

**暂停和恢复job：**

    apscheduler.job.Job.pause()
    apscheduler.schedulers.base.BaseScheduler.pause_job()
    
    apscheduler.job.Job.resume()
    apscheduler.schedulers.base.BaseScheduler.resume_job()

**获取jobs列表**

    apscheduler.get_jobs()

**修改job：**

可以通过`apscheduler.job.Job.modify() or apscheduler.modify_job()`修改`除了id之外`的job属性。

    job.modify(max_instances=6, name='Alternate name')

如果你想修改job的调度器，你可以使用`apscheduler.job.Job.reschedule() or reschedule_job()`

```
scheduler.reschedule_job('my_job_id', trigger='cron', minute='*/5')
```

## 限制同一个job实例的并发执行数
默认情况下同一个job，只允许一个job实例运行。这在某个job在下次运行时间到达之后仍未执行完毕时，能达到控制的目的。你也可以该变这一行为，在你调用add_job时可以传递`max_instances`=5来运行同时运行同一个job的5个job实例。

## job错过执行时间与job合并(Missed job executions and coalescing):
一个job可能由于某些情况错过执行时间,比如上一点提到的，或者是线程池或进程池用光了，或者是当要调度job时，突然down机了等。
这时可以通过设置job的misfire_grace_time选项来指示之后尝试执行的次数。
当然如果这不符合你的期望，你可以合并所有错过时间的job到一个job来执行，通过设定job的`coalesce=True`。

# Scheduler events
可以监听调度、任务执行情况相关的事件。

```
def my_listener(event):
    if event.exception:
        print('The job crashed :(')
    else:
        print('The job worked :)')

scheduler.add_listener(my_listener, EVENT_JOB_EXECUTED | EVENT_JOB_ERROR)
```
支持的事件列表：
http://apscheduler.readthedocs.io/en/v3.3.0/modules/events.html#module-apscheduler.events

# 小结
有木有非常强大，又非常易于理解的感觉。Good Job！

  [1]: https://github.com/nvie/rq/
  [2]: http://apscheduler.readthedocs.io/en/v3.3.0/index.html
  [3]: http://apscheduler.readthedocs.io/en/v3.3.0/modules/schedulers/base.html#module-apscheduler.schedulers.base
  [4]: http://apscheduler.readthedocs.io/en/v3.3.0/modules/triggers/date.html#module-apscheduler.triggers.date
  [5]: http://apscheduler.readthedocs.io/en/v3.3.0/modules/triggers/interval.html#module-apscheduler.triggers.interval
  [6]: http://apscheduler.readthedocs.io/en/v3.3.0/modules/triggers/cron.html#module-apscheduler.triggers.cron