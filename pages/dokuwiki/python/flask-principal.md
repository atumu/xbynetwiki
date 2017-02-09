title: flask-principal 

#  flask-Principal权限管理 
官网：https://pythonhosted.org/Flask-Principal/
当前版本：0.4.0

主要接口：Identity, Needs, Permission, and the IdentityContext.
  * Identity：代表用户，这个用户包含对应的权限信息。
  * Need ：细粒度的访问控制0。代表一个访问资源的权限。如has the admin role”, “can edit blog posts.Need可以是任何tuple，或者一个对象
Need is a permission to access a resource, an Identity should provide a set of Needs that it has access to. 
  * Permission：代表权限的集合。
  * IdentityContext ：标识与权限的上下文，可以通过contextmanager使用，或者装饰器使用。

![](/data/dokuwiki/python/pasted/20161020-213917.png)

##  资源访问控制 
```

from flask import Flask, Response
from flask.ext.principal import Principal, Permission, RoleNeed

app = Flask(__name__)

# load the extension
principals = Principal(app)

# Create a permission with a single Need, in this case a RoleNeed.
admin_permission = Permission(RoleNeed('admin'))

# protect a view with a principal for that need
@app.route('/admin')
@admin_permission.require()
def do_admin_index():
    return Response('Only if you are an admin')

# this time protect with a context manager
@app.route('/articles')
def do_articles():
    with admin_permission.require():
        return Response('Only if you are admin')

```
##  认证机制 
当一个请求被认证之后需要通过identity-changed信号来进行处理。
flask-Principal结合Flask-Login 进行授权的示例：
```

from flask import Flask, current_app, request, session
from flask.ext.login import LoginManager, login_user, logout_user, \
     login_required, current_user
from flask.ext.wtf import Form, TextField, PasswordField, Required, Email
from flask.ext.principal import Principal, Identity, AnonymousIdentity, \
     identity_changed

app = Flask(__name__)

Principal(app)

login_manager = LoginManager(app)

@login_manager.user_loader
def load_user(userid):
    # Return an instance of the User model
    return datastore.find_user(id=userid)

class LoginForm(Form):
    email = TextField()
    password = PasswordField()

@app.route('/login', methods=['GET', 'POST'])
def login():
    # A hypothetical login form that uses Flask-WTF
    form = LoginForm()

    # Validate form input
    if form.validate_on_submit():
        # Retrieve the user from the hypothetical datastore
        user = datastore.find_user(email=form.email.data)

        # Compare passwords (use password hashing production)
        if form.password.data == user.password:
            # Keep the user info in the session using Flask-Login
            login_user(user)

            # Tell Flask-Principal the identity changed
            identity_changed.send(current_app._get_current_object(),
                                  identity=Identity(user.id))

            return redirect(request.args.get('next') or '/')

    return render_template('login.html', form=form)

@app.route('/logout')
@login_required
def logout():
    # Remove the user information from the session
    logout_user()

    # Remove session keys set by Flask-Principal
    for key in ('identity.name', 'identity.auth_type'):
        session.pop(key, None)

    # Tell Flask-Principal the user is anonymous
    identity_changed.send(current_app._get_current_object(),
                          identity=AnonymousIdentity())

    return redirect(request.args.get('next') or '/')

```
##  提供用户权限、角色等信息 
identity-loaded信号
```

from flask.ext.login import current_user
from flask.ext.principal import identity_loaded, RoleNeed, UserNeed

@identity_loaded.connect_via(app)
def on_identity_loaded(sender, identity):
    # Set the identity user object
    identity.user = current_user

    # Add the UserNeed to the identity
    if hasattr(current_user, 'id'):
        identity.provides.add(UserNeed(current_user.id))

    # Assuming the User model has a list of roles, update the
    # identity with the roles that the user provides
    if hasattr(current_user, 'roles'):
        for role in current_user.roles:
            identity.provides.add(RoleNeed(role.name))

```
            
##  细粒度的资源控制 
```

from collections import namedtuple
from functools import partial

from flask.ext.login import current_user
from flask.ext.principal import identity_loaded, Permission, RoleNeed, \
     UserNeed

BlogPostNeed = namedtuple('blog_post', ['method', 'value'])
EditBlogPostNeed = partial(BlogPostNeed, 'edit')

class EditBlogPostPermission(Permission):
    def __init__(self, post_id):
        need = EditBlogPostNeed(unicode(post_id))
        super(EditBlogPostPermission, self).__init__(need)

@identity_loaded.connect_via(app)
def on_identity_loaded(sender, identity):
    # Set the identity user object
    identity.user = current_user

    # Add the UserNeed to the identity
    if hasattr(current_user, 'id'):
        identity.provides.add(UserNeed(current_user.id))

    # Assuming the User model has a list of roles, update the
    # identity with the roles that the user provides
    if hasattr(current_user, 'roles'):
        for role in current_user.roles:
            identity.provides.add(RoleNeed(role.name))

    # Assuming the User model has a list of posts the user
    # has authored, add the needs to the identity
    if hasattr(current_user, 'posts'):
        for post in current_user.posts:
            identity.provides.add(EditBlogPostNeed(unicode(post.id)))
          
@app.route('/posts/<post_id>', methods=['PUT', 'PATCH'])
def edit_post(post_id):
    permission = EditBlogPostPermission(post_id)

    if permission.can():
        # Save the edits ...
        return render_template('edit_post.html')

    abort(403)  # HTTP Forbidden

```
