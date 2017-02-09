title: flask-mail 

#  flask-mail插件 
官网：https://pythonhosted.org/Flask-Mail/
中文：http://www.pythondoc.com/flask-mail/index.html
当前版本0.9.1

pip install Flask-Mail

##  配置 Flask-Mail 
Flask-Mail 使用标准的 Flask 配置 API 进行配置。下面这些是可用的配置型(每一个将会在文档中进行解释):
  * MAIL_SERVER : 默认为 ‘localhost’
  * MAIL_PORT : 默认为 25
  * MAIL_USE_TLS : 默认为 False
  * MAIL_USE_SSL : 默认为 False
  * MAIL_DEBUG : 默认为 app.debug
  * MAIL_USERNAME : 默认为 None
  * MAIL_PASSWORD : 默认为 None
  * MAIL_DEFAULT_SENDER : 默认为 None
  * MAIL_MAX_EMAILS : 默认为 None
  * MAIL_SUPPRESS_SEND : 默认为 app.testing
  * MAIL_ASCII_ATTACHMENTS : 默认为 False
```

from flask import Flask
from flask_mail import Mail

app = Flask(__name__)
mail = Mail(app)

```
或者你也可以在应用程序配置的时候设置你的 Mail 实例，通过使用 init_app 方法:
```

mail = Mail()
app = Flask(__name__)
mail.init_app(app)

```
##  发送邮件 
为了能够发送邮件，首先需要创建一个 Message 实例:
```

from flask_mail import Message
@app.route("/")
def index():

    msg = Message("Hello",
                  sender="from@example.com",
                  recipients=["to@example.com"])

```
你能够设置一个或者多个收件人:
msg.recipients = ["you@example.com"]
msg.add_recipient("somebodyelse@example.com")
如果你设置了 MAIL_DEFAULT_SENDER，就不必再次填写发件人，默认情况下将会使用配置项的发件人:
msg = Message("Hello",recipients=["to@example.com"])
**如果 sender 是一个二元组，它将会被分成姓名和邮件地址:**
```

msg = Message("Hello",sender=("Me", "me@example.com"))

```
assert msg.sender == "Me <me@example.com>"
邮件内容可以包含主体以及/或者 HTML:
```

msg.body = "testing"
msg.html = "<b>testing</b>"

```
最后，发送邮件的时候请使用 Flask 应用设置的 Mail 实例:
```

mail.send(msg)

```
##  大量邮件 
通常在一个 Web 应用中每一个请求会同时发送一封或者两封邮件。在某些特定的场景下，有可能会发送数十或者数百封邮件，不过这种发送工作会给交离线任务或者脚本执行。
```

with mail.connect() as conn:
    for user in users:
        message = '...'
        subject = "hello, %s" % user.name
        msg = Message(recipients=[user.email],
                      body=message,
                      subject=subject)

        conn.send(msg)

```
与电子邮件服务器的连接会一直保持活动状态直到所有的邮件都已经发送完成后才会关闭（断开）。
有些邮件服务器会限制一次连接中的发送邮件的上限。你可以设置重连前的发送邮件的最大数，通过配置 MAIL_MAX_EMAILS 。

##  附件 
在邮件中添加附件同样非常简单:
```

with app.open_resource("image.png") as fp:
    msg.attach("image.png", "image/png", fp.read())

```
具体细节请参看 API 。
##  单元测试以及禁止发送邮件 
当在单元测试中，或者在一个开发环境中，能够禁止邮件发送是十分有用的。
如果设置项 TESTING 设置成 True或MAIL_SUPPRESS_SEND 为 True，emails 将会被禁止发送。调用 send() 发送邮件实际上不会有任何邮件被发送。
为了能够追踪发送邮件的“轨迹”，可以使用 record_messages 方法:
```

with mail.record_messages() as outbox:
    mail.send_message(subject='testing',
                      body='test',
                      recipients=emails)

    assert len(outbox) == 1
    assert outbox[0].subject == "testing"

```
outbox 是一个 发送 Message 实例的列表。
为了使得上述代码能够正常运行，必须安装 **blinker** 包。

##  信号量 
email_dispatched信号，只要邮件被调度，信号就会发送(即使邮件没有真正的发送，例如，在测试环境中)。
订阅 email_dispatched 信号的函数**使用 Message 实例作为第一参数**，Flask app 实例作为一个可选的参数:
```

def log_message(message, app):
    app.logger.debug(message.subject)

email_dispatched.connect(log_message)

```