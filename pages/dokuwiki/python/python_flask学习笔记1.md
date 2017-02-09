modifyAt:2016-12-18 13:23:20
title:flaskWeb开发读书笔记1入门
createAt:2016-12-18 12:57:44
location:dokuwiki/python/python_flask学习笔记1
author:xbynet

代码示例:https://github.com/miguelgrinberg/flasky
Flask官网:http://flask.pocoo.org
Flask有两个主要依赖：路由、调试和Web服务器网关接口(WSGI)的子系统由Werkzeug提供，模板系统由Jinja2提供。
安装pip install flask
##  使用虚拟环境 
虚拟环境是Python解释器的一个私有副本，在这个环境中你可以安装私有包，而且不会影响到系统中安装的全局Python解释器。
这样就可以解决同一个依赖不同版本冲突的矛盾。为每个程序单独创建虚拟环境可以保证程序只能访问虚拟环境中的包，从而保证全局解释器的干净整洁。
虚拟环境使用第三方实用工具virtualenv创建。输入以下命令可以检查是否已经安装:virtualenv --version
Python3.3通过venv模块原生支持虚拟环境，命令为pyvenv。pyvenv可以完全替代virtualenv。不过我们还是介绍virtualenv。
  * 安装pip install virtualenv
  * 创建虚拟环境，在项目目录下执行：virtualenv venv(这是虚拟环境名字)。
  * 在使用这个虚拟环境之前，你需要先将其激活:
linux: source venv/bin/active
windows: venv\Scripts\active
虚拟环境被激活后，其中Python解释器的路径就被添加进PATH中，但这种改变只会影响当前的命令行会话。
  * 如果想退出虚拟环境:输入:deactive即可

##  flask程序的基本结构 
```

from flask import Flask
app=Flask(__name__)#__name__参数代表程序主模块或包的名字。Flask用这个参数决定程序的根目录，以便稍后能够找到相对于根目录的资源文件位置。
if __name__='__main__':
  app.run(debug=True)#服务器启动后会进入轮询，等待并处理请求

```
路由:处理URL和函数之间映射关系
在Flask中通过@app.route修饰器或者app.add_url_rule()可以将函数注册为路由。
被注册为处理URL请求的函数称为视图函数.该函数的返回值作为HTTP响应内容。

##  请求与响应以及相关上下文 
为了避免大量可有可无的参数把视图函数弄得一团糟，Flask使用上下文临时把某些对象变为全局可访问。有了上下文就可以写出下面的视图函数。
```

from flask import request
@app.route('/')
def index():
	user_agent=request.headers.get('User-Agent')
	return '%s' % user_agent

```
**Flask使用上下文让特定的变量在一个线程中全局可访问，与此同时却不会干扰其他线程。**
Flask中有两种上下文:**程序上下文和请求上下文。**
上下文变量名 类别 说明

  * current_app 程序上下文 当前激活程序的程序实例代理
  * g 		程序上下文 处理请求时用作**临时存储**的对象，**每次请求都会重设这个变量**
  * request 	请求上下文 请求对象，封装了客户端发出的HTTP请求的内容
  * session 	请求上下文 用户会话，用于存储请求之间需要记住的值的字典
 
**` Flask 在分发请求之前激活（或推送）程序和请求上下文，请求处理完成后再将其删除`。程序上下文被推送后，就可以在线程中使用current_app 和g 变量。类似地，请求上下文被
推送后，就可以使用request 和session 变量。如果使用这些变量时我们没有激活程序上
下文或请求上下文，就会导致错误。**

上下文获取方式:

  * 正常服务器接收请求创建上下文
  * 手动创建app.app_context()
  * 测试时app.test_app_context()
  
获取了上下文之后需要压入栈。如下:
```

app_ctx=app.app_context()
app_ctx.push()
current_app.name
app_ctx.pop()

```
获取所有URL： app.url_map
##  请求钩子 
有时在处理请求之前或之后执行代码会很有用。Flask提供了注册通用函数的功能。
支持以下4种请求钩子函数:

  * @app.before_first_request:在处理第一个请求之前运行
  * @app.before_request:在每次请求之前运行
  * @app.after_request:如果没有未处理的异常抛出，在每次请求之后运行
  * @app.teardown_request:即使有未处理的异常抛出，也在每次请求之后运行
 
在请求钩子函数和视图函数之间共享数据一般使用上下文全局变量g。

##  响应方式 
支持几种形式：(字符串,状态码)元组,render_template,redirect,abort,make_response,jsonify,send_file等。
其中abort()不会将控制权返回给视图函数，而是抛出异常。make_response函数返回一个response对象，该对象可以用于进一步设置返回的头部。send_file用于返回图片验证码或者文件下载等。

```
@app.route('/')
def index():
	return '<h1>jaaa</h1>',200
	#return redirect('http://wiki.xby1993.net')
	#abort(401)
    #send_file(img_io, mimetype='image/png',cache_timeout=0)
    #jsonify({'status':'ok'})
	#response=make_response('<h1>xby1993<h1>') 
    #response.set_cookie('abc',11) 
    #return response

```
##  Flask扩展 
扩展安装:pip install flask_扩展名
引入方式 from flask.ext.扩展名 import 。。。但是新版本Flask已经不推荐使用flask.ext.作为扩展的模块前缀了，而是推荐使用例如flask_script作为模块名。

注:后续flask会集成click作为命令行工具，后续请不要使用flask_script而是使用内置的click集成。

例如

```
pip install flask-script
form flask_script import Manager
```

使用Flask-Script扩展支持命令行选项:

```
form flask.ext.script import Manager
manager=Manager(app)
....
if __name__=='__main__':
	manager.run() #manager.run启动后才能解析命令
```
