title: angularjs启动加载流程 
author:xby@xbynet.net
createAt:2016-11-28 23:55:43
modifyAt:2016-11-28 23:55:43
location:dokuwiki\web\AngularJS启动加载流程

#  AngularJS启动加载流程 
##  AngularJS应用的四个阶段 
一、启动阶段

大家应该都知道，当浏览器加载一个HTML页面时，它会将HMTL页面先解析成DOM树，然后逐个加载DOM树中的每一个元素节点。我们可以把AngularJS当做一个类似jQuery的js库，我们通过< script>标签引入到HTML中，那么此时Angular就做为一个普通的DOM节点等待浏览器解析，当浏览器解析到这个节点时，发现它是一个js文件，那么浏览器会停止解析剩余的DOM节点，开始执行这个js（即angular.js），**同时Angular会设置一个事件监听器来监听浏览器的DOMContentLoaded事件。当Angular监听到这个事件时，就会启动Angular应用。**

二、初始化阶段
Angular开始启动后，它会**查找ng-app指令**，然后初始化一系列必要的组件（即**$injector、$compile服务以及$rootScope**），接着重新开始解析DOM树。

三、编译、链接
**$compile服务通过遍历DOM树**的方式查找有声明指令的DOM元素。当碰到带有一个或多个指令的DOM元素时，它会排序这些指令（基于指令的priority优先级），然后使用**$injector服务查找和收集指令的compile函数并执行它**。**每个节点的编译方法运行之后，$compile服务就会调用链接函数**。这个链接函数为绑定了封闭作用域的指令设置监控。这一行为会**创建实时视图**。
最后，在$compile服务完成后，AngularJS运行时就准备好了。

四、运行阶段
 Angular提供了自己的**事件循环**。**指令自身会注册事件监听器，因此当事件被触发时，指令函数就会运行在AngularJS的$digest循环中**。$digest循环会等待$watch表达式列表，当检测到模型变化后，就会调用$watch函数，然后再次查看$watch列表以确保没有模型被改变。
 一旦$digest循环稳定下来，并且检测到没有潜在的变化了，执行过程就会离开Angular上下文并且通常会回到浏览器中，DOM将会被渲染到这里。
![](/data/dokuwiki/web/pasted/20151118-215214.png)
##  启动流程解析 
```

<!doctype html>
<html ng-app>
  <head>
    <script src="angular.js"></script>
  </head>
  <body>
    <png-init=" name='World' ">Hello ![](/data/dokuwikiname)!</p>
  </body> 
</html>

```
 当你用浏览器去访问index.html的时候，浏览器依次做了如下一些事情：
  * 加载html，然后解析成DOM；
  * 加载angular.js脚本；
  * AngularJS等待DOMContentLoaded事件的触发；（注意是等DOM加载完毕后Angular才开始工作）
  * AngularJS寻找ng-app指令，根据这个指令确定应用程序的边界；
  * 使用ng-app中指定的模块配置$injector；
  * 使用$injector创建$compile服务和$rootScope；
  * 使用$compile服务编译DOM并把它链接到$rootScope上；
  * ng-init指令对scope里面的变量name进行赋值；
  * 对表达式&lt;nowiki&gt;![](/data/dokuwikiname)&lt;/nowiki&gt;进行替换，于是乎，显示为“Hello World!”     
　　整个过程可以用这张图来表示：
![](/data/dokuwiki/web/pasted/20151118-213742.png)

那么AngularJS又是如何和浏览器的事件回路来交互的呢？
1.  浏览器的事件回路一直等待着事件的触发，事件包括用户的交互操作、定时事件或者网络事件（如服务器的响应等）；
2.  一旦有事件触发，就会进入到Javascript的context中，一般通过回调函数来修改DOM；
3.  等到回调函数执行完毕之后，浏览器又根据新的DOM来渲染新的页面。
正如下面一张图所示，交互过程主要由几个循环组成：
![](/data/dokuwiki/web/pasted/20151118-213906.png)
 AngularJS修改了一般的Javascript工作流，并且提供了它自己的事件处理机制。这样就把Javascript的context分隔成两部分，一部分是原生的Javascript的context，另一部分是AngularJS的context。**只有处在AngularJS的context中的操作才能享受到Angular的data-binding、exception handling、property watching等服务**，但是对于外来者（如原生的Javascript操作、自定义的事件回调、第三方的库等）Angular也不是一概不接见，可以**使用AngularJS提供的` $apply()函数 `将这些外来者包进AngularJS的context中，让Angular感知到他们产生的变化。**
接下来，让我们一起来看看交互过程中的这几个循环是怎么工作的？
　　1.  首先，浏览器会一直处于**监听状态**，一旦有事件被触发，就会被加到一个**event queue**中，event queue中的事件会一个一个的执行。
　　2.  event queue中的事件如果是**被$apply()包起来**的话，就会**进入到AngularJS的context中**，这里的fn()是我们希望在AngularJS的context中执行的函数。
　　3.  AngularJS将执行fn()函数，通常情况下，这个函数会改变应用的某些状态。
　　4.  然后AngularJS会进入到由两个小循环组成的**$digest循环**中，一个循环是用来处理$evalAsync队列（用来schedule一些需要在渲染视图之前处理的操作，通常通过setTimeout(0)实现，速度会比较慢，可能会出现视图抖动的问题）的，一个循环是处理**$watch列表**（是一些表达式的集合，一旦有改变发生，那么$watch函数就会被调用）的。$digest循环会一直迭代知道$evalAsync队列为空并且$watch列表也为空的时候，即model不再有任何变化。
　　5.  **一旦AngularJS的$digest循环结束**，整个执行就会离开AngularJS和Javascript的context，紧接着浏览器就会把数据改变后的**视图重新渲染出来。**
```

<!doctype html>
<html ng-app>
  <head>
    <script src="angular.js"></script>
  </head>
  <body>
    <input ng-model="name">
    <p>Hello ![](/data/dokuwikiname)!</p>
  </body> 
</html>

```
这段代码和上一段代码唯一的区别就是有了一个input来接收用户的输入。在用浏览器去访问这个html文件的时候，input上的ng-model指令会给input绑上keydown事件，并且会给name变量建议一个$watch来接收变量值改变的通知。在交互阶段主要会发生以下一系列事件：
　　1.  当用户按下键盘上的某一个键的时候（比如说A），触发input上的keydown事件；
　　2.  input上的指令察觉到input里值的变化，调用$apply(“name=‘A’”)更新处于AngularJS的context中的model；
　　3.  AngularJS将’A’赋值给name；
　　4.  $digest循环开始，$watch列表检测到name值的变化，然后通知![](/data/dokuwikiname)表达式，更新DOM；
　　5.  退出AngularJS的context，然后退出Javascript的context中的keydown事件；
　　6.  浏览器重新渲染视图。

##  AngularJS三种启动方式 
方式 1：自动启动
Angular会自动的找到ng-app，将它作为启动点，自动启动
```

<!DOCTYPE html>
<html ng-app="myModule">

<head>
    <title>New Page</title>
    <meta charset="utf-8" />
    <script type="text/javascript" src="../../vendor/bower_components/angular/angular.min.js"></script>
    <script type="text/javascript" src="./02.boot1.js"></script>
</head>

<body>
    <div ng-controller="MyCtrl">
        <span>![](/data/dokuwikiName)</span>
    </div>
</body>
</html>

```
```

var myModule = angular.module("myModule", []);
myModule.controller('MyCtrl', ['$scope',
    function($scope) {
        $scope.Name = "Puppet";
    }
]);

```

方式 2：手动启动
在没有ng-app的情况下，只需要在js中添加一段注册代码即可
```

<body>
    <div ng-controller="MyCtrl">
        <span>![](/data/dokuwikiName)</span>
    </div>
</body>

var myModule = angular.module("myModule", []);
myModule.controller('MyCtrl', ['$scope',
    function($scope) {
        $scope.Name = "Puppet";
    }
]);

```
```

/**
 * 这里要用ready函数等待文档初始化完成
 */
angular.element(document).ready(function() {
    angular.bootstrap(document, ['myModule']);
});

```

方式 3：多个ng-app
ng中，` **angular的ng-app是无法嵌套使用的** `，**在不嵌套的情况下有多个ng-app，他默认只会启动第一个ng-app，第二个第三个需要手动启动**(注意，不要手动启动第一个，虽然可以运行，但会抛异常)
```

<body>
    <div id="app1" ng-app="myModule1">
        <div ng-controller="MyCtrl">
            <span>![](/data/dokuwikiName)</span>
        </div>
    </div>
    <div id="app2" ng-app="myModule2">
        <div ng-controller="MyCtrl">
            <span>![](/data/dokuwikiName)</span>
        </div>
    </div>
</body>

```
```

/**
 * 第一个APP
 * @type {[type]}
 */
var myModule1 = angular.module("myModule1", []);
myModule1.controller('MyCtrl', ['$scope',
    function($scope) {
        $scope.Name = "Puppet";
    }
]);
// angular.element(document).ready(function() {
//     angular.bootstrap(app1, ['MyModule1']);
// });

/**
 * 第二个APP
 * @type {[type]}
 */
var myModule2 = angular.module("myModule2", []);
myModule2.controller('MyCtrl', ['$scope',
    function($scope) {
        $scope.Name = "Vincent";
    }
]);
angular.element(document).ready(function() {
    angular.bootstrap(app2, ['myModule2']);
});

```

参考：http://www.cnblogs.com/leo_wl/p/3446075.html
http://segmentfault.com/a/1190000002788586
http://my.oschina.net/brant/blog/419641