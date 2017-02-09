title: pythonflask-security 

#  Flask-Security完整的权限插件 
官网：https://pythonhosted.org/Flask-Security/
当前版本：1.7.5

特点：
  * 基于session的认证机制
  * 角色管理
  * 密码加密
  * Basic HTTP authentication
  * Token based authentication
  * account activation，Token based  (optional)
  * 密码重置 ，Token based (optional)
  * 用户注册 (optional)
  * Login tracking (optional)
  * JSON/Ajax Support

集成的插件：
  * Flask-Login
  * Flask-Mail
  * Flask-Principal
  * Flask-Script
  * Flask-WTF
  * itsdangerous
  * passlib
  * Flask-SQLAlchemy
  * Flask-MongoEngine
  * Flask-Peewee

安装：pip install flask-security

##  配置 
https://pythonhosted.org/Flask-Security/configuration.html

**核心配置**
SECURITY_BLUEPRINT_NAME	Specifies the name for the Flask-Security blueprint. Defaults to security.
SECURITY_URL_PREFIX	Specifies the URL prefix for the Flask-Security blueprint. Defaults to None.
SECURITY_FLASH_MESSAGES	Specifies whether or not to flash messages during security procedures. Defaults to True.
**SECURITY_PASSWORD_HASH**	密码加密算法，默认为不加密，支持bcrypt, sha512_crypt, or pbkdf2_sha512. 
**SECURITY_PASSWORD_SALT**	指定 HMAC salt. 默认为None.

**SECURITY_EMAIL_SENDER	**

SECURITY_TOKEN_AUTHENTICATION_KEY	当使用token authentication指定查询参数.默认为auth_token.
SECURITY_TOKEN_AUTHENTICATION_HEADER	当使用token authentication指定HTTP header，默认为 Authentication-Token.
**SECURITY_TOKEN_MAX_AGE**	指定authentication token的失效时间，以秒为单位。默认为不失效。
SECURITY_DEFAULT_HTTP_AUTH_REALM	Specifies the default authentication realm when using basic HTTP auth. Defaults to Login Required

**注册、登录、密码相关URL和视图**

**SECURITY_LOGIN_URL**	Specifies the login URL. Defaults to /login.
**SECURITY_LOGOUT_URL**	Specifies the logout URL. Defaults to /logout.
**SECURITY_REGISTER_URL**	Specifies the register URL. Defaults to /register.
**SECURITY_RESET_URL**	Specifies the password reset URL. Defaults to /reset.
**SECURITY_CHANGE_URL**	Specifies the password change URL. Defaults to /change.
**SECURITY_CONFIRM_URL**	Specifies the email confirmation URL. Defaults to /confirm.
**SECURITY_POST_LOGIN_VIEW**	指定用户登录成功后重定向的页面。可以设置为a URL or an endpoint name. 默认为 /.
**SECURITY_POST_LOGOUT_VIEW**	指定用户退出登录后重定向的页面。可以设置为a URL or an endpoint name. 默认为 /.

SECURITY_CONFIRM_ERROR_VIEW	Specifies the view to redirect to if a confirmation error occurs. This value can be set to a URL or an endpoint name. If this value is None, the user is presented the default view to resend a confirmation link. Defaults to None.
**SECURITY_POST_REGISTER_VIEW**	指定用户注册成功后重定向的页面。默认为None，如果为 None,redirected to the value of SECURITY_POST_LOGIN_VIEW. 
**SECURITY_POST_CONFIRM_VIEW**	指定用户确认电子邮件后重定向的页面. 
**SECURITY_POST_RESET_VIEW**	指定用户成功重置密码后重定向的页面。
**SECURITY_POST_CHANGE_VIEW**	指定用户成功更改密码后重定向的页面。
**SECURITY_UNAUTHORIZED_VIEW**	指定未授权（403）重定向页面。

**模板路径：**
**SECURITY_FORGOT_PASSWORD_TEMPLATE**	指定忘记密码的模板路径。Defaults to security/forgot_password.html.
**SECURITY_LOGIN_USER_TEMPLATE**	指定登录页面模板路径。Defaults to security/login_user.html.
**SECURITY_REGISTER_USER_TEMPLATE**		指定注册页面模板路径。	 Defaults to security/register_user.html.
**SECURITY_RESET_PASSWORD_TEMPLATE**	指定密码重置页面模板路径。 Defaults to security/reset_password.html.
**SECURITY_CHANGE_PASSWORD_TEMPLATE**	指定修改密码页面模板路径。 Defaults to security/change_password.html.
**SECURITY_SEND_CONFIRMATION_TEMPLATE**	指定发送确认邮件页面模板路径。 Defaults to security/send_confirmation.html.
**SECURITY_SEND_LOGIN_TEMPLATE**	Specifies the path to the template for the send login instructions page for passwordless logins. Defaults to security/send_login.html.

**特殊标志**
**SECURITY_CONFIRMABLE**	用户注册时是否需要确认邮件，默认为False。如果为 True, 使用 SECURITY_CONFIRM_URL来处理. 
**SECURITY_REGISTERABLE**	是否允许用户注册。默认为False。如果为True，那么交由 SECURITY_REGISTER_URL处理。
**SECURITY_RECOVERABLE**	是否允许密码重置，默认为False。如果为True，那么交由SECURITY_RESET_URL处理。
**SECURITY_TRACKABLE**	默认为False，是否需要用户登录统计的相关信息。如果设为True，用户模型需要添加字段，见后面。如果使用了代理服务器，请确保配置正确，具体见 <http://flask.pocoo.org/docs/0.10/deploying/wsgi-standalone/#proxy-setups> 

SECURITY_PASSWORDLESS	Specifies if Flask-Security should enable the passwordless login feature. If set to True, users are not required to enter a password to login but are sent an email with a login link. This feature is experimental and should be used with caution. Defaults to False.

**SECURITY_CHANGEABLE**	是否允许密码修改，默认为False。如果为True，那么交由SECURITY_CHANGE_URL处理。

**电子邮件：**
**SECURITY_EMAIL_SUBJECT_REGISTER**	注册确认邮件的主题。
SECURITY_EMAIL_SUBJECT_PASSWORDLESS	Sets the subject for the passwordless feature. Defaults to Login instructions
**SECURITY_EMAIL_SUBJECT_PASSWORD_NOTICE**	Sets subject for the password notice. Defaults to Your password has been reset
**SECURITY_EMAIL_SUBJECT_PASSWORD_RESET**	Sets the subject for the password reset email. Defaults to Password reset instructions
**SECURITY_EMAIL_SUBJECT_PASSWORD_CHANGE_NOTICE**	Sets the subject for the password change notice. Defaults to Your password has been changed
**SECURITY_EMAIL_SUBJECT_CONFIRM**	Sets the subject for the email confirmation message. Defaults to Please confirm your email

**杂项**
SECURITY_SEND_REGISTER_EMAIL	Specifies whether registration email is sent. Defaults to True.
SECURITY_SEND_PASSWORD_CHANGE_EMAIL	Specifies whether password change email is sent. Defaults to True.
SECURITY_SEND_PASSWORD_RESET_NOTICE_EMAIL	Specifies whether password reset notice email is sent. Defaults to True.
**SECURITY_CONFIRM_EMAIL_WITHIN**	确认邮件链接失效时间，默认5 days.
**SECURITY_RESET_PASSWORD_WITHIN**	密码重置邮件链接失效时间，默认5 days.
SECURITY_LOGIN_WITHIN	Specifies the amount of time a user has before a login link expires. This is only used when the passwordless login feature is enabled. Always pluralized the time unit for this value. Defaults to 1 days.
SECURITY_LOGIN_WITHOUT_CONFIRMATION	Specifies if a user may login before confirming their email when the value of SECURITY_CONFIRMABLE is set to True. Defaults to False.
SECURITY_CONFIRM_SALT	Specifies the salt value when generating confirmation links/tokens. Defaults to confirm-salt.
SECURITY_RESET_SALT	Specifies the salt value when generating password reset links/tokens. Defaults to reset-salt.
SECURITY_LOGIN_SALT	Specifies the salt value when generating login links/tokens. Defaults to login-salt.
**SECURITY_REMEMBER_SALT**	Specifies the salt value when generating remember tokens. Remember tokens are used instead of user ID’s as it is more secure. Defaults to remember-salt.
**SECURITY_DEFAULT_REMEMBER_ME**	默认为False，是否允许记住我功能。

**消息配置：**
略。


##  快速入门-Basic SQLAlchemy Application 
```

$ pip install flask-security flask-sqlalchemy

```
```

from flask import Flask, render_template
from flask.ext.sqlalchemy import SQLAlchemy
from flask.ext.security import Security, SQLAlchemyUserDatastore, \
    UserMixin, RoleMixin, login_required

# Create app
app = Flask(__name__)
app.config['DEBUG'] = True
app.config['SECRET_KEY'] = 'super-secret'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite://'

# Create database connection object
db = SQLAlchemy(app)

# Define models
roles_users = db.Table('roles_users',
        db.Column('user_id', db.Integer(), db.ForeignKey('user.id')),
        db.Column('role_id', db.Integer(), db.ForeignKey('role.id')))

class Role(db.Model, RoleMixin):
    id = db.Column(db.Integer(), primary_key=True)
    name = db.Column(db.String(80), unique=True)
    description = db.Column(db.String(255))

class User(db.Model, UserMixin):
    id = db.Column(db.Integer(), primary_key=True)
    email = db.Column(db.String(255), unique=True)
    password = db.Column(db.String(255))
    active = db.Column(db.Boolean())
    confirmed_at = db.Column(db.DateTime())
    roles = db.relationship('Role', secondary=roles_users,
                            backref=db.backref('users', lazy='dynamic'))

# Setup Flask-Security
user_datastore = SQLAlchemyUserDatastore(db, User, Role)
security = Security(app, user_datastore)

# Create a user to test with
@app.before_first_request
def create_user():
    db.create_all()
    user_datastore.create_user(email='matt@nobien.net', password='password')
    db.session.commit()

# Views
@app.route('/')
@login_required
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run()

```
##  受保护视图 
flask_security.decorators.**login_required**(func)
```

@app.route('/post')
@login_required
def post():
    pass
完成如下工作：
if not current_user.is_authenticated:
    return current_app.login_manager.unauthorized()

```
flask_security.decorators.**roles_required**(*roles)
```

@app.route('/dashboard')
@roles_required('admin', 'editor')
def dashboard():
    return 'Dashboard'

```
flask_security.decorators.**roles_accepted**(*roles)
```

@app.route('/create_post')
@roles_accepted('editor', 'author')
def create_post():
    return 'Create Post'

```
  
class flask_security.core.**UserMixin**
  * has_role(role)
  * is_active
  * get_auth_token()

class flask_security.core.**RoleMixin**
class flask_security.core.**AnonymousUser**
##  Datastores与用户/角色管理 
class flask_security.datastore.**UserDatastore**(user_model, role_model)
  * activate_user(user)
  * deactivate_user(user)
  * add_role_to_user(user, role)
  * remove_role_from_user(user, role)

  * create_role(* *kwargs)
  * create_user(* *kwargs)
  * delete_user(user)

  * find_or_create_role(name, * *kwargs)
  * find_role(*args, * *kwargs)
  * find_user(*args, * *kwargs)
  * get_user(id_or_email)

  * toggle_active(user)

class flask_security.datastore.**SQLAlchemyUserDatastore**(db, user_model, role_model),实现了UserDatastore当中的所有方法
class flask_security.datastore.**MongoEngineUserDatastore**(db, user_model, role_model)

##  工具类： 
flask_security.utils.**url_for_security**(endpoint, * *values) ，Return a URL for the security blueprint
flask_security.utils.login_user(user, remember=None) 
flask_security.utils.logout_user()
flask_security.utils.get_hmac(password)
flask_security.utils.verify_password(password, password_hash)
flask_security.utils.verify_and_update_password(password, user)
flask_security.utils.encrypt_password(password)

flask_security.utils.**send_mail**(subject, recipient, template, * *context)
flask_security.utils.get_token_status(token, serializer, max_age=None, return_data=False)

##  信号 
**user_registered**
In addition to the app (which is the sender), it is passed user and confirm_token arguments.

**user_confirmed**
Sent when a user is confirmed. In addition to the app (which is the sender), it is passed a user argument.

**confirm_instructions_sent**
Sent when a user requests confirmation instructions. In addition to the app (which is the sender), it is passed a user argument.

**login_instructions_sent**
Sent when passwordless login is used and user logs in. In addition to the app (which is the sender), it is passed user and login_token arguments.

**password_reset**
Sent when a user completes a password reset. In addition to the app (which is the sender), it is passed a user argument.

**password_changed**
Sent when a user completes a password change. In addition to the app (which is the sender), it is passed a user argument.

**reset_password_instructions_sen**t
Sent when a user requests a password reset. In addition to the app (which is the sender), it is passed user and token arguments.
##  邮件配置 
集成Flask-mail
```

# At top of file
from flask_mail import Mail

# After 'Create app'
app.config['MAIL_SERVER'] = 'smtp.example.com'
app.config['MAIL_PORT'] = 465
app.config['MAIL_USE_SSL'] = True
app.config['MAIL_USERNAME'] = 'username'
app.config['MAIL_PASSWORD'] = 'password'
mail = Mail(app)

```
##  用户/角色模型 
Flask-Security允许你通过SQLAlchemy, MongoEngine or Peewee定义User and Role model。
根据你需要的功能不同，User、Role模型需要约定性定义字段。
最小的要求：
User
  * id
  * email
  * password
  * active

Role
  * id
  * name
  * description

如果你配置了SECURITY_CONFIRMABLE=True，那么你的User模型需要添加以下字段：
  * **confirmed_at**
如果你配置了SECURITY_TRACKABLE=True，那么你的User模型需要添加以下字段：
  * last_login_at
  * current_login_at
  * last_login_ip
  * current_login_ip
  * login_count

##  自定义视图 
  * security/forgot_password.html
  * security/login_user.html
  * security/register_user.html
  * security/reset_password.html
  * security/change_password.html
  * security/send_confirmation.html
  * security/send_login.html

每个模板包含以下两个额外的上下文变量：
  * <template_name>_form: A form object for the view
  * security: The Flask-Security extension object
如果想添加更多的模板上下文变量：你可以通过context processor给所有，或者特定view。
```

security = Security(app, user_datastore)

# This processor is added to all templates
@security.context_processor
def security_context_processor():
    return dict(hello="world")

# This processor is added to only the register view
@security.register_context_processor
def security_register_processor():
    return dict(something="else")

```
可用的装饰器有：
  * context_processor: All views
  * forgot_password_context_processor: Forgot password view
  * login_context_processor: Login view
  * register_context_processor: Register view
  * reset_password_context_processor: Reset password view
  * change_password_context_processor: Reset password view
  * send_confirmation_context_processor: Send confirmation view
  * send_login_context_processor: Send login view

###  自定义表单 
```

from flask_security.forms import RegisterForm

class ExtendedRegisterForm(RegisterForm):
    first_name = StringField('First Name', [Required()])
    last_name = StringField('Last Name', [Required()])

security = Security(app, user_datastore,
         register_form=ExtendedRegisterForm)

```
The following is a list of all the available form overrides:
  * login_form: Login form
  * confirm_register_form: Confirmable register form
  * register_form: Register form
  * forgot_password_form: Forgot password form
  * reset_password_form: Reset password form
  * change_password_form: Reset password form
  * send_confirmation_form: Send confirmation form
  * passwordless_login_form: Passwordless login form

###  电子邮件模板自定义 
  * security/email/confirmation_instructions.html
  * security/email/confirmation_instructions.txt
  * security/email/login_instructions.html
  * security/email/login_instructions.txt
  * security/email/reset_instructions.html
  * security/email/reset_instructions.txt
  * security/email/reset_notice.html
  * security/email/change_notice.txt
  * security/email/change_notice.html
  * security/email/reset_notice.txt
  * security/email/welcome.html
  * security/email/welcome.txt

给模板添加额外变量：@security.mail_context_processor
```

security = Security(app, user_datastore)

# This processor is added to all emails
@security.mail_context_processor
def security_mail_processor():
    return dict(hello="world")

```



##  通过Celery来发送邮件 
@security.send_mail_task
```

# Setup the task
@celery.task
def send_security_email(msg):
    # Use the Flask-Mail extension instance to send the incoming ``msg`` parameter
    # which is an instance of `flask_mail.Message`
    mail.send(msg)

@security.send_mail_task
def delay_security_email(msg):
    send_security_email.delay(msg)

```
    
    
##  使用datastore.create_user创建用户，密码不会被hash 
Conceptually there is a difference between the **Datastore** object and the **registerable** module. The **Datastore** object is a very simple layer on top of the ORM. It is not concerned with manipulating the data before it is inserted into the database. I prefer this separation of concerns. If you want to create a user with the **datastore.create_user** function, you can import **flask_security.utils.encrypt_password** to do what you need before creating the new record in the database.
https://github.com/mattupstate/flask-security/issues/136