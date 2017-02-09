title: python学习笔记031 

#  Python学习笔记之Flask日志 
##  错误邮件 
Flask 使用 Python 内置的日志系统，而且它确实向你发送你可能需要的错误邮件。 这里给出你如何配置 Flask 日志记录器向你发送报告异常的邮件:
```

ADMINS = ['yourname@example.com']
if not app.debug:
    import logging
    from logging.handlers import SMTPHandler
    mail_handler = SMTPHandler('127.0.0.1',
                               'server-error@example.com',
                               ADMINS, 'YourApplication Failed')
    mail_handler.setLevel(logging.ERROR)
    app.logger.addHandler(mail_handler)

```
那么刚刚发生了什么？我们创建了一个新的 SMTPHandler 来用监听 127.0.0.1 的邮件服务器向所有的 ADMINS 发送发件人为 server-error@example.com ，主题为 “YourApplication Failed” 的邮件。如果你的邮件服务器需要凭证，这些功能也被提供了。详情请见 SMTPHandler 的文档。
我们同样告诉处理程序只发送错误和更重要的消息。因为我们的确不想收到警告或是其它没用的，每次请求处理都会发生的日志邮件。

##  记录到文件 
  * FileHandler - 在文件系统上记录日志
  * RotatingFileHandler - 在文件系统上记录日志， 并且当消息达到一定数目时，会滚动记录
  * NTEventLogHandler - 记录到 Windows 系统中的系统事件日志。如果你在 Windows 上做开发，这就是你想要用的。
  * SysLogHandler - 发送日志到 Unix 的系统日志
当你选择了日志处理程序，像前面对 SMTP 处理程序做的那样，只要确保使用一个低级的设置（我推荐 WARNING ）:
```

if not app.debug:
    import logging
    from themodule import TheHandlerYouWant
    file_handler = TheHandlerYouWant(...)
    file_handler.setLevel(logging.WARNING)
    app.logger.addHandler(file_handler)

```
##  控制日志格式 
这里有一些配置实例:
邮件
```

from logging import Formatter
mail_handler.setFormatter(Formatter('''
Message type:       %(levelname)s
Location:           %(pathname)s:%(lineno)d
Module:             %(module)s
Function:           %(funcName)s
Time:               %(asctime)s

Message:

%(message)s
'''))

```
日志文件
```

from logging import Formatter
file_handler.setFormatter(Formatter(
    '%(asctime)s %(levelname)s: %(message)s '
    '[in %(pathname)s:%(lineno)d]'
))

```
##  复杂日志格式 
这里给出一个用于格式化字符串的格式变量列表。注意这个列表并不完整，**完整的列表请翻阅 logging 包的官方文档。**
格式	描述
  * %(levelname)s	消息文本的记录等级 ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL').
  * %(pathname)s	发起日志记录调用的源文件的完整路径（如果可用）
  * %(filename)s	路径中的文件名部分
  * %(module)s	模块（文件名的名称部分）
  * %(funcName)s	包含日志调用的函数名
  * %(lineno)d	日志记录调用所在的源文件行的行号（如果可用）
  * %(asctime)s	LogRecord 创建时的人类可读的时间。默认情况下，格式为 "2003-07-08 16:49:45,896" （逗号后的数字时间的毫秒部分）。这可以通过继承 :class:~logging.Formatter，并重载 formatTime() 改变。
  * %(message)s	记录的消息，视为 msg % args

**如果你想深度定制日志格式，你可以继承 Formatter 。 Formatter 有三个需要关注的方法:**
  * format():处理实际上的格式。需要一个 LogRecord 对象作为参数，并必须返回一个格式化字符串。
  * formatTime():控制 asctime 格式。如果你需要不同的时间格式，可以重载这个函数。
  * formatException()控制异常的格式。需要一个 exc_info 元组作为参数，并必须返 回一个字符串。默认的通常足够好，你不需要重载它。更多信息请见其官方文档。

###  整合其它的库日志记录 
至此，我们只配置了应用自己建立的日志记录器。其它的库也可以记录它们。例如， SQLAlchemy 在它的核心中大量地使用日志。而在 logging 包中有一个方法可以一次性配置所有的日志记录器，我不推荐使用它。可能存在一种情况，当你想要在同一个 Python 解释器中并排运行多个独立的应用时，则不可能对它们的日志记录器做不同的设置。
作为替代，我推荐你找出你有兴趣的日志记录器，**用 getLogger() 函数来获取日志记录器，并且遍历它们来附加处理程序:**
```

from logging import getLogger
loggers = [app.logger, getLogger('sqlalchemy'),
           getLogger('otherlibrary')]
for logger in loggers:
    logger.addHandler(mail_handler)
    logger.addHandler(file_handler)

```

