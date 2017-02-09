modifyAt:2016-12-18 14:23:22
title:Flask-WTF插件
createAt:2016-12-18 14:23:22
location:dokuwiki/python/flask-wtf
author:xbynet

#  Flask-WTF表单插件 
Flask-WTF 提供了简单的 [WTForms](http://wtforms.simplecodes.com/docs/) 集成。
官网:http://flask-wtf.readthedocs.io/en/stable/
当前版本：Version 0.13.1
特性

  * 与 WTForms 的集成。
  * 带有 CSRF 令牌的安全表单。
  * 全局的 CSRF 保护。
  * Recaptcha 支持。
  * 支持 Flask-Uploads 的文件上传。
  * 国际化集成。

pip install Flask-WTF

获取源码
git clone git:/ /github.com/lepture/flask-wtf.git

##  快速入门 
```

from flask_wtf import FlaskForm
from wtforms import StringField
from wtforms.validators import DataRequired

class MyForm(FlaskForm):
    name = StringField('name', validators=[DataRequired()])

```
CSRF token hidden field 会被自动创建.你可以在模板中这样使用
```

<form method="POST" action="/">
    {{ form.csrf_token }}
    {{ form.name.label }} {{/data/dokuwiki form.name(size=20) }}
    <input type="submit" value="Go">
</form>

```
如果存在多个隐藏域，可以通过hidden_tag()来直接一次性渲染所有的，而不必一一那么做。
```

<form method="POST" action="/">
    ![](/data/dokuwiki form.hidden_tag() )
    ![](/data/dokuwiki form.name.label ) ![](/data/dokuwiki form.name(size=20) )
    <input type="submit" value="Go">
</form>

```

##  表单验证 
```

@app.route('/submit', methods=('GET', 'POST'))
def submit():
    form = MyForm()
    if form.validate_on_submit():
        return redirect('/success')
    return render_template('submit.html', form=form)

```
**注意你不需要把 request.form 传递给 Flask-WTF，它会自动加载。并且 validate_on_submit 会便捷地检查该请求是否是一个 POST 请求以及是否有效。**

##  文件上传 
Flask-WTF 提供了 FileField 类来处理文件上传，它在表单提交后， 自动从 flask.request.files 抽取数据。
FileField` 的 data 属性是一个 Werkzeug FileStorage 实例。
```

from werkzeug.utils import secure_filename
from flask_wtf.file import FileField

class PhotoForm(FlaskForm):
    photo = FileField('Your photo')

@app.route('/upload/', methods=('GET', 'POST'))
def upload():
    form = PhotoForm()
    if form.validate_on_submit():
        filename = secure_filename(form.photo.data.filename)
        form.photo.data.save('uploads/' + filename)
    else:
        filename = None
    return render_template('upload.html', form=form, filename=filename)

```
此外，Flask-WTF 支持文件上传的验证。提供了 FileRequired 和 FileAllowed 。FileAllowed 可与 Flask-Uploads 协同工作
```

from flask.ext.uploads import UploadSet, IMAGES
from flask_wtf import Form
from flask_wtf.file import FileField, FileAllowed, FileRequired

images = UploadSet('images', IMAGES)

class UploadForm(Form):
    upload = FileField('image', validators=[
        FileRequired(),
        FileAllowed(images, 'Images only!')
    ])

```
它也可以在没有 Flask-Uploads 的情况下工作。你需要向 FileAllowed 传递扩展名:
```

class UploadForm(Form):
    upload = FileField('image', validators=[
        FileRequired(),
        FileAllowed(['jpg', 'png'], 'Images only!')
    ])

```
##  google在线验证码(Recaptcha)(国情还是算了) 
RecaptchaField
```

from flask_wtf import FlaskForm, RecaptchaField
from wtforms import TextField

class SignupForm(FlaskForm):
    username = TextField('Username')
    recaptcha = RecaptchaField()

```
配置选项
RECAPTCHA_PUBLIC_KEY	required A public key.
RECAPTCHA_PRIVATE_KEY	required A private key.
RECAPTCHA_API_SERVER	optional Specify your Recaptcha API server.
RECAPTCHA_PARAMETERS	optional A dict of JavaScript (api.js) parameters.
RECAPTCHA_DATA_ATTRS	optional A dict of data attributes options. https://developers.google.com/recaptcha/docs/display
例子：
RECAPTCHA_PARAMETERS = {'hl': 'zh', 'render': 'explicit'}
RECAPTCHA_DATA_ATTRS = {'theme': 'dark'}

` 如果是测试环境，recaptcha field会自动通过验证。 `**if app.testing is True, recaptcha field will always be valid for you convenience.**
在模板中使用：
```

<form action="/" method="post">
    ![](/data/dokuwiki form.username )
    ![](/data/dokuwiki form.recaptcha )
</form>

```
更多例子：https://github.com/lepture/flask-wtf/tree/master/examples/recaptcha

##  CSRF 保护 
为什么有 CSRF
Flask-WTF 表单保护你免受 CSRF 威胁，你不需要有任何担心。
尽管如此，**如果你有不包含表单的视图，那么它们仍需要额外的保护。**
**例如，由 AJAX 发送的 POST 请求，并没有通过表单。**
要对所有视图函数启用 CSRF 保护，你需要启用 **CsrfProtect** 模块:
```

from flask_wtf.csrf import CsrfProtect

CsrfProtect(app)

```
与任何其它的 Flask 扩展一样，你可以惰性加载它:
```

from flask_wtf.csrf import CsrfProtect

csrf = CsrfProtect()
def create_app():
    app = Flask(__name__)
    csrf.init_app(app)

```
你需要为 CSRF 指定一个密钥。通常，这与你的 Flask 应用 SECRET_KEY 一致。
如果模板中有表单，你不需要做任何事。与之前一样:
```

<form method="post" action="/">
    {{ form.csrf_token }}
</form>

```
但如果模板中没有表单，你仍需要 CSRF 令牌:
```

<form method="post" action="/">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}" />
</form>

```
无论何时未通过 CSRF 验证，都会返回 400 响应。你可以自定义这个错误响应:
```

@csrf.error_handler
def csrf_error(reason):
    return render_template('csrf_error.html', reason=reason), 400

```
我们强烈建议你对所有视图启用 CSRF 保护。**但也提供了将某些视图函数除外的途径**:
```

@csrf.exempt
@app.route('/foo', methods=('GET', 'POST'))
def my_handler():
    # ...
    return 'ok'

```
你也可以排除一个blueprint中的所有视图函数，如下：
csrf.exempt(account_blueprint)

###  AJAX CSRF
```

CsrfProtect(app)

```
假设你已经使用了 CsrfProtect(app) ，你可以通过 ![](/data/dokuwiki csrf_token() ) 获取 CSRF 令牌。这个方法在每个模板中都可以使用，你并不需要担心在没有表单时如何渲染 CSRF 令牌字段。
我们推荐的方式是在 < meta> 标签中渲染 CSRF 令牌:
```

<meta name="csrf-token" content="{{ csrf_token() }}">

```
在 < script> 标签中渲染同样可行:
```

<script type="text/javascript">
    var csrftoken = "{{ csrf_token() }}"
</script>

```
下面的例子采用了 < meta> 的方式， < script> 会更简单，你无须担心没有对应的例子。

无论何时你发送 AJAX POST 请求，**为其添加 X-CSRFToken 标头**:
```

var csrftoken = $('meta[name=csrf-token]').attr('content')
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type)) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken)
        }
    }
})

```

##  配置选项 
表单与 CSRF

  * WTF_CSRF_ENABLED	Disable/enable CSRF protection for forms. Default is** True**.
  * WTF_CSRF_CHECK_DEFAULT	Enable CSRF checks for all views by default. Default is** True.**
  * WTF_I18N_ENABLED	Disable/enable I18N support. This should work together with Flask-Babel. Default is** True.**
  * WTF_CSRF_HEADERS	CSRF token HTTP headers checked. Default is **['X-CSRFToken', 'X-CSRF-Token']**
  * WTF_CSRF_SECRET_KEY	A random string for generating CSRF token. **Default is the same as SECRET_KEY.**
  * WTF_CSRF_TIME_LIMIT	CSRF token expiring time. Default is 3600 seconds. If set to None, the CSRF token is then bound to the life-time of the session.
  * WTF_CSRF_SSL_STRICT	Strictly protection on SSL. This will check the referrer, validate if it is from the same origin. **Default is True.**
  * WTF_CSRF_METHODS	CSRF protection on these request methods. Default is **['POST', 'PUT', 'PATCH']**

##  字段类 
每个 Web 表单都由一个继承自 **Form** 的类表示。
这个类定义表单中的一组字段（表单的内容与类的字段是一一对应的）,每个字段都用对象（表单中的对象）表示。字段对象可附属一个或多个验证函数。

  * StringField	文本字段
  * TextAreaField	多行文本字段
  * PasswordField	密码文本字段
  * HiddenField	隐藏文本字段
  * DateField	文本字段,值为 datetime.date 格式
  * DateTimeField	文本字段,值为 datetime.datetime 格式
  * IntegerField	文本字段,值为整数
  * DecimalField	文本字段,值为 decimal.Decimal
  * FloatField	文本字段,值为浮点数
  * BooleanField	复选框,值为 True 和 False
  * RadioField	一组单选框
  * SelectField	下拉列表
  * SelectMultipleField	下拉列表,可选择多个值
  * FileField	文件上传字段
  * SubmitField	表单提交按钮
  * FormField	把表单作为字段嵌入另一个表单
  * FieldList	一组指定类型的字段

##  验证函数 

  * Email	验证电子邮件地址
  * EqualTo	比较两个字段的值;常用于要求输入两次密码进行确认的情况
  * IPAddress	验证 IPv4 网络地址
  * Length	验证输入字符串的长度
  * NumberRange	验证输入的值在数字范围内
  * Optional	无输入值时跳过其他验证函数
  * Required	确保字段中有数据
  * Regexp	使用正则表达式验证输入值
  * URL		验证 URL
  * AnyOf	确保输入值在可选值列表中
  * NoneOf	确保输入值不在可选值列表中
