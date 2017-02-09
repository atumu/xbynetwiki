title: python学习笔记032 

#  Python学习笔记之Flask与JSON支持 
##  JSON 视图函数 
这个服务端函数接收两个数字形式的 URL 参数， **然后将这两个数字相加并以 JSON 对象的形式返回给应用。**
```

from flask import Flask, jsonify, render_template, request
app = Flask(__name__)
@app.route('/_add_numbers')
def add_numbers():
    a = request.args.get('a', 0, type=int)
    b = request.args.get('b', 0, type=int)
    return jsonify(result=a + b)
@app.route('/')
def index():
    return render_template('index.html')

```
**注意，这里我们使用不会抛出错误的 get() 方法。 如果对应的键不存在，一个默认值(这里是 0)将hi被返回。**更进一步，我们还可以将值转换为一个特定类型(就像我们这里的 int 类型)。这对于由脚本(APIs,JavaScript等)激发的代码来说是个非常顺手的工具，因为在这种情况下您不需要特别的错误报告。

from flask import json,jsonify
```

<script type=text/javascript>
doSomethingWith(![](/data/dokuwiki user.username|tojson|safe ));
</script>

```

**flask.json.jsonify**(*args, * *kwargs)
Creates a Response with the JSON representation of the given arguments with an **application/json mimetype**. The arguments to this function are the same as to the dict constructor.
Example usage:
```

from flask import jsonify
@app.route('/_get_current_user')
def get_current_user():
    return jsonify(username=g.user.username,
                   email=g.user.email,
                   id=g.user.id)
  
from flask import make_response

@app.errorhandler(404)
def not_found(error):
    return make_response(jsonify({'error': 'Not found'}), 404)

```
This will send a JSON response like this to the browser:
```

{
    "username": "admin",
    "email": "admin@localhost",
    "id": 42
}

```
flask.json则类似pythn内置的json模块
