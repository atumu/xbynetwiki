title: celery 

#  Celery任务调度 
官网：http://www.celeryproject.org/
文档：http://docs.celeryproject.org/en/latest/index.html
当前版本：3.1,	4.0处于RC版本，改动较大。本文以3.1为基础

pip install Celery

Celery是Python开发的分布式任务调度模块.

rabbitmq是一个采用Erlang写的强大的消息队列工具。在celery中可以扮演broker的角色。那么什么是broker？
broker是一个消息传输的中间件，可以理解为一个邮箱。每当应用程序调用celery的异步任务的时候，会向broker传递消息，而后celery的worker将会取到消息，进行对于的程序执行。好吧，这个邮箱可以看成是一个消息队列。那么什么又是backend，通常程序发送的消息，发完就完了，可能都不知道对方时候接受了。为此，celery实现了一个backend，用于存储这些消息以及celery执行的一些消息和结果。对于 brokers，官方推荐是rabbitmq和redis，至于backend，就是数据库啦。为了简单起见，建议都用redis。

##  快速入门 
  * 选择和安装第三方消息代理（message transport (broker).）
  * 创建task
  * 启动worker，然后调用tasks
  * 追踪任务的状态，检查返回值
Celery本身不含消息服务，它需要使用第三方消息服务来传递任务，目前，Celery支持的消息服务有[RabbitMQ](http://docs.celeryproject.org/en/latest/getting-started/brokers/rabbitmq.html#broker-rabbitmq)、[Redis](http://redis.io/)甚至是[数据库(sqlalchemy)](http://docs.celeryproject.org/en/latest/getting-started/brokers/sqlalchemy.html#broker-sqlalchemy)、[MongoDB](http://docs.celeryproject.org/en/latest/getting-started/brokers/mongodb.html#broker-mongodb)等

主要接口：Celery 

接下来演示RabbitMQ的例子：
```

from celery import Celery

app = Celery('tasks', broker='amqp://guest@localhost//')

@app.task
def add(x, y):
    return x + y

```
Celery的第一个参数是当前模块的名字，第二个参数是消息代理的链接参数BROKER_URL,
###  BROKER_URL 
[详情查询](http://docs.celeryproject.org/en/latest/getting-started/brokers/index.html):

```

  * RabbitMQ : BROKER_URL = 'amqp://guest:guest@localhost:5672//'

  * Redis: BROKER_URL = 'redis://localhost:6379/0'            #redis://:password@hostname:port/db_number

  * MongoDB:BROKER_URL = 'mongodb://localhost:27017/database_name'  #mongodb://userid:password@hostname:port/database_name

  * 如果是RDBMS:
# sqlite (filename)
BROKER_URL = 'sqla+sqlite:///celerydb.sqlite'
# mysql
BROKER_URL = 'sqla+mysql://scott:tiger@localhost/foo'
# oracle
BROKER_URL = 'sqla+oracle://scott:tiger@127.0.0.1:1521/sidname'

```

确保你的消息代理服务器是开着的。

运行 celery worker server
```

$ celery -A tasks worker --loglevel=info

```
当然你也可以使用supervisord等工具在后台运行celery worker server
**-A选项指定celery实例，上述其实是tasks.app tasks.py中的app就是Celery实例。**
帮助：$  celery worker --help， $ celery help
当然你也可以通过代码来启动worker，而不是通过命令行
```

import celery
from celery.bin import worker as celery_worker
  application = celery.current_app._get_current_object()
    c_worker = celery_worker.worker(app=application)
    options = {
        'broker': config.get('CELERY_CONFIG').BROKER_URL,
        'loglevel': 'INFO',
        'traceback': True,
    }
    c_worker.run(**options)

```
接下来就是执行任务了
```

>>> from tasks import add
>>> add.delay(4, 4)  #这里的add，是上面定义的task，delay() 方法是强大的 apply_async() 调用的快捷方式

```
之后这个任务就会被上一步启动的worker执行了。

delay()方法会返回一个AsyncResult实例，通过这个实例可以追踪任务的状态，强制等待任务完成，获取返回值，异常等。**但是默认没有被启用，需要进行result  backend相关的配置才可以使用。**

###  result  backend配置： 
CELERY_RESULT_BACKEND 选项只有在你必须要 Celery 任务的存储状态和运行结果的时候才是必须的。即需要AsyncResult对象的时候。
支持的有：
SQLAlchemy/Django ORM, Memcached, Redis, AMQP (RabbitMQ), and MongoDB – or you can define your own.
[链接配置查询](http://docs.celeryproject.org/en/latest/userguide/tasks.html#task-result-backends)
可以通过 Celery的backend参数或者CELERY_RESULT_BACKEND 配置
例如RPC: app = Celery('tasks', backend='rpc://', broker='amqp://')
```

Redis:CELERY_RESULT_BACKEND = 'redis://:password@host:port/db'   #'redis://localhost/0'

RDBMS:
CELERY_RESULT_BACKEND = 'db+scheme://user:password@host:port/dbname'
# sqlite (filename)
CELERY_RESULT_BACKEND = 'db+sqlite:///results.sqlite'
# mysql
CELERY_RESULT_BACKEND = 'db+mysql://scott:tiger@localhost/foo'
# oracle
CELERY_RESULT_BACKEND = 'db+oracle://scott:tiger@127.0.0.1:1521/sidname'

MongoDB:
CELERY_RESULT_BACKEND = 'mongodb://192.168.1.100:30000/'
CELERY_MONGODB_BACKEND_SETTINGS = {
    'database': 'mydb',
    'taskmeta_collection': 'my_taskmeta_collection',
}

```
配置好result backend之后，我们就可以使用[AsyncResult](http://docs.celeryproject.org/en/latest/reference/celery.result.html#module-celery.result)对象了。如：
```

result = add.delay(4, 4)
>>> result.ready()
False
>>> result.get(timeout=1)
8
>>> result.get(propagate=False)
>>> result.traceback
>>> result.wait()

```
###  配置 
[详情](http://docs.celeryproject.org/en/latest/configuration.html#configuration)
默认使用pickle序列化数据，比如可以改为json
```

app.conf.CELERY_TASK_SERIALIZER = 'json'
app.conf.update(
    BROKER_URL = 'amqp://'
    CELERY_RESULT_BACKEND = 'rpc://'
    CELERY_TASK_SERIALIZER='json',
    CELERY_ACCEPT_CONTENT=['json'],  # Ignore other content
    CELERY_RESULT_SERIALIZER='json',
    CELERY_TIMEZONE='Europe/Oslo',
    CELERY_ENABLE_UTC=True,
)
app.config_from_object('celeryconfig')

```

Celery默认设置就能满足基本要求。Worker以Pool模式启动，默认大小为CPU核心数量，缺省序列化机制是pickle，但可以指定为json。


###  任务调用配置与回调 
task的delay() 方法是强大的 apply_async() 调用的快捷方式
apply_async(args[, kwargs[, …]])
delay(*args, * *kwargs)

例如
T.apply_async( (arg, ), {'kwarg': value})

T.apply_async(countdown=10) 10秒以后执行
T.apply_async(eta=now + timedelta(seconds=10)) executes 10 seconds from now, specifed using eta

T.apply_async(countdown=60, expires=120) 60秒以后执行，120秒之后过期
T.apply_async(expires=now + timedelta(days=2)) expires in 2 days, set using datetime.

```

>>> from datetime import datetime, timedelta
>>> tomorrow = datetime.utcnow() + timedelta(days=1)
>>> add.apply_async((2, 2), eta=tomorrow)

```

**回调Linking (callbacks/errbacks)**
Celery支持将不同的task连接到一起，形成一条链 so that one task follows another.
**The callback task will be applied with the result of the parent task as a partial argument:**
```

add.apply_async((2, 2), link=add.s(16)) #相当于(2+2)+16=20
add.apply_async((2, 2), link=[add.s(16), other_task.s()]) #同时执行多个回调：

```
为什么是add.s?The add.s call used here is** called a subtask**,也就是说将link后的任务add作为之前任务的子任务来执行。

**错误回调link_error**
```

@app.task(bind=True)
def error_handler(self, uuid): #接收一个参数uuid为parent task的id, not the result。
    result = self.app.AsyncResult(uuid)
    print('Task {0} raised exception: {1!r}\n{2!r}'.format(
          uuid, result.result, result.traceback))
 
add.apply_async((2, 2), link_error=error_handler.s())

```
` 在 Celery 装饰器中添加了 bind=True 参数。这个参数告诉 Celery 发送一个 self 参数到我的函数，我能够使用它(self)。 `
###  任务失败重试策略 
celery会自动重试失败的任务(默认重试3次)，但是你可以控制重试次数，或者禁用。
比如禁用重试：add.apply_async( (2, 2), retry=False)
```

add.apply_async((2, 2), retry=True, retry_policy={
    'max_retries': 3, #默认3次，0 or None代表一直重试
    'interval_start': 0,
    'interval_step': 0.2,
    'interval_max': 0.2,
})

```
相关配置项：
CELERY_TASK_PUBLISH_RETRY
CELERY_TASK_PUBLISH_RETRY_POLICY

###  任务状态 
AsyncResult.state   ——The tasks current state.
Possible values includes:
PENDING
The task is waiting for execution.
STARTED
The task has been started.
RETRY
The task is to be retried, possibly because of failure.
FAILURE
The task raised an exception, or has exceeded the retry limit. The result attribute then contains the exception raised by the task.
SUCCESS
The task executed successfully. The result attribute then contains the tasks return value.
###  消息压缩 
配置项：CELERY_MESSAGE_COMPRESSION
```

add.apply_async((2, 2), compression='zlib') #gzip, or bzip2

```
##  路由任务到不同队列 
http://docs.celeryproject.org/en/latest/userguide/calling.html#routing-options
http://docs.celeryproject.org/en/latest/userguide/routing.html#guide-routing
配置选项：CELERY_ROUTES
```

add.apply_async(queue='priority.high')
$ celery -A proj worker -l info -Q celery,priority.high #-Q选项指定队列

```

##  子任务的处理一些函数 
Groups分组
A group calls a list of tasks in parallel, and it returns a special result instance that lets you inspect the results as a group, and retrieve the return values in order.
```

>>> from celery import group
>>> from proj.tasks import add

>>> group(add.s(i, i) for i in xrange(10))().get()
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
Partial group
>>> g = group(add.s(i) for i in xrange(10))
>>> g(10).get()
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]

```
**Chains链式调用**
Tasks can be linked together so that after one task returns the other is called:
```

>>> from celery import chain
>>> from proj.tasks import add, mul

# (4 + 4) * 8
>>> chain(add.s(4, 4) | mul.s(8))().get()
64
or a partial chain:

# (? + 4) * 8
>>> g = chain(add.s(4) | mul.s(8))
>>> g(4).get()
64
Chains can also be written like this:

>>> (add.s(4, 4) | mul.s(8))().get()
64

```
**Chords**
A chord is a group with a callback:

```

>>> from celery import chord
>>> from proj.tasks import add, xsum

>>> chord((add.s(i, i) for i in xrange(10)), xsum.s())().get()
90
A group chained to another task will be automatically converted to a chord:

>>> (group(add.s(i, i) for i in xrange(10)) | xsum.s())().get()
90
Since these primitives are all of the subtask type they can be combined almost however you want, e.g:

>>> upload_document.s(file) | group(apply_filter.s() for filter in filters)

```

##  周期性任务（Schduler、Crontab） 
http://docs.celeryproject.org/en/latest/userguide/periodic-tasks.html
###  Scheduler 
一种常见的需求是每隔一段时间执行一个任务。配置如下
config.py
```

#!/usr/bin/env python
# -*- coding:utf-8 -*-
from __future__ import absolute_import
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/5'
BROKER_URL = 'redis://127.0.0.1:6379/6'
CELERY_TIMEZONE = 'Asia/Shanghai'
from datetime import timedelta
CELERYBEAT_SCHEDULE = {
    'add-every-30-seconds': {
         'task': 'proj.tasks.add',
         'schedule': timedelta(seconds=30),
         'args': (16, 16)
    },
}

```
注意配置文件**需要指定时区**。**这段代码表示每隔30秒执行 add 函数。**
一旦使用了 scheduler,`  启动 celery需要加上-B 参数 `
celery -A proj worker -B -l info

###  crontab 
计划任务当然也可以用crontab实现，celery也有crontab模式。修改 config.py
```

CELERY_TIMEZONE = 'Asia/Shanghai'
from celery.schedules import crontab

CELERYBEAT_SCHEDULE = {
    # Executes every Monday morning at 7:30 A.M
    'add-every-monday-morning': {
        'task': 'tasks.add',
        'schedule': crontab(hour=7, minute=30, day_of_week=1),
        'args': (16, 16),
    },
}

```
**总而言之，scheduler的切分度更细，可以精确到秒。crontab模式就不用说了。**
当然celery还有更高级的用法，比如多个机器使用，启用多个worker并发处理等。
语法规则：
crontab()	Execute every minute.
crontab(minute=0, hour=0)	Execute daily at midnight.
crontab(minute=0, hour='*/3')	Execute every three hours: midnight, 3am, 6am, 9am, noon, 3pm, 6pm, 9pm.
crontab(minute=0,hour='0,3,6,9,12,15,18,21') Same as previous.
crontab(minute='*/15')	Execute every 15 minutes.
crontab(day_of_week='sunday')	Execute every minute (!) at Sundays.
crontab(minute='*',hour='*', day_of_week='sun') Same as previous.
crontab(minute='*/10',hour='3,17,22', day_of_week='thu,fri')
Execute every ten minutes, but only between 3-4 am, 5-6 pm and 10-11 pm on Thursdays or Fridays.
crontab(minute=0, hour='*/2,*/3')	Execute every even hour, and every hour divisible by three. This means: at every hour except: 1am, 5am, 7am, 11am, 1pm, 5pm, 7pm, 11pm
crontab(minute=0, hour='*/5')	Execute hour divisible by 5. This means that it is triggered at 3pm, not 5pm (since 3pm equals the 24-hour clock value of “15”, which is divisible by 5).
crontab(minute=0, hour='*/3,8-17')	Execute every hour divisible by 3, and every hour during office hours (8am-5pm).
crontab(0, 0, day_of_month='2')	Execute on the second day of every month.
crontab(0, 0,day_of_month='2-30/3')
Execute on every even numbered day.
crontab(0, 0,day_of_month='1-7,15-21')
Execute on the first and third weeks of the month.
crontab(0, 0, day_of_month='11',month_of_year='5')
Execute on 11th of May every year.
crontab(0, 0,month_of_year='*/3')
Execute on the first month of every quarter.
[更多规则](http://docs.celeryproject.org/en/latest/reference/celery.schedules.html#celery.schedules.crontab)
##  Flask中使用Celery 
Celery 是一个异步任务队列/基于分布式消息传递的作业队列.Flask 与 Celery 整合是十分简单，不需要任何插件。
总的想法就是你的应用程序可能需要执行任何消耗资源的任务都可以交给任务队列，让你的应用程序自由和快速地响应客户端请求。
使用 Celery 运行后台任务并不像在线程中这样做那么简单。但是好处多多，Celery 具有分布式架构，使你的应用易于扩展。
一个 Celery 安装有三个核心组件：
  * Celery 客户端: 用于发布后台作业。当与 Flask 一起工作的时候，客户端即为 Flask 应用。
  * Celery workers: 这些是运行后台作业的进程。Celery 支持本地和远程的 workers，因此你就可以在 Flask 服务器上启动一个单独的 worker，随后随着你的应用需求的增加而新增更多的 workers。
  * 消息代理: 客户端通过消息队列和 workers 进行通信，Celery 支持多种方式来实现这些队列。最常用的代理就是 RabbitMQ 和 Redis。小型应用也可以直接使用DB

这就是所有在 Flask 中整合 Celery 必要步骤:
```

from celery import Celery

def make_celery(app):
    celery = Celery(app.import_name, broker=app.config['CELERY_BROKER_URL'])
    celery.conf.update(app.config)
    TaskBase = celery.Task
    class ContextTask(TaskBase):
        abstract = True
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return TaskBase.__call__(self, *args, **kwargs)
    celery.Task = ContextTask
    return celery

```
示例：
```

rom flask import Flask

app = Flask(__name__)
app.config.update(
    CELERY_BROKER_URL='redis://localhost:6379',
    CELERY_RESULT_BACKEND='redis://localhost:6379'
)
celery = make_celery(app)


@celery.task()
def add_together(a, b):
    return a + b

```
然后需要先运行一个worker来处理即将到来的任务
```

$ celery -A your_application worker

```
这个任务现在能够后台运行：
```

>>> result = add_together.delay(23, 42)
>>> result.wait()
65

```
###  简单例子：异步发送邮件 
非常普通的需求：能够发送邮件但是不阻塞主应用。
```

<html>
  <head>
    <title>Flask + Celery Examples</title>
  </head>
  <body>
    <h1>Flask + Celery Examples</h1>
    <h2>Example 1: Send Asynchronous Email</h2>
    {% for message in get_flashed_messages() %}
    <p style="color: red;">![](/data/dokuwiki message )</p>
    {% endfor %}
    <form method="POST">
      <p>Send test email to: <input type="text" name="email" value="![](/data/dokuwiki email )"></p>
      <input type="submit" name="submit" value="Send">
      <input type="submit" name="submit" value="Send in 1 minute">
    </form>
  </body>
</html>

```
Flask-Mail 扩展需要一些配置
注意为了避免我的账号丢失的风险，**我将其设置在系统的环境变量**，这是我从应用中导入的。
```

# Flask-Mail configuration
app.config['MAIL_SERVER'] = 'smtp.googlemail.com'
app.config['MAIL_PORT'] = 587
app.config['MAIL_USE_TLS'] = True
app.config['MAIL_USERNAME'] = os.environ.get('MAIL_USERNAME')
app.config['MAIL_PASSWORD'] = os.environ.get('MAIL_PASSWORD')
app.config['MAIL_DEFAULT_SENDER'] = 'flask@example.com'

```
```

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'GET':
        return render_template('index.html', email=session.get('email', ''))
    email = request.form['email']
    session['email'] = email

    # send the email
    msg = Message('Hello from Flask',
                  recipients=[request.form['email']])
    msg.body = 'This is a test email sent from a background Celery task.'
    if request.form['submit'] == 'Send':
        # send right away
        send_async_email.delay(msg)
        flash('Sending email to {0}'.format(email))
    else:
        # send in one minute
        send_async_email.apply_async(args=[msg], countdown=60)
        flash('An email will be sent to {0} in one minute'.format(email))

    return redirect(url_for('index'))
  
@celery.task
def send_async_email(msg):
    """Background task to send an email with Flask-Mail."""
    with app.app_context():
        mail.send(msg)

```
###  复杂例子：显示状态更新和结果 
上面的示例过于简单，后台作业启动然后应用忘记它。但是事实上许多应用程序有必要监控它的后台任务并且获取运行结果。
这个示例展示一个虚构的长时间运行的任务。用户点击按钮启动一个或者更多的长时间运行的任务，在浏览器上的页面使用 ajax 轮询服务器更新所有任务的状态。每一个任务，页面都会显示一个图形的状态栏，进度条，一个状态消息，并且当任务完成的时候，也会显示任务的执行结果。
```

@celery.task(bind=True)
def long_task(self):
    """Background task that runs a long function with progress reports."""
    verb = ['Starting up', 'Booting', 'Repairing', 'Loading', 'Checking']
    adjective = ['master', 'radiant', 'silent', 'harmonic', 'fast']
    noun = ['solar array', 'particle reshaper', 'cosmic ray', 'orbiter', 'bit']
    message = ''
    total = random.randint(10, 50)
    for i in range(total):
        if not message or random.random() < 0.25:
            message = '{0} {1} {2}...'.format(random.choice(verb),
                                              random.choice(adjective),
                                              random.choice(noun))
        self.update_state(state='PROGRESS',
                          meta={'current': i, 'total': total,
                                'status': message})
        time.sleep(1)
    return {'current': 100, 'total': 100, 'status': 'Task completed!',
            'result': 42}
  
  
@app.route('/longtask', methods=['POST'])
def longtask():
    task = long_task.apply_async()
    return jsonify({}), 202, {'Location': url_for('taskstatus',task_id=task.id)}

```
` 在 Celery 装饰器中添加了 bind=True 参数。这个参数告诉 Celery 发送一个 self 参数到我的函数，我能够使用它(self)来记录状态更新。 `
self.update_state() 调用是 Celery 如何接受这些任务更新**。有一些内置的状态，比如 STARTED, SUCCESS 等等，但是 Celery 也支持自定义状态**。这里我使用一个叫做 PROGRESS 的自定义状态。连同状态，还有一个附件的元数据，该元数据是 Python 字典形式


正如你所见，客户端需要发起一个 POST 请求到 /longtask 来掀开这些任务中的一个的序幕。服务器启动任务，并且存储返回值。
**对于响应我使用状态码 202，这个状态码通常是在 REST APIs 中使用用来表明一个请求正在进行中。**
我也**添加了 Location 头，值为一个客户端用来获取状态信息的 URL。**这个 URL 指向另一个叫做 taskstatus 的 Flask 路由，并且有 task.id 作为动态的要素。
从 Flask 应用中访问任务状态
```

@app.route('/status/<task_id>')
def taskstatus(task_id):
    task = long_task.AsyncResult(task_id)
    if task.state == 'PENDING':
        // job did not start yet
        response = {
            'state': task.state,
            'current': 0,
            'total': 1,
            'status': 'Pending...'
        }
    elif task.state != 'FAILURE':
        response = {
            'state': task.state,
            'current': task.info.get('current', 0),
            'total': task.info.get('total', 1),
            'status': task.info.get('status', '')
        }
        if 'result' in task.info:
            response['result'] = task.info['result']
    else:
        # something went wrong in the background job
        response = {
            'state': task.state,
            'current': 1,
            'total': 1,
            'status': str(task.info),  # this is the exception raised
        }
    return jsonify(response)

```
  
##  Celery使用最佳实践 
**1、不要使用数据库作为你的AMQP Broker**
某一天，你发现因为太多的任务产生，4个worker不够用了，处理任务的速度已经大大落后于生产任务的速度，于是你不停去增加worker的数量。突然，你的数据库因为大量进程轮询任务而变得响应缓慢，磁盘IO一直处于高峰值状态，你的web应用也开始受到影响。这一切，都因为workers在不停地对数据库进行DDOS。而当你使用一个合适的AMQP（譬如RabbitMQ）的时候，这一切都不会发生

**2，使用更多的queue（不要只用默认的）**
Celery非常容易设置，通常它会使用默认的queue用来存放任务（除非你显示指定其他queue）。通常写法如下：
```

@app.task()
def my_taskA(a, b, c):
    print("doing something here...")
@app.task()
def my_taskB(x, y):
    print("doing something here...")

```
这两个任务都会在同一个queue里面执行，这样写其实很有吸引力的，因为你只需要使用一个decorator就能实现一个异步任务。
作者关心的是**taskA和taskB没准是完全两个不同的东西，或者一个可能比另一个更加重要，那么为什么要把它们放到一个篮子里面呢？**（鸡蛋都不能放到一个篮子里面，是吧！）**没准taskB其实不怎么重要，但是量太多，以至于重要的taskA反而不能快速地被worker进行处理**。
增加workers也解决不了这个问题，因为taskA和taskB仍然在一个queue里面执行。

**3、使用具有优先级的workers**
为了解决2里面出现的问题，我们需要让taskA在一个队列Q1，而taskB在另一个队列Q2执行。同时指定x workers去处理队列Q1的任务，然后使用其它的workers去处理队列Q2的任务。**使用这种方式，taskB能够获得足够的workers去处理，同时一些优先级workers也能很好地处理taskA而不需要进行长时间的等待。**
首先手动**定义queue**
```

CELERY_QUEUES = (
    Queue('default', Exchange('default'), routing_key='default'),
    Queue('for_task_A', Exchange('for_task_A'), routing_key='for_task_A'),
    Queue('for_task_B', Exchange('for_task_B'), routing_key='for_task_B'),
)

```
然后**定义routes**用来决定不同的任务去哪一个queue
```

CELERY_ROUTES = {
    'my_taskA': {'queue': 'for_task_A', 'routing_key': 'for_task_A'},
    'my_taskB': {'queue': 'for_task_B', 'routing_key': 'for_task_B'},
}

```
最后**再为每个task启动不同的workers**
```

celery worker -E -l INFO -n workerA -Q for_task_A 
celery worker -E -l INFO -n workerB -Q for_task_B

```
在我们项目中，会涉及到大量文件转换问题，有大量小于1mb的文件转换，同时也有少量将近20mb的文件转换，小文件转换的优先级是最高的，同时不用占用很多时间，但大文件的转换很耗时。如果将转换任务放到一个队列里面，那么很有可能因为出现转换大文件，导致耗时太严重造成小文件转换延时的问题。
**所以我们按照文件大小设置了3个优先队列，并且每个队列设置了不同的workers，很好地解决了我们文件转换的问题。**

**4、使用Celery的错误处理机制**
大多数任务并没有使用错误处理，如果任务失败，那就失败了。在一些情况下这很不错，但是作者见到的多数失败任务都是去调用第三方API然后出现了网络错误，或者资源不可用这些错误，而对于这些错误，最简单的方式就是重试一下，也许就是第三方API临时服务或者网络出现问题，没准马上就好了，那么为什么不试着重试一下呢？
```

@app.task(bind=True, default_retry_delay=300, max_retries=5)
def my_task_A():
    try:
        print("doing stuff here...")
    except SomeNetworkException as e:
        print("maybe do some clenup here....")
        self.retry(e)

```
作者喜欢给每一个任务定义一个等待多久重试的时间，以及**最大的重试次数**。当然还有更详细的参数设置，自己看文档去。
对于错误处理，我们因为使用场景特殊，例如一个文件转换失败，那么无论多少次重试都会失败，所以没有加入重试机制。

**5、使用Flower监控celery的tasks和works**
[Flower](http://celery.readthedocs.io/en/latest/userguide/monitoring.html#flower-real-time-celery-web-monitor)是一个非常强大的工具，用来监控celery的tasks和works。当然你也可以使用命令行，或者检查对应的消息代理。[具体x详情](http://celery.readthedocs.io/en/latest/userguide/monitoring.html)
```

$ pip install flower
三种启动方式
$ celery -A proj flower #will start a web-server that you can visit:  default port is http://localhost:5555
$ celery -A proj flower --port=5555 
$ celery flower --broker=amqp://guest:guest@localhost:5672// # $ celery flower --broker=amqp://guest:guest@localhost:5672//

``` 

**6，没事别太关注任务退出状态**
一个任务状态就是该任务结束的时候成功还是失败信息，没准在一些统计场合，这很有用。但我们需要知道，任务退出的状态并不是该任务执行的结果，该任务执行的一些结果因为会对程序有影响，通常会被写入数据库（例如更新一个用户的朋友列表）。
作者见过的多数项目都将任务结束的状态存放到sqlite或者自己的数据库，但是存这些真有必要吗，没准可能影响到你的web服务的，所以作者通常设置CELERY_IGNORE_RESULT = True去丢弃。
对于我们来说，因为是异步任务，知道任务执行完成之后的状态真没啥用，所以果断丢弃。

**7，不要给任务传递 Database/ORM的实体对象**
这个其实就是不要传递Database对象**（例如一个用户的实例）**给任务，因为没准序列化之后的数据已经是**过期的数据**了。所以最好还是直接传递一个user id，然后在任务执行的时候实时的从数据库获取。
对于这个，我们也是如此，给任务只传递相关id数据，譬如文件转换的时候，我们只会传递文件的id，而其它文件信息的获取我们都是直接通过该id从数据库里面取得。
其余参考：http://www.liaoxuefeng.com/article/00137760323922531a8582c08814fb09e9930cede45e3cc000
http://www.pythondoc.com/flask-celery/first.html
http://blog.csdn.net/siddontang/article/details/34447003
http://www.jianshu.com/p/1840035cb510