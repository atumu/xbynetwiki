modifyAt:2016-12-18 14:34:30
title:FlaskWeb开发学习笔记3Web表单 
createAt:2016-12-18 12:58:13
location:dokuwiki/python/python_flask学习笔记3
author:xbynet

以前介绍过request.form能够获取POST请求中提交的表单数据
但是Flask-WTF扩展可以把处理Web表单的过程变成一种愉悦的体验。这个扩展对独立的WTForms包进行了包装以便于集成到Flask程序中。
Flask-WTF安装:`pip install flask-wtf`

建议参考[flask-wtf](http://wiki.xby1993.net/pages/dokuwiki/python/flask-wtf)和[wtforms](http://wiki.xby1993.net/pages/dokuwiki/python/wtforms)

#  跨站请求伪造(CSRF)保护 
一.CSRF是什么？
CSRF（Cross-site request forgery），中文名称：跨站请求伪造，也被称为：one click attack/session riding，缩写为：CSRF/XSRF。
二.CSRF可以做什么？
你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。
四.CSRF的原理
下图简单阐述了CSRF攻击的思想：
![](/data/dokuwiki/python/pasted/20160330-003105.png)
**　从上图可以看出，要完成一次CSRF攻击，受害者必须依次完成两个步骤：**
　　1.登录受信任网站A，并在本地生成Cookie。
　　2.在不登出A的情况下，访问危险网站B。
　　3.上图中所谓的攻击网站，可能是一个存在其他漏洞的可信任的经常被人访问的网站。
　　上面大概地讲了一下CSRF攻击的思想，下面我将用几个例子详细说说具体的CSRF攻击，这里我以一个银行转账的操作作为例子（仅仅是例子，真实的银行网站没这么傻:>）
**　　示例1：**
　　银行网站A，它以GET请求来完成银行转账的操作，如：http://www.mybank.com/Transfer.php?toBankId=11&money=1000
　　危险网站B，它里面有一段HTML的代码如下：
`　　<img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>`
　　首先，你登录了银行网站A，然后访问危险网站B，噢，这时你会发现你的银行账户少了1000块......
　　为什么会这样呢？原因是**银行网站A违反了HTTP规范，使用GET请求更新资源。**在访问危险网站B的之前，你已经登录了银行网站A，**而B中的< img>以GET的方式请求第三方资源**（这里的第三方就是指银行网站了，原本这是一个合法的请求，但这里被不法分子利用了），所以你的浏览器会带上你的银行网站A的Cookie发出Get请求，去获取资源http://www.mybank.com/Transfer.php?toBankId=11&money=1000” 结果银行网站服务器收到请求后，认为这是一个更新资源操作（转账操作），所以就立刻进行转账操作......

** 示例2：**
　　为了杜绝上面的问题，银行决定改用POST请求完成转账操作。
　　银行网站A的WEB表单如下：　　
  
```
　　<form action="Transfer.php" method="POST">
　　　　<p>ToBankId: <input type="text" name="toBankId" /></p>
　　　　<p>Money: <input type="text" name="money" /></p>
　　　　<p><input type="submit" value="Transfer" /></p>
　　</form>
```
　　后台处理页面Transfer.php如下：
  
```php
　　<?php
　　　　session_start();
　　　　if (isset($_REQUEST['toBankId'] &&　isset($_REQUEST['money']))
　　　　{
　　　　    buy_stocks($_REQUEST['toBankId'],　$_REQUEST['money']);
　　　　}
　　?>
```
危险网站B，仍然只是包含那句HTML代码：

```
< img src=http://www.mybank.com/Transfer.php?toBankId=11&money=1000>
```
和示例1中的操作一样，你首先登录了银行网站A，然后访问危险网站B，**结果.....和示例1一样**，你再次没了1000块～T_T，这次事故的原因是：银行后台使用了$_REQUEST去获取请求的数据，**而$_REQUEST既可以获取GET请求的数据，也可以获取POST请求的数据，这就造成了在后台处理程序无法区分这到底是GET请求的数据还是POST请求的数据。在PHP中，可以使用$_GET和$_POST分别获取GET请求和POST请求的数据。在JAVA中，用于获取请求数据request一样存在不能区分GET请求数据和POST数据的问题。**

**　　示例3：**
经过前面2个惨痛的教训，银行决定把获取请求数据的方法也改了，改用$_POST，只获取POST请求的数据，后台处理页面Transfer.php代码如下：
  
```php
　　<?php
　　　　session_start();
　　　　if (isset($_POST['toBankId'] &&　isset($_POST['money']))
　　　　{
　　　　    buy_stocks($_POST['toBankId'],　$_POST['money']);
　　　　}
　　?>
```
然而，危险网站B与时俱进，它改了一下代码：

```
<html>
　　<head>
　　　　<script type="text/javascript">
　　　　　　function steal()
　　　　　　{
          　　　　 iframe = document.frames["steal"];
　　     　　      iframe.document.Submit("transfer");
　　　　　　}
　　　　</script>
　　</head>

　　<body onload="steal()">
　　　　<iframe name="steal" display="none">
　　　　　　<form method="POST" name="transfer"　action="http://www.myBank.com/Transfer.php">
　　　　　　　　<input type="hidden" name="toBankId" value="11">
　　　　　　　　<input type="hidden" name="money" value="1000">
　　　　　　</form>
　　　　</iframe>
　　</body>
</html>
```

如果用户仍是继续上面的操作，很不幸，结果将会是再次不见1000块......**因为这里危险网站B暗地里发送了POST请求到银行!**
理解上面的3种攻击模式，其实可以看出，**CSRF攻击是源于WEB的隐式身份验证机制！WEB的身份验证机制虽然可以保证一个请求是来自于某个用户的浏览器，` 但却无法保证该请求是用户批准发送的！ `**

**五.CSRF的防御**
CSRF的防御可以从服务端和客户端两方面着手，防御效果是从服务端着手效果比较好，现在一般的CSRF防御也都在服务端进行。
**1.服务端进行CSRF防御**
服务端的CSRF方式方法很多样，但总的思想都是一致的，就是**在客户端页面增加伪随机数**。
(1).Cookie Hashing(所有表单都包含同一个伪随机值)：
这可能是最简单的解决方案了，因为攻击者不能获得第三方的Cookie(理论上)，所以表单中的数据也就构造失败了:>
``` php
<?php
　　　　//构造加密的Cookie信息
　　　　$value = “DefenseSCRF”;
　　　　setcookie(”cookie”, $value, time()+3600);
　　?>

```
在表单里增加Hash值，以认证这确实是用户发送的请求。

```  php

　　<?php
　　　　$hash = md5($_COOKIE['cookie']);
　　?>
　　<form method=”POST” action=”transfer.php”>
　　　　<input type=”text” name=”toBankId”>
　　　　<input type=”text” name=”money”>
　　　　<input type=”hidden” name=”hash” value=”<?=$hash;?>”>
　　　　<input type=”submit” name=”submit” value=”Submit”>
　　</form>
```
然后在服务器端进行Hash值验证

``` php
<?php
      if(isset($_POST['check'])) {
           $hash = md5($_COOKIE['cookie']);
           if($_POST['check'] == $hash) {
                doJob();
           } else {
      //...
           }
      } else {
    //...
      }
    ?>
```
这个方法个人觉得已经可以杜绝99%的CSRF攻击了，那还有1%呢....由于用户的Cookie很容易由于网站的XSS漏洞而被盗取，这就另外的1%。一般的攻击者看到有需要算Hash值，基本都会放弃了，某些除外，所以如果需要100%的杜绝，这个不是最好的方法。
(2).验证码
这个方案的思路是：每次的用户提交都需要用户在表单中填写一个图片上的随机字符串，厄....这个方案可以完全解决CSRF，但个人觉得在易用性方面似乎不是太好，还有听闻是验证码图片的使用涉及了一个被称为MHTML的Bug，可能在某些版本的微软IE中受影响。
(3).One-Time Tokens(不同的表单包含一个不同的伪随机值)
在实现One-Time Tokens时，需要注意一点：就是“并行会话的兼容”。如果用户在一个站点上同时打开了两个不同的表单，CSRF保护措施不应该影响到他对任何表单的提交。**考虑一下如果每次表单被装入时站点生成一个伪随机值来覆盖以前的伪随机值将会发生什么情况：用户只能成功地提交他最后打开的表单，因为所有其他的表单都含有非法的伪随机值。必须小心操作以确保CSRF保护措施不会影响选项卡式的浏览或者利用多个浏览器窗口浏览一个站点。**
参考:http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html

# Flask-WTF基础
默认情况下Flask-WTF能保护所有表单免受跨站请求伪造(Cross-site request forgery，CSRF)攻击.恶意网站把请求发送到被攻击者已登录的其他网站时就会引发CSRF攻击。
为了实现CSRF保护，Flask-WTF需要程序设置一个密钥。Flask-WTF使用这个密钥生成加密令牌，再用令牌验证请求中表单数据的真伪。
**1、设置密钥:**

```
app=Flask(__name__)
app.config['SECRET_KEY']='hard to guess string'
```

app.config字典可用来存储框架、扩展和程序本身的配置变量。
SECRET_KEY配置变量是通用密钥，可在Flask和多个第三方扩展中使用。

 2、表单类 
使用Flask-WTF时，每个Web表单都由一个继承自Form的类表示。这个类定义表单中的一组字段，每个字段都用对象表示。字段对象可附属一个或多个验证函数。验证函数用来验证用户提交的输入值是否符合要求。
**注：最新的Flask-WTF建议使用`FlaskForm`来代替Form。**

hello.py定义表单类:

```
from flask_wtf import Form
from wforms import StringField,SubmitField
from wtforms.validators import Required

class NameForm(Form):
	name=StringField('What is your name?',validators=[Required()])
	submit=SubmitField('Submit')
```

WTForms支持的HTML标准字段

* StringField
* TextAreaField
* PasswordField
* HiddenField
* DateField、DateTimeField
* IntegerField、DecimalField、FloatField、BooleanField
* RadioField
* SelectField
* SelectMultipleField
* FileField
* SubmitField
* FormField
* FieldList

WTForms验证函数：

* Email
* EqualTo
* IPAddress
* Length
* NumberRange
* Optional
* Required
* Regexp
* URL
* AnyOf
* NoneOf

##  把表单渲染成HTML 

```
<form method="POST">
{{ form.hidden_tag() }}
{{ form.name.label }} {{ form.name() }}
{{ form.submit() }}
</form>
```
```
<form method="POST">
{{ form.hidden_tag() }}
{{ form.name.label }} {{ form.name(id='my-text-field') }}
{{ form.submit() }}
</form>
```
使用Flask-Bootstrap，上述表单可使用下面的方式渲染：

```
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```

# 在视图函数中处理表单
```
@main.route('/edit-profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = EditProfileForm()
    if form.validate_on_submit():
        current_user.name = form.name.data
        current_user.location = form.location.data
        current_user.about_me = form.about_me.data
        db.session.add(current_user)
        flash('Your profile has been updated.')
        return redirect(url_for('.user', username=current_user.username))
    form.name.data = current_user.name
    form.location.data = current_user.location
    form.about_me.data = current_user.about_me
    return render_template('edit_profile.html', form=form)
```

## 自定义验证函数
```
class CaptchaValid(object):
    def __init__(self,message=None):
        self.message=message

    def verify_captcha(self,code):
        captchaCode = session.get('captcha', None)
        if not captchaCode or not code:
            return False
        if captchaCode.lower()==code.lower():
            return True
        return False

    def __call__(self, form, field):
        if  not self.verify_captcha(field.data):
            if self.message is None:
                message = field.gettext('验证码错误')
            else:
                message = self.message
            #field.errors[:] = []
            raise ValidationError(message)


class LoginForm(SecLoginForm):
    code=StringField('验证码',validators=[CaptchaValid("验证码无效")])
```

# flash消息
```
<div class="row">
                <div style="margin-left: 10px;">
                    {% with messages = get_flashed_messages(with_categories=True) %}
                    {% if messages %}
                    {% for category, message in messages %}
                    <div class="alert  alert-{{ category }} alert-dismissible">
                        <button type="button" class="close" data-dismiss="alert">&times;</button>
                        {{ message }}
                    </div>
                    {% endfor %}
                    {% endif %}
                    {% endwith %}
                    {% if form and form.errors %}
                        {% for field_name in form.errors %}
                            <div class="alert alert-dismissible alert-danger">{{ form[field_name].label.text }}:{{ ','.join(form.errors[field_name]) }}</div>
                        {% endfor %}

                    {% endif %}
                    {% block content %}
                    {% endblock content %}
                </div>
                <div class="span3">
                    {% block rightbar %}
                    {% endblock rightbar %}
                </div>
            </div>
```

# render fields with errors
```
{% macro render_field_with_errors(field) %}

   <!--  {{ field.label }}  -->{{ field(**kwargs)|safe }}
   {% if field.errors %}
     <span class="text-danger">
        {{ ','.join(field.errors) }}
      </span>
   {% endif %}

{% endmacro %}
{% macro render_field(field) %}
  {{ field(**kwargs)|safe }}
{% endmacro %}
```