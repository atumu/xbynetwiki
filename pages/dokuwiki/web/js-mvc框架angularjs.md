title: js-mvc框架angularjs 

#  js-MVC框架AngularJS 
##  概述 

AngularJS如果为创建web应用而设计，那它就是HTML的套路了。具有数据绑定, MVW, MVVM, MVC, 依赖注入的声明式模板和出色的可测试性都是用纯客户端 JavaScript来实现的! AngularJS 是一个创建富客户端应用的**JavaScript MVC框架。**你仍然需要具有服务端后台，但大多数的用户交互逻辑将优雅地放到客户端上处理。
AngularJS是一个开源的web应用框架，由Google和社区进行维护，它可以创建单页的应用程序，一个页面的应用仅仅需要HTML,CSS和JavaScript在客户端。它的目标是增强页面的模型-视图-控制（MVC）的功能，为简化开发和测试。
**Angular 提供了：**
结构模型的引入（MVC,SPA等）
增强HTML支持新特性。
**它通过双向数据绑定使你的UI（视图层）与你的JavaScript对象（模型层）的数据自动同步。**
避免直接DOM操作来避免很难调试不可追踪的代码。
包含低耦合和高可复用性
应用程序内部规则测试
视图模板更接近服务器端模板

AngularJS 是基于**声明式**编程模式 是用户可以基于业务逻辑进行开发. 该框架基于HTML的内容填充并做了**双向数据绑定**从而完成了**自动数据同步机制**. 最后, AngularJS 强化的DOM操作增强了可测试性.

###  双向数据绑定: 

数据绑定可能是AngularJS里最酷，最实用的功能。 它将节省你大量的样板代码编写。 一个典型的Web应用程序可以包含多达80％的代码基础，如遍历，操作，并听取了监听DOM。 数据绑定使得不用编写这些代码，这样你就可以专注于你的应用程序,AngularJS双向数据绑定会处理DOM和模型之间的同步，反之亦然。
![](/data/dokuwiki/web/pasted/20150728-025345.png?400*200)
###  MVC 

AngularJS引入了软件设计的ＭＶＣ模式.这对于使用者来说仁者见仁智者见智. AngularJS并不是完全的ＭＶＣ而是 MVVM (Model-View-ViewModel).
**模型**
model就是数据模型 就是一些JavaScript 对象. 没必要从父类继承，代理包装亦或是使用getter/setter来使用. 使用vanilla JavaScript 十分方便便捷.
**视图**
视图就是提供特殊数据或方法来支持特定场景的对象.
视图对象就是 $scope. $scope就是个简单的js对象，提供一些简单的ＡＰＩ监控其状态.
**业务控制**
控制器起到设置 $scope对象的初始状态及后续的动作关联。 
**页面**
在.AngularJS处理完相关的业务逻辑进行ＨＴＭＬ模式的展示。
这样就奠定了应用的架构.  $scope对象拥有数据的引用关系, 控制器定义行为, 视图处理页面展示布局以及相应的处理跳转.

###  客户机/服务器架构的web应用程序 
世界已经被改变，部分应用的逻辑已经被移到客户端。当我们需要以某种方式处理来自服务器的所有数据时，这里就描述这种情形。JS MVC框架鼓励把表现层逻辑从服务器端移动到客户端，实现了RIA（Rich-Internet-Apllication），传统web应用程序下的客户机/服务器架构和JS MVC下的客户机/服务器架构都基于web应用。
![](/data/dokuwiki/web/pasted/20150728-025917.png)

较流行的一种包含客户端服务端的模式是** 后端RESTful API** ` 通过** JSON**发送数据模型 ` **客户端使用MVC模式** 处理应用.
Client-side MVC with server-side RESTful API
![](/data/dokuwiki/web/pasted/20150728-030036.png)
**Data Flow**
![](/data/dokuwiki/web/pasted/20150728-030053.png?800*300)
###  RESTful Interation 
AngularJS使用angular-resource(ngResource)模块来提供RESTful交互功能，该模块表示一个REST资源并提供帮助方法(GET/POST/PUT/DELETE)来轻松的实现RESTful交互。另外也提供其它的可选模块。
###  什么时候需要使用一个JS MV*框架 

下面这个列表并不完备，但是我们希望它能提供充分的理由帮你决定是否在你的应用中应该使用一个MVC框架:
你的应用需要异步连接到后台
你的应用有这样的功能，它不需要重新载入整个页面(比如给博文增加一条评论，无下限滚动)
多数视图或者数据操作将会在浏览器内完成，而不是在服务器端完成
同样的数据在页面上需要进行不同方式的渲染
你的应用有许多琐碎的交互来修改数据(按钮, 开关)
满足这些情况的比较好的web应用的例子有Google Docs，Gmail或者Spotify。