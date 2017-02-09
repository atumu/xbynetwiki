title: flask-login 

#  flask-login插件 
当前版本0.3.2
官网：https://flask-login.readthedocs.io/en/latest/
中文:http://www.pythondoc.com/flask-login/index.html

pip install flask-login

Flask-Login 为 Flask 提供了用户会话管理。它处理了日常的登入，登出并且长时间记住用户的会话。
它会:
  * 在会话中存储当前活跃的用户 ID，让你能够自由地登入和登出。
  * 让你限制登入(或者登出)用户可以访问的视图。
  * 处理让人棘手的 “记住我” 功能。
  * 帮助你保护用户会话免遭 cookie 被盗的牵连。
  * 可以与以后可能使用的 Flask-Principal 或其它认证扩展集成。

但是，它不会:
  * 限制你使用特定的数据库或其它存储方法。如何加载用户完全由你决定。
  * 限制你使用用户名和密码，OpenIDs，或者其它的认证方法。
  * 处理超越 “登入或者登出” 之外的权限。
  * 处理用户注册或者账号恢复。

##  快速入门 
Flask-Login 的应用最重要的一部分就是 LoginManager 类
你必须提供一个 user_loader 回调。这个回调用于从会话中存储的用户 ID 重新加载用户对象。它应该接受一个用户的 unicode ID 作为参数，并且返回相应的用户对象。比如:
```

@login_manager.user_loader
def load_user(userid):
    return User.get(userid)

```
如果 ID 无效的话，它应该返回 None (而不是抛出异常)。

**用户类需要实现这些属性和方法:**
  * is_authenticated()	如果用户已经登录,必须返回 True ,否则返回 False
  * is_active()	如果允许用户登录,必须返回 True ,否则返回 False 。如果要禁用账户,可以返回 False
  * is_anonymous()	对普通用户必须返回 False
  * get_id()	必须返回用户的唯一标识符,使用 Unicode 编码字符串
要简便地实现用户类，**你可以从 UserMixin 继承**，它提供了对所有这些方法的默认 实现。（虽然这不是必须的。）

Login 示例
一旦用户通过验证，你可以使用 **login_user** 函数让用户登录。例如:
```

from flask.ext.login import LoginManager
#创建实例
login_manager = LoginManager()
#LoginManager 对象的 session_protection 属性可以设为 None 、 'basic' 或 'strong'设为 'strong' 时,Flask-Login 会记录客户端 IP地址和浏览器的用户代理信息,如果发现异动就登出用户。
login_manager.session_protection = 'strong'
#设置登录页面的端点
login_manager.login_view = 'login'

login_manager.init_app(app)


from flask.ext.login import UserMixin
#... ...
class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)

#必须指定回调函数
@login_manager.user_loader
def load_user(user_id):
    return User.query.get(int(user_id))#should return None not raise an exception

@app.route('/login', methods=['GET', 'POST'])
def login():
    # Here we use a class of some kind to represent and validate our
    # client-side form data. For example, WTForms is a library that will
    # handle this for us, and we use a custom LoginForm to validate.
    form = LoginForm()
    if form.validate_on_submit():
        # Login and validate the user.
        # user should be an instance of your `User` class
        login_user(user)

        flask.flash('Logged in successfully.')

        next = flask.request.args.get('next')
        # next_is_valid should check if the user has valid
        # permission to access the `next` url
        if not next_is_valid(next):
            return flask.abort(400)

        return flask.redirect(next or flask.url_for('index'))
    return flask.render_template('login.html', form=form)

```
**警告: 你必须验证 next 参数的值。如果不验证的话，你的应用将会受到重定向的攻击。**
就这么简单。你可用使用 **current_user** 代理来访问登录的用户，在每一个模板中都可以使用 current_user:
```

{% if current_user.is_authenticated() %}
  Hi ![](/data/dokuwiki current_user.name )!
{% endif %}

```
需要用户**登入** 的视图可以用 login_required 装饰器来装饰:
```

@app.route("/settings")
@login_required
def settings():
    pass

```
当用户要**登出**时:

```

@app.route("/logout")
@login_required
def logout():
    logout_user()
    return redirect(somewhere)

```
##  定制登入过程 
默认情况下，当未登录的用户尝试访问一个 login_required 装饰的视图，Flask-Login 会闪现一条消息并且重定向到登录视图。(如果未设置登录视图，它将会以 401 错误退出。)
登录视图的名称可以设置成 LoginManager.login_view。例如:
login_manager.login_view = "users.login"
默认的闪现消息是 Please log in to access this page.。要自定义该信息，请设置 LoginManager.login_message:
login_manager.login_message =  "请登录"
要自定义消息分类的话，请设置 LoginManager.login_message_category:
login_manager.login_message_category = "info"
` 当重定向到登入视图，它的请求字符串中会有一个 next 变量，其值为用户之前访问的页面。 `
如果你想要进一步自定义登入过程，请使用 LoginManager.unauthorized_handler 装饰函数:
```

@login_manager.unauthorized_handler
def unauthorized():
    # do stuff
    return a_response

```
  
##  Post/ 重定向 /Get 模式 
```

#“Post/ 重定向 /Get 模式”,提交登录密令的 POST 请求最后也做了重定向
#重定向的地址有两种：
#1、用户访问一个为授权的页面，会显示登录表单，Flask-login会把原地址保存在查询字符串的next参数中，通过flask.request.args.get('next')访问。
#2、回到主页面
@app.route('/login', methods=['GET', 'POST'])
def login():
    # Here we use a class of some kind to represent and validate our
    # client-side form data. For example, WTForms is a library that will
    # handle this for us, and we use a custom LoginForm to validate.
    form = LoginForm()
    if form.validate_on_submit():
        # Login and validate the user.
        # user should be an instance of your `User` class
        login_user(user)#flask-login中的函数，在用户会话中把用户标记为已登录。如果想要实现"Remember me"功能，只需要传递remembeer=True即可。此时：一个cokkie即可保存到用户的电脑上，并且，如果userID没有在会话时，Flask-Login将自动从cookie中恢复。并且这个cookie是防修改的，如果用户尝试修改，这个cookie会被服务器丢弃。

        flask.flash('Logged in successfully.')

        next = flask.request.args.get('next')
        # next_is_valid should check if the user has valid
        # permission to access the `next` url
        if not next_is_valid(next):#检查用户是否有权限访问该next页面
            return flask.abort(400)

        return flask.redirect(next or flask.url_for('index'))
    return flask.render_template('login.html', form=form)

```
##  使用 Request Loader 定制登录 
有时你想要不使用 cookies 情况下登录用户，比如使用 HTTP 头或者一个作为查询参数的 api 密钥。
这种情况下，你应该使用 **request_loader** 回调。这个回调和 user_loader 回调作用一样，但是 user_loader 回调只接受 Flask 请求而不是一个 user_id。
例如，为了同时支持一个 url 参数和使用 Authorization 头的基本用户认证的登录:
```

@login_manager.request_loader
def load_user_from_request(request):

    # first, try to login using the api_key url arg
    api_key = request.args.get('api_key')
    if api_key:
        user = User.query.filter_by(api_key=api_key).first()
        if user:
            return user

    # next, try to login using Basic Auth
    api_key = request.headers.get('Authorization')
    if api_key:
        api_key = api_key.replace('Basic ', '', 1)
        try:
            api_key = base64.b64decode(api_key)
        except TypeError:
            pass
        user = User.query.filter_by(api_key=api_key).first()
        if user:
            return user

    # finally, return None if both methods did not login the user
    return None

```
##  匿名用户 
默认情况下，当一个用户没有真正地登录，current_user 被设置成一个 **AnonymousUserMixin** 对象。它由如下的属性和方法:
  * is_active 和 is_authenticated 的值为 False
  * is_anonymous 的值为 True
  * get_id() 返回 None
如果需要为匿名用户定制一些需求(比如，需要一个权限域)，你可以向 LoginManager 提供一个创建匿名用户的回调（类或工厂函数）:
login_manager.anonymous_user = MyAnonymousUser

##  记住我 
“记住我”的功能很难实现。但是，Flask-Login 几乎透明地实现它 - **只要把 remember=True 传递给 login_user。**
一个 cookie 将会存储在用户计算机中，如果用户会话中没有用户 ID 的话，Flask-Login 会自动地从 cookie 中恢复用户 ID。cookie 是防纂改的。

##  可选令牌 
使用用户 ID 作为记住的令牌值不一定是安全的。更安全的方法是使用用户名和密码联合的 hash 值，或类似的东西。
要添加一个额外的令牌，向你的用户对象添加一个方法：
get_auth_token()
返回用户的认证令牌（返回为 unicode ）。这个认证令牌应能唯一识别用户，且不易通过用户的公开信息，如 UID 和名称来猜测出——同样也不应暴露这些信息。
相应地，**你应该在 LoginManager 上设置一个 token_loader 函数， 它接受令牌（存储在 cookie 中）作为参数并返回合适的 User 对象。**

**make_secure_token 函数**用于便利创建认证令牌。它会连接所有的参数，然后用应用的密钥来 HMAC 它确保最大的密码学安全。（如果你永久地在数据库中存储用户令牌，那么你会希望向令牌中添加随机数据来阻碍猜测。）

如果你的应用使用密码来验证用户，在认证令牌中包含密码（或你应使用的加盐值的密码 hash ）能确保若用户更改密码，他们的旧认证令牌会失效。

##  ”新鲜的“登录(Fresh Logins) 
当用户登入，他们的会话被标记成“新鲜的”，就是说在这个会话只中用户实际上登录过。
当会话销毁用户使用“记住我”的 cookie 重新登入，会话被标记成“非新鲜的”。
login_required 并不在意它们之间的不同，这适用于大部分页面。然而，更改某人 的个人信息这样的敏感操作应需要一个“新鲜的”的登入。（像修改密码这样的操作总是需要 密码，无论是否重登入。）

**fresh_login_required**，除了验证用户登录，也将确保他们的登录是“新鲜的”。如果不是“新鲜的”，它会把用户送到可以重输入验证条件的页面。你可以定制 fresh_login_required 就像定制 login_required 那样，通过设置 LoginManager.refresh_view，needs_refresh_message，和needs_refresh_message_category:

```

login_manager.refresh_view = "accounts.reauthenticate"
login_manager.needs_refresh_message = (
    u"To protect your account, please reauthenticate to access this page."
)
login_manager.needs_refresh_message_category = "info"

```
或者提供自己的回调来处理“非新鲜的”刷新:
```

@login_manager.needs_refresh_handler
def refresh():
    # do stuff
    return a_response

```
调用 **confirm_login** 函数可以重新标记会话为”新鲜“。

##  记住我 Cookie 设置 
cookie 的细节可以在应用设置中定义。
REMEMBER_COOKIE_NAME	存储“记住我”信息的 cookie 名。 默认值： remember_token
REMEMBER_COOKIE_DURATION	cookie 过期时间，为一个 datetime.timedelta 对象。** 默认值： 365 天** (1 非闰阳历年)
REMEMBER_COOKIE_DOMAIN	如果“记住我” cookie 应跨域，在此处设置域名值 （即 .example.com 会允许 example 下所有子域 名）。 默认值： None
REMEMBER_COOKIE_PATH	限制”记住我“ cookie 存储到某一路径下。 默认值： /
REMEMBER_COOKIE_SECURE	限制 “Remember Me” cookie 在某些安全通道下有用 （典型地 HTTPS）。默认值： None
REMEMBER_COOKIE_HTTPONLY	保护 “Remember Me” cookie 不能通过客户端脚本访问。 默认值： False
##  会话保护 
你可以在 LoginManager 上和应用配置中配置会话保护。如果它被启用，它可以在 basic 或 strong 两种模式中运行。要在 LoginManager 上设置它，设置 session_protection 属性为 "basic" 或 "strong":

login_manager.session_protection = "strong" 
或者，禁用它:

login_manager.session_protection = None
默认，它被激活为 "basic" 模式。它可以在应用配置中设定 SESSION_PROTECTION 为 None 、 "basic" 或 "strong" 来禁用。
` 设为 'strong' 时,Flask-Login 会记录客户端 IP地址和浏览器的用户代理信息,如果发现异动就登出用户. `
##  信号 
flask.ext.login.user_logged_in
当一个用户登入的时候发出。除应用（信号的发送者）之外，它还传递正登入的用户 user 。
flask.ext.login.user_logged_out
当一个用户登出的时候发出。除应用（信号的发送者）之外，它还传递正登出的用户 user 。

flask.ext.login.user_login_confirmed
当用户的登入被证实，把它标记为活跃的。（它不用于常规登入的调用。） 它不接受应用以外的任何其它参数。

flask.ext.login.user_unauthorized
当 LoginManager 上的 unauthorized 方法被调用时发出。它不接受应用以外的任何其它参数。

flask.ext.login.user_needs_refresh
当 LoginManager 上的 needs_refresh 方法被调用时发出。它不接受应用以外的任何其它参数。

flask.ext.login.session_protected
当会话保护起作用时，且会话被标记为非活跃或删除时发出。它不接受应用以外的任何其它参数。

其余参考：http://blog.csdn.net/sun_dragon/article/details/51735944