title: tornado 

#  Tornado（支持win） 
Tornado 是一个开源的可伸缩的、非阻塞式的 web 服务器和工具集，它驱动了 FriendFeed 。因为它使用了 epoll 模型且是非阻塞的，它可以处理数以千计的并发固定连接，这意味着它对实时 web 服务是理想的。把 Flask 集成这个服务是直截了当的:

```

from tornado.wsgi import WSGIContainer
from tornado.httpserver import HTTPServer
from tornado.ioloop import IOLoop
from yourapplication import app

http_server = HTTPServer(WSGIContainer(app))
http_server.listen(5000)
IOLoop.instance().start()

```