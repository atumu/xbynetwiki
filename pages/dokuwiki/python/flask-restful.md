title: flask-restful 

#  flask-restful插件 
当前版本0.3.5
官网:http://flask-restful.readthedocs.io/en/0.3.5/
中文:http://www.pythondoc.com/Flask-RESTful/index.html
http://www.pythondoc.com/flask-restful/index.html

pip install flask-restful

##  快速入门 
```

from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource): #视图类继承Resource，覆盖get, put, post,delete等方法
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)

```
###  支持的视图方法返回值类型 

```

class Todo1(Resource):
    def get(self):
        # Default to 200 OK
        return {'task': 'Hello world'}

class Todo2(Resource):
    def get(self):
        # Set the response code to 201
        return {'task': 'Hello world'}, 201

class Todo3(Resource):
    def get(self):
        # Set the response code to 201 and return custom headers
        return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}

```
###  URL 
```

#支持多个URL
api.add_resource(HelloWorld,
    '/',
    '/hello')
#支持路径化参数
api.add_resource(TodoSimple, '/todo/<string:todo_id>')
api.add_resource(Todo,
    '/todo/<int:todo_id>', endpoint='todo_ep')

```
add_resource 函数**使用指定的 endpoint 注册路由到框架上**。如果没有指定 endpoint，Flask-RESTful 会根据类名生成一个，
但是有时候有些函数比如** url_for 需要 endpoint**，因此我会明确给 endpoint 赋值。
##  请求参数解析 
 Flask-RESTful 自带了请求数据验证机制
```

from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate to charge for this resource')
args = parser.parse_args()

```
验证失败时会出现：
$ curl -d 'rate=foo' http://127.0.0.1:5000/todos
{'status': 400, 'message': 'foo cannot be converted to int'}



###  完整示例 
```

from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)
api = Api(app)

TODOS = {
    'todo1': {'task': 'build an API'},
    'todo2': {'task': '?????'},
    'todo3': {'task': 'profit!'},
}


def abort_if_todo_doesnt_exist(todo_id):
    if todo_id not in TODOS:
        abort(404, message="Todo {} doesn't exist".format(todo_id))

parser = reqparse.RequestParser()
parser.add_argument('task')


# Todo
# shows a single todo item and lets you delete a todo item
class Todo(Resource):
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]

    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204

    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        TODOS[todo_id] = task
        return task, 201


# TodoList
# shows a list of all todos, and lets you POST to add new tasks
class TodoList(Resource):
    def get(self):
        return TODOS

    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

##
## Actually setup the Api resource routing here
##
api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')


if __name__ == '__main__':
    app.run(debug=True)

```
    
##  请求解析与验证 
在2.0版本被标记为废弃。但是好像目前只能用它来进行input 验证。

```

from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate cannot be converted')
parser.add_argument('name')
args = parser.parse_args()

```
必须参数：
```

parser.add_argument('name', required=True,help="Name cannot be blank!")

```
多值参数：action='append'
curl http://api.example.com -d "name=bob" -d "name=sue" -d "name=joe"
```

parser.add_argument('name', action='append')

```
And your args will look like this
args = parser.parse_args()
args['name']    # ['bob', 'sue', 'joe']

以不同的名字来引用：
```

parser.add_argument('name', dest='public_name')
args = parser.parse_args()
args['public_name']

```

参数来源：
By default, the RequestParser tries to parse values from **flask.Request.values**, and** flask.Request.json.**
添加参数来源：
```

# Look only in the POST body
parser.add_argument('name', type=int, location='form')

# Look only in the querystring
parser.add_argument('PageSize', type=int, location='args')

# From the request headers
parser.add_argument('User-Agent', location='headers')

# From http cookies
parser.add_argument('session_id', location='cookies')

# From file uploads
parser.add_argument('picture', type=werkzeug.datastructures.FileStorage, location='files')

```
**Only use type=list when location='json'**
同时指定多个来源：
parser.add_argument('text', location=['headers', 'values'])

错误处理与错误消息：
```

from flask import Flask
app = Flask(__name__)
app.config['BUNDLE_ERRORS'] = True

```
```

from flask_restful import reqparse

parser = reqparse.RequestParser() parser.add_argument(
‘foo’, choices=(‘one’, ‘two’), help=’Bad choice: {error_msg}’
)

# If a request comes in with a value of “three” for foo:
{
“message”: {
“foo”: “Bad choice: three is not a valid choice”,
}

}

```

示例：
```

from flask_restful import fields, marshal_with, reqparse, Resource

def email(email_str):
    """Return email_str if valid, raise an exception in other case."""
    if valid_email(email_str):
        return email_str
    else:
        raise ValueError('{} is not a valid email'.format(email_str))


user_fields = {
    'id': fields.Integer,
    'username': fields.String,
    'email': fields.String,
    'user_priority': fields.Integer,
    'custom_greeting': fields.FormattedString('Hey there {username}!'),
    'date_created': fields.DateTime,
    'date_updated': fields.DateTime,
    'links': fields.Nested({
        'friends': fields.Url('user_friends'),
        'posts': fields.Url('user_posts'),
    }),
}

class User(Resource):
    def __init__(self):
      self.post_parser = reqparse.RequestParser()
	 self.post_parser.add_argument(
	     'username', dest='username',
	     location='form', required=True,
	     help='The user\'s username',
	 )
	 self.post_parser.add_argument(
	     'email', dest='email',
	     type=email, location='form',
	     required=True, help='The user\'s email',
	 )
	 self.post_parser.add_argument(
	     'user_priority', dest='user_priority',
	     type=int, location='form',
	     default=1, choices=range(5), help='The user\'s priority',
	 )
      super(User,self).__init__()
    @marshal_with(user_fields)
    def post(self):
        args = self.post_parser.parse_args()
        user = create_user(args.username, args.email, args.user_priority)
        return user

    @marshal_with(user_fields)
    def get(self, id):
        args = selfpost_parser.parse_args()
        user = fetch_user(id)
        return user

```
      
**认证**
在 REST 服务器中的路由都是由 HTTP 基本身份验证保护着。
**pip install flask-httpauth**
因为 Resouce 类是继承自 Flask 的 MethodView，它能够通过定义 decorators 变量并且把装饰器赋予给它:
```

from flask.ext.httpauth import HTTPBasicAuth
# ...
auth = HTTPBasicAuth()
# ...

class TaskAPI(Resource):
    decorators = [auth.login_required]
    # ...

class TaskAPI(Resource):
    decorators = [auth.login_required]
    # ...

```
```

from flask.ext.httpauth import HTTPBasicAuth
auth = HTTPBasicAuth()

@auth.get_password
def get_password(username):
    if username == 'miguel':
        return 'python'
    return None

@auth.error_handler
def unauthorized():
    return {'error': 'Unauthorized access'}, 401

```
  
当然，你也可以使用自定义装饰器进行权限控制，比如
```

def authenticate(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if not getattr(func, 'authenticated', True):
            return func(*args, **kwargs)

        acct = basic_authentication()  # custom account lookup function

        if acct:
            return func(*args, **kwargs)

        restful.abort(401)
    return wrapper


class Resource(restful.Resource):
    method_decorators = [authenticate]   # applies to all inherited resources

```
##  Output Fields 
输出数据格式化
```

from flask_restful import Resource, fields, marshal_with

resource_fields = {
    'name': fields.String,
    'address': fields.String,
    'date_updated': fields.DateTime(dt_format='rfc822'),
}

class Todo(Resource):
    @marshal_with(resource_fields, envelope='resource')
    def get(self, **kwargs):
        return db_get_todo()  # Some function that queries the db

```
重命名属性：
```

fields = {
    'name': fields.String(attribute='private_name'),
    'address': fields.String,
}

```
默认值：
```

fields = {
    'name': fields.String(default='Anonymous User'),
    'address': fields.String,
}

```
###  Custom Fields & Multiple Values 
```

class UrgentItem(fields.Raw):
    def format(self, value):
        return "Urgent" if value & 0x01 else "Normal"

class UnreadItem(fields.Raw):
    def format(self, value):
        return "Unread" if value & 0x02 else "Read"

fields = {
    'name': fields.String,
    'priority': UrgentItem(attribute='flags'),
    'status': UnreadItem(attribute='flags'),
}

```
###  Url & Other Concrete Fields 
```

class RandomNumber(fields.Raw):
    def output(self, key, obj):
        return random.random()

fields = {
    'name': fields.String,
    # todo_resource is the endpoint name when you called api.add_resource()
    'uri': fields.Url('todo_resource'),
    'random': RandomNumber,
}

```
默认 fields.Url returns a relative uri. 如果要生成绝对uri：设定 absolute=True
fields.Uri 是一个用于生成一个 URL 的特定的参数。 **它需要的参数是 endpoint**。
```

fields = {
    'uri': fields.Url('todo_resource', absolute=True)
    'https_uri': fields.Url('todo_resource', absolute=True, scheme='https')
}

```
###  复杂结构 
```

>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> resource_fields = {'name': fields.String}
>>> resource_fields['address'] = {}
>>> resource_fields['address']['line 1'] = fields.String(attribute='addr1')
>>> resource_fields['address']['line 2'] = fields.String(attribute='addr2')
>>> resource_fields['address']['city'] = fields.String
>>> resource_fields['address']['state'] = fields.String
>>> resource_fields['address']['zip'] = fields.String
>>> data = {'name': 'bob', 'addr1': '123 fake street', 'addr2': '', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> json.dumps(marshal(data, resource_fields))
'{"name": "bob", "address": {"line 1": "123 fake street", "line 2": "", "state": "NY", "zip": "10468", "city": "New York"}}'

```
###  嵌套Field 
```

user_fields = {
    'id': fields.Integer,
    'name': fields.String,
}

user_list_fields = {
    fields.List(fields.Nested(user_fields)),
}

```
##  自定义错误处理 
```

def log_exception(sender, exception, **extra):
    """ Log an exception to our logging framework """
    sender.logger.debug('Got exception during processing: %s', exception)

from flask import got_request_exception
got_request_exception.connect(log_exception, app)

```
自定义错误消息：
```

errors = {
    'UserAlreadyExistsError': {
        'message': "A user with that username already exists.",
        'status': 409,
    },
    'ResourceDoesNotExist': {
        'message': "A resource with that ID no longer exists.",
        'status': 410,
        'extra': "Any extra information you want.",
    },
}
app = Flask(__name__)
api = flask_restful.Api(app, errors=errors)

```

##  完整Demo 
http://flask-restful.readthedocs.io/en/0.3.5/intermediate-usage.html

其它参考：http://www.pythondoc.com/flask-restful/index.html