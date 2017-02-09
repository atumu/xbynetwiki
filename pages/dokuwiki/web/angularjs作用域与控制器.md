title: angularjs作用域与控制器 

#  AngularJS作用域与控制器 
以AngularJS 1.2版本为例
##  作用域$scope 

作用域$scope 是和应用的数据模型相关联的，同时也是表达式执行的上下文。作用域是视图和控制器之间的胶水。
作用域是应用状态的基础，$scope被设计成和DOM类似的层级嵌套结构。可以进行嵌套和原型继承。也就是说我们可以引用父级$scope中的属性.顶级父级ng-app的为$rootScope 
我们在` 作用域的执行上下文环境 `中定义和执行表达式。

AngularJS启动并生成视图时，会将根ng-app元素同$rootScope进行绑定。$rootScope是所有$scope对象的最上层。
$scope对象就是一个普通的js对象。我们可以在其上随意修改或添加属性。
$scope对象在AngularJS中充当数据模型。而它本身并不复杂处理数据。它的所有属性都可以被视图访问到
```

<div ng-app="myApp">
	<div ng-controller="MyController">
		<h1>Hello ![](/data/dokuwikiname)</h1>
	</div>
</div>

angular.module("myApp",[]).controller('MyController',function($scope){$scope.name="hja"});
或者
function MyController($scope){
	$scope.name="hja"
}

```

$scope的生命周期：
创建：当创建控制器或指令时，AngularJS会用注入器$injector创建一个继承父级的新的作用域，并传递过去
链接：将$scope附加到视图。$scope对象注册到当Angular应用上下文发生变化时需要执行的函数$watch
更新：事件循环脏检查+回调
销毁

##  控制器 
控制器在AngularJS中的作用是增强视图.它本身就是一个函数。
控制器初始化:
```

function MyController($scope){
	$scope.msg="haha";
}

```
但是上面那个写法会污染全局命名空间，更合理的方式是创建一个模块，然后在模块中创建控制器，如下：
```

var app=angular.module('app',[]);
app.controller("MyController",function($scope){
	$scope.msg="haha";
 	$scope.add=function(count){...}
});

```

控制器嵌套与作用域嵌套：原型继承


参考：http://www.ibm.com/developerworks/cn/opensource/os-cn-AngularJS/index.html