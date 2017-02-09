title: python学习笔记54 

#  Python学习笔记之gevent协程与并发库实战 
gevent是一个使用完全同步编程模型的可扩展的异步I/O框架。
##  简单的echo 服务器 
```

from gevent.server import StreamServer
 
def connection_handler(socket, address):
    for l in socket.makefile('r'):
        socket.sendall(l)
 
if __name__ == '__main__':
    server = StreamServer(('0.0.0.0', 8000), connection_handler)
    server.serve_forever()

```
在这个例子中，我们并行发出100个web请求:
```

from gevent import monkey
monkey.patch_all()
 
import urllib2
from gevent.pool import Pool
 
def download(url):
    return urllib2.urlopen(url).read()
 
if __name__ == '__main__':
    urls = ['http://httpbin.org/get'] * 100
    pool = Pool(20)
    print pool.map(download, urls)

```
有些奇怪` monkey.patch_all()的调用 `？不用担心，这可不像你每天打的猴子补丁(译注：monkey patching，即动态修改执行代码)。这仅仅是Python发行版恰好要打的一组猴子补丁。

参考：http://www.oschina.net/translate/gevent-asynchronous-io-made-easy

##  Gevent ZeroMQ 
**ZeroMQ 被它的作者描述为 “一个表现得像一个并发框架的socket库”。 它是一个非常强大的，为构建并发和分布式应用的消息传递层。**
ZeroMQ提供了各种各样的socket原语。最简单的是请求-应答socket对 (Request-Response socket pair)。**一个socket有两个方法send和recv， 两者一般都是阻塞操作。但是Travis Cline 的一个杰出的库弥补了这一点，这个库使用gevent.socket来以非阻塞的方式 轮询ZereMQ socket。**通过命令：
```

pip install gevent-zeromq

```
```

# Note: Remember to ``pip install pyzmq gevent_zeromq``
import gevent
from gevent_zeromq import zmq

# Global Context
context = zmq.Context()

def server():
    server_socket = context.socket(zmq.REQ)
    server_socket.bind("tcp://127.0.0.1:5000")

    for request in range(1,10):
        server_socket.send("Hello")
        print('Switched to Server for %s' % request)
        # Implicit context switch occurs here
        server_socket.recv()

def client():
    client_socket = context.socket(zmq.REP)
    client_socket.connect("tcp://127.0.0.1:5000")

    for request in range(1,10):

        client_socket.recv()
        print('Switched to Client for %s' % request)
        # Implicit context switch occurs here
        client_socket.send("World")

publisher = gevent.spawn(server)
client    = gevent.spawn(client)

gevent.joinall([publisher, client])

```
##  简单server 
```

# On Unix: Access with ``$ nc 127.0.0.1 5000``
# On Window: Access with ``$ telnet 127.0.0.1 5000``
from gevent.server import StreamServer

def handle(socket, address):
    socket.send("Hello from a telnet!\n")
    for i in range(5):
        socket.send(str(i) + '\n')
    socket.close()

server = StreamServer(('127.0.0.1', 5000), handle)
server.serve_forever()

```

##  Websockets 
**运行Websocket的例子需要gevent-websocket包**。
```

# Simple gevent-websocket server
import json
import random
from gevent import pywsgi, sleep
from geventwebsocket.handler import WebSocketHandler
class WebSocketApp(object):
    ` 'Send random data to the websocket `'

    def __call__(self, environ, start_response):
        ws = environ['wsgi.websocket']
        x = 0
        while True:
            data = json.dumps({'x': x, 'y': random.randint(1, 5)})
            ws.send(data)
            x += 1
            sleep(0.5)

server = pywsgi.WSGIServer(("", 10000), WebSocketApp(),
    handler_class=WebSocketHandler)
server.serve_forever()

```

HTML Page:

```

<html>
    <head>
        <title>Minimal websocket application</title>
        <script type="text/javascript" src="jquery.min.js"></script>
        <script type="text/javascript">
        $(function() {
            // Open up a connection to our server
            var ws = new WebSocket("ws://localhost:10000/");

            // What do we do when we get a message?
            ws.onmessage = function(evt) {
                $("#placeholder").append('<p>' + evt.data + '</p>')
            }
            // Just update our conn_status field with the connection status
            ws.onopen = function(evt) {
                $('#conn_status').html('<b>Connected</b>');
            }
            ws.onerror = function(evt) {
                $('#conn_status').html('<b>Error</b>');
            }
            ws.onclose = function(evt) {
                $('#conn_status').html('<b>Closed</b>');
            }
        });
    </script>
    </head>
    <body>
        <h1>WebSocket Example</h1>
        <div id="conn_status">Not Connected</div>
        <div id="placeholder" style="width:600px;height:300px;"></div>
    </body>
</html>

```

##  聊天server 
最后一个生动的例子，实现一个实时聊天室。**运行这个例子需要 Flask** (你可以使用Django, Pyramid等，但不是必须的)。 对应的Javascript和HTML文件可以在 这里找到。
```

# Micro gevent chatroom.
# ----------------------

from flask import Flask, render_template, request

from gevent import queue
from gevent.pywsgi import WSGIServer

import simplejson as json

app = Flask(__name__)
app.debug = True

rooms = {
    'topic1': Room(),
    'topic2': Room(),
}

users = {}

class Room(object):

    def __init__(self):
        self.users = set()
        self.messages = []

    def backlog(self, size=25):
        return self.messages[-size:]

    def subscribe(self, user):
        self.users.add(user)

    def add(self, message):
        for user in self.users:
            print(user)
            user.queue.put_nowait(message)
        self.messages.append(message)

class User(object):

    def __init__(self):
        self.queue = queue.Queue()

@app.route('/')
def choose_name():
    return render_template('choose.html')

@app.route('/<uid>')
def main(uid):
    return render_template('main.html',
        uid=uid,
        rooms=rooms.keys()
    )

@app.route('/<room>/<uid>')
def join(room, uid):
    user = users.get(uid, None)

    if not user:
        users[uid] = user = User()

    active_room = rooms[room]
    active_room.subscribe(user)
    print('subscribe %s %s' % (active_room, user))

    messages = active_room.backlog()

    return render_template('room.html',
        room=room, uid=uid, messages=messages)

@app.route("/put/<room>/<uid>", methods=["POST"])
def put(room, uid):
    user = users[uid]
    room = rooms[room]

    message = request.form['message']
    room.add(':'.join([uid, message]))

    return ''

@app.route("/poll/<uid>", methods=["POST"])
def poll(uid):
    try:
        msg = users[uid].queue.get(timeout=10)
    except queue.Empty:
        msg = []
    return json.dumps(msg)

if __name__ == "__main__":
    http = WSGIServer(('', 5000), app)
    http.serve_forever()

```