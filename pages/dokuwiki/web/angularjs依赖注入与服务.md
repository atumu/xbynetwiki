title: angularjs依赖注入与服务 

#  AngularJS依赖注入与服务 
##  依赖注入injector 
Dependency Injection (DI,依赖注入)是一种软件设计模式,用于处理如何让对象获得其依赖(对象的)引用。
对象或者函数只有以下3种获取其依赖(的对象)引用的方式:
  * 依赖可以被使用者自己创建,通过 new 操作符.
  * 依赖可以通过全局变量(如 window)来查找并引用
  * 依赖可以在需要的地方通过参数被传入
AngularJS的依赖注入是通过第三种方式实现的。
要管理依赖创建的责任,每个 Angular 应用程序都有一个 injector 。该injector是一个 service locator(定位器) ,负责创建并查找依赖。

angularjs中与DI相关有` angular.module()、angular.injector()、 $injector、$provide `。
对于一个**DI容器**来说，必须具备3个要素：**服务的注册、依赖关系的声明、对象的获取**。比如spring中，服务的注册是通过xml配置文件的<bean>标签或是注解@Repository、@Service、@Controller、@Component实现的；对象的获取可以ApplicationContext.getBean()实现；依赖关系的声明，即可以在xml文件中配置，也可以使用@Resource等注解在java代码中声明。
在angular中，**module和$provide相当于是服务的注册**；injector用来获取对象（angular会自动完成依赖的注入）；依赖关系的声明在angular中有3种方式。下面从这3个方面:隐式，显式，数组形式。

###  使用依赖注入场景 
 依赖注入在angular应用底层代码中使用很频繁。我们可以在定义组件或者在模块的run和config块中使用。
- 可在angular组件(**控制器、服务、过滤器、指令、动画**等)的构造方法或工厂方法中**声明依赖关系**。可以向这些组件中**注入服务(service)或者值（value）类型的依赖**。
-  可为控制器添加一些特殊的依赖，我们将在下面具体介绍。
-  可在模块**的run方法中**定义依赖，**可被注入的依赖包括服务(service)、值(value)和参量(constant)**。注意.` 在run方法中不能注入"provider"类型的依赖. `
-  可在模块的**config中声明“provider”和“constant”类型的依赖**，但不能注入service和value组件。

**工厂方法**
 angular中的**指令、服务或者过滤器**可以通过工厂方法定义。**工厂方法是模块（module）的子方法**。通常推荐按下面的方式使用工厂方法:
```

angular.module('myModule', [])
.factory('serviceId', ['depService', function(depService) {
  // ...
}])
.directive('directiveName', ['depService', function(depService) {
  // ...
}])
.filter('filterName', ['depService', function(depService) {
  // ...
}]);

```

**模块方法**
angular模块提供了**config和run方法**方便开发者为模块**添加配置和运行上下文的设置**。和工厂方法类似，我们可以在config和run方法中定义依赖。例如:
```

angular.module('myModule', [])
.config(['depProvider', function(depProvider) {
  // ...
}])
.run(['depService', function(depService) {
  // ...
}]);

```

**控制器**
 控制器的构造方法主要用来定义模板中所用的模型数据和行为。通常推荐按如下用法使用构造器:
```

someModule.controller('MyController', ['$scope', 'dep1', 'dep2', function($scope, dep1, dep2) {
  ...
  $scope.aMethod = function() {
    ...
  }
  ...
}]);

```
angular应用中可存在多个控制器实例，**` 而服务组件是单例 `**.
除了service, value类的依赖， 控制器中还注入一下依赖:
- $scope:  控制器是与某个DOM元素通过作用域相关联。其他类型的组件只能访问$rootScope服务。
- resolves : 如果在路由中实例化控制器，那么在路由里面解析的任何值域都可以作为依赖注入到控制器。

**值**
值是简单的JavaScript对象，它是用来将值传递过程中的配置相位控制器。 config方法不能传递值依赖，一般用在run方法中
```

//define a module
var mainApp = angular.module("mainApp", []);
//create a value object as "defaultInput" and pass it a data.
mainApp.value("defaultInput", 5);
...
//inject the value in the controller using its name "defaultInput"
mainApp.controller('CalcController', function($scope, CalcService, defaultInput) {
      $scope.number = defaultInput;
      $scope.result = CalcService.square($scope.number);

      $scope.square = function() {
      $scope.result = CalcService.square($scope.number);
   }
});

```

**工厂**
工厂是用于返回函数的值。它根据需求创造值，每当一个服务或控制器需要。它通常使用一个工厂函数来计算并返回对应值
```

//define a module
var mainApp = angular.module("mainApp", []);
//create a factory "MathService" which provides a method multiply to return multiplication of two numbers
mainApp.factory('MathService', function() {     
   var factory = {};  
   factory.multiply = function(a, b) {
      return a * b 
   }
   return factory;
}); 

//inject the factory "MathService" in a service to utilize the multiply method of factory.
mainApp.service('CalcService', function(MathService){
      this.square = function(a) { 
      return MathService.multiply(a,a); 
   }
});
...

```

**服务**
服务是一个单一的JavaScript包含了一组函数对象来执行某些任务。服务使用service()函数，然后注入到控制器的定义。
```

//define a module
var mainApp = angular.module("mainApp", []);
...
//create a service which defines a method square to return square of a number.
mainApp.service('CalcService', function(MathService){
      this.square = function(a) { 
      return MathService.multiply(a,a); 
   }
});
//inject the service "CalcService" into the controller
mainApp.controller('CalcController', function($scope, CalcService, defaultInput) {
      $scope.number = defaultInput;
      $scope.result = CalcService.square($scope.number);

      $scope.square = function() {
      $scope.result = CalcService.square($scope.number);
   }
});

```

**提供者**
**提供者所使用的AngularJS内部创建过程中配置阶段的服务**，工厂等(相AngularJS引导自身期间)。下面提到的脚本，可以用来创建，我们已经在前面创建MathService。**提供者是一个特殊的工厂方法以及get()方法，用来返回值/服务/工厂。** ` 在run方法中不能注入"provider"类型的依赖 `
```

//define a module
var mainApp = angular.module("mainApp", []);
...
//create a service using provider which defines a method square to return square of a number.
mainApp.config(function($provide) {
   $provide.provider('MathService', function() {
      this.$get = function() {
         var factory = {};  
         factory.multiply = function(a, b) {
            return a * b; 
         }
         return factory;
      };
   });
});

```

**常量**
常量用于通过配置相位值考虑事实，值不能使用期间的配置阶段被传递。 可以用于run和config函数
```

mainApp.constant("configParam", "constant value");

```

angular注入器（**$injector类似于spring容器**）负责**创建、查找注入依赖**, **每个module都会有自己的注入器。**
![](/data/dokuwiki/web/pasted/20151116-223632.png)
```

// 在一个模块中提供连接信息  
angular.module('myModule', []).  
   
  // 告诉 injector 如何去构建一个 'greeter'  
  // 注意, greeter 自身是依赖于 '$window' 的  
  factory('greeter', function($window) {  
    // 这是一个 factory function,   
    // 职责是为创建  'greet' 服务.  
    return {  
      greet: function(text) {  
        $window.alert(text);  
      }  
    };  
  });  
   
// 新的 injector 从  module 创建.   
// (这通常由 angular bootstrap 自动创建)  
var injector = angular.injector(['myModule', 'ng']);  
   
// 从 injector 获取所有依赖  
var greeter = injector.get('greeter');  

```
要解决依赖关系硬编码的问题,也就意味着** injector 需要贯穿整个应用程序生命周期**。传递 injector 打破了 得墨忒耳定律(Law of Demeter, 最少知识原则)。为了弥补这一点,在下面的例子中,我们通过依赖声明的方式将查找依赖的职责交给了 injector:
HTML代码:
```

<!-- Given this HTML -->  
<div ng-controller="MyController">  
  <button ng-click="sayHello()">Hello</button>  
</div> 

``` 
Angular代码
```

// 这是 controller 定义  
function MyController($scope, greeter) {  
  $scope.sayHello = function() {  
    greeter.greet('Hello World');  
  };  
}  
   
// 由 'ng-controller' directive 在后台执行  
injector.instantiate(MyController);

```  
注意通过 ng-controller实例化此类,它可以 在 controller 不知道有 injector 的情况下满足MyController 所有的依赖。这是最好的结果。应用程序代码简单地要求所需的依赖项,无需和 injector 打交道。这个设置不违背 得墨忒耳定律。

###  控制器中使用DI  
Controllers 类负责应用程序的行为。**声明 controllers 的推荐的方法是` 使用数组表示法 `**:
```

someModule.controller('MyController', ['$scope', 'dep1', 'dep2', function($scope, dep1, dep2) {  
  ...  
  $scope.aMethod = function() {  
    ...  
  }  
  ...  
}]);

```  
**这避免了为 controllers 创建全局函数,并且在代码压缩时继续可用.**

ngMin工具可以减少依赖注入相关数组表示法的代码编写量

###  依赖注入实例 
```

<html>
<head>
   <title>AngularJS Dependency Injection</title>
</head>
<body>
   <h2>AngularJS Sample Application</h2>
   <div ng-app="mainApp" ng-controller="CalcController">
      <p>Enter a number: <input type="number" ng-model="number" />
      <button ng-click="square()">X<sup>2</sup></button>
      <p>Result: ![](/data/dokuwikiresult)</p>
   </div>
   <script src="http://ajax.googleapis.com/ajax/libs/angularjs/1.2.15/angular.min.js"></script>
   <script>
      var mainApp = angular.module("mainApp", []);
	  
      mainApp.config(function($provide) {
         $provide.provider('MathService', function() {
            this.$get = function() {
               var factory = {};  
               factory.multiply = function(a, b) {
                  return a * b; 
               }
               return factory;
            };
         });
      });

      mainApp.value("defaultInput", 5);

      mainApp.factory('MathService', function() {     
         var factory = {};  
         factory.multiply = function(a, b) {
            return a * b; 
         }
         return factory;
      }); 

      mainApp.service('CalcService', function(MathService){
            this.square = function(a) { 
            return MathService.multiply(a,a); 
         }
      });

      mainApp.controller('CalcController', function($scope, CalcService, defaultInput) {
            $scope.number = defaultInput;
            $scope.result = CalcService.square($scope.number);

            $scope.square = function() {
            $scope.result = CalcService.square($scope.number);
         }
      });
   </script>
</body>
</html>

```

##  服务 
控制器只会在需要时被实例化，并且在不再需要就会被销毁。这意味着每次切换路由或重新加载视图时，当前的控制器会被AngularJS清除掉。
服务提供了一种能在应用的整个生命周期内保持数据的方法，它能够在控制器之间进行通信。并且能保证数据的一致性。不同于控制器，**服务只会在应用启动时被初始化一次，它是` 单例 `的**。
Angular提供的常用标准服务组件有以下：
  * $http：用于处理 XMLHttpRequest 
  * $location：提供当前URL的信息
  * $q： 异步请求使用，promise/deferred模块
  * $routeProvider：配置路由
  * $log：日志服务
###  注册服务 
AngularJS提供例如许多内在的服务，如：$http, $route, $window, $location等。每个服务负责例如一个特定的任务，$http是用来创建AJAX调用，以获得服务器的数据。 $route用来定义路由信息等。内置的服务总是前缀$符号。
系统内置的服务以$开头，我们也可以自己定义一个服务。定义服务的方式有如下几种：
  * 使用系统内置的$provide服务
  * 使用Module的factory方法
  * 使用Module的service方法

**使用工厂方法 推荐，这是最常见最灵活的模式**
使用工厂方法，我们先定义一个工厂，然后分配方法给它。
```

      var mainApp = angular.module("mainApp", []);
      mainApp.factory('MathService', function() {     
         var factory = {};  
         factory.multiply = function(a, b) {
            return a * b 
         }
         return factory;
      });

``` 

使用服务方法
使用服务的方法，我们定义了一个服务，然后分配方法。还注入已经可用的服务。
```

mainApp.service('CalcService', function(MathService){
    this.square = function(a) { 
		return MathService.multiply(a,a); 
	}
});

```
###  provider方法及provider供应商的概念 
所有服务工厂底层都是由$provider服务创建的，$provider服务负责在运行时初始化这些提供者。
提供者是一个具有` $get() `方法的对象，**$injector通过调用$.get()方法创建服务实例。**
**所有创建服务的方法都构建在provider方法之上，provider()方法负责在` $providerCache `中注册服务**
```

      var mainApp = angular.module("mainApp", []);
      mainApp.factory('MathService', function() {     
         var factory = {};  
         factory.multiply = function(a, b) {
            return a * b 
         }
         return factory;
      }).provider('MyService', function() {     
         var factory = {};  
         factory.multiply = function(a, b) {
            return a * b 
         }
         factory.setName=function(name){
         	factory.name=name;
         }
         $get:function( return factory;);
      });

``` 
**是否可以一直使用.factory()方法来代替.provider()呢？**
答案是否定的，因为**Providers是唯一一种你可以传进 .config() 函数的 service**。**当你想要在 service 对象启用之前，先进行模块范围的配置，那就应该用 provider()来定义服务。**
provider方法在文本MyService后面添加Provider后缀生成了一个新的提供者MyServiceProvider，通过这个MyServiceProvider可以在config()函数中对MyService服务进行配置。
**在这里要注意：MyService为服务的名字和服务实例的名字，MyService` Provider `为服务提供者。用provider()方法创建服务，必须有一个` $get() `函数。否则会导致错误**

```

angular.module('app',[]).config(function(MyServiceProvider){
	MyServiceProvider.setName('haha');

});

```
###  服务示例1 
在本例中，在输入框内容的改变间隔如果没有超过350毫秒，$timeout service不会发送任何网络请求。换句话说，如果在键盘输入时超过350毫秒，我们就假定用户已经完成输入，我们就可以开始向GitHub发送请求
```

<div ng-controller="ServiceController">
  <label for="username">Type in a GitHub username</label>
  <input type="text" ng-model="username" placeholder="Enter a GitHub username, like auser" />
  <pre ng-show="username">![](/data/dokuwiki events )</pre>
</div>

```
```

angular.module('myApp.services', [])
  .factory('githubService', ['$http', function($http) {
 
    var doRequest = function(username, path) {
      return $http({
        method: 'JSONP',
        url: 'https://api.github.com/users/' + username + '/' + path + '?callback=JSON_CALLBACK'
      });
    }
    return {
      events: function(username) { return doRequest(username, 'events'); },
    };
  }]);

```
```

app.controller('ServiceController', ['$scope', '$timeout', 'githubService',
  function($scope, $timeout, githubService) {
    
// 观察username属性的变化，如果有变化就会运行该函数。
    var timeout;
    $scope.$watch('username', function(newVal,oldValue) {
      if (newVal) {
        if (timeout) $timeout.cancel(timeout);//取消以前的请求
        timeout = $timeout(function() { 
          githubService.events(newVal)
          .success(function(data, status) {
            $scope.events = data.data;
          });
        }, 350); //在输入框内容的改变间隔如果没有超过350毫秒不会执行具体逻辑
      }
    });
  }]);

```

###  decorator装饰器 
装饰器` $provide.decorator() `可以在服务实例创建时(包括AngularJS核心服务)对其进行拦截，终端，甚至替换。
例如像给定义的服务加入日志功能，可以借助装饰器方便地实现这个功能。而不需要对原始服务进行修改。类似于AOP或拦截器
decorator()函数接受两个参数：
  * name字符串：将要拦截的服务名称
  * 拦截函数
在一面的代码中，我们将对Angular内置service $log进行装饰，让它在每次被调用时除了自身接受的参数之外，还能输出一个当前的时间戳。
具体实现如下：
```

angular.module('myapp',[]).config(function($provide){                  
	$provide.decorator('MyService',function($delegate,$log){                      
		var events=function(path){
			var startAt=new Date();
			var events=$delegate.events(path);//$delegate代表的service的引用,这句话就是调用MyService上的events()方法。
			//事件是一个promise 
			events.finally(function(){
				$log.info("aaaa"+(new Date()-startAt)+"ms");
			});
			return events;
		}; 
          	$delegate.events=events;
		return $delegate;
	});               
}); 

```   
上面的代码很简单，没有什么难点，关键的地方是在于**decorator方法的回调函数的参数$delegate**，它是decorator**第一个参数代表的service的引用**。

###  装饰器实例 
Use case 使用场景：
我有一个module A依赖于另外一个module B。 module B有个service Mail， 这个服务提供 两个方法 setReceiver 和 setBody 分别用来指定邮件的收件人和邮件的内容。 
但是在 module A 使用Mail服务的时候， 我希望还可以指定抄送的人。 这个时候我就可以在已有 的service上扩展下（装饰下）加个addCC的方法。
Module A
```

var Mail = function() {
    this.receiver = '';
    this.body = '';
    this.cc = [];
};

Mail.prototype.setReceiver = function(receiver) {
    this.receiver = receiver;
};

Mail.prototype.setBody = function(body) {
    this.body = body;
};

angular.module('A', []).service('Mail', Mail);

```
Module B
```

angular.module('B', ['A']).config(function($provide) {
    $provide.decorator('Mail', function($delegate) {
        $delegate.addCC = function(cc) {
            this.cc.push(cc);
        };
        return $delegate;
    });
})
.controller('TestCtrl', function($scope, Mail) {
    Mail.addCC('jack');
    console.log(Mail);
});

```
参考：http://blog.csdn.net/renfufei/article/details/19038123
http://blog.jobbole.com/49745/
http://boyitech.iteye.com/blog/2163138?utm_source=tuicool&utm_medium=referral
http://www.mamicode.com/info-detail-247448.html
http://www.yiibai.com/angularjs/angularjs_dependency_injection.html
http://www.cnblogs.com/Leo_wl/p/3852347.html
http://www.tuicool.com/articles/ZNbeEj
http://www.ithao123.cn/content-8367758.html
http://www.angularjs.cn/A086