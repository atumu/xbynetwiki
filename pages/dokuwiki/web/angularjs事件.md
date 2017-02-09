title: angularjs事件 

#  AngularJS事件 
使用angularjs，发现**controller间的值传递**: 
  * 可以通过父子控制器共享scope里的变量和方法，
  * 或者在服务里共享，
  * 或者通过事件传递。

事件处理是基于作用域的。由于作用域本身是分层次的。所以我们可以通过在这个作用域链上传递事件的方式在作用域之间通信传递事件。
由于作用域是有层次的，基于这一点，**事件传播的方式：1、向上冒泡传递$scope.$emit,这种方式可以被中断，2、向下广播$scope.$broadcast,不能被中断。**
**发送事件通过 $scope.$broadcast(eventName, ...args);  $scope.$emit(eventName, ...args');来触发事件。**
**注意：Angular的事件与DOM事件不是同一个。**
**参数解析：**
  * Angular应用可以通过$scope.$on(eventName，function(event,...args){})来进行监听事件。
  * eventName:事件名称，
  * event：事件对象
```

 event：事件对象拥有如下属性：
	targetScope - {Scope}: 事件发送或广播者作用域。
	currentScope - {Scope}: 事件监听者作用域。
	name - {string}: 事件名
	stopPropagation - {function=}: 调用stopPropagation()函数将会取消$emit事件向上继续冒泡。
	preventDefault - {function}: 调用preventDefault()函数设置defaultPrevented标志为true.尽管博能停止事件的广播，我们可以告诉子作用域无需处理这个事件。也就是提示可以忽略。
	defaultPrevented - {boolean}: true if preventDefault was called.

```
  * ...args：需要传输的数据参数列表，可以有多个参数。

**同时$scope.$on()函数返回了一个反注册函数，我们可以调用它来取消监听器。**

##  核心系统的$emit事件 
```

 $scope.$on('$viewContentLoaded', function(event,data) {  
        console.log(data);      
    });

```
  * $includeContentLoaded: ngInclude内容加载完成，从ngInclude指令触发。
  * $includeContentRequested
  * $viewContentLoaded:当ngView内容被加载时，从当前的ngView作用域上触发
##  核心系统的$broadcast事件 
  * $locationChangeStart：从$rootScope广播出来
  * $locationChangeSuccess
  * $routeChangeStart从$rootScope广播出来
  * $routeChangeSuccess
  * $routeChangeError
  * $routeUpdate
  * $destroy:在作用域被销毁之前，该事件会在作用域上广播。这个顺序给子作用域一个清除父引用的机会。
例如：我们在控制器中有一个正在运行的$timeout,我们不希望在包含他的控制器已经不存在的情况下，它还继续触发。
```

angular.module('myApp',[]).controller("MainCtrl",function($scope,$timeout){
	var timer;
	var updateTime=function(){
		$scope.date=new Date();
		timer=$timeout(updateTime,1000);
	}
	timer=$timeout(updateTIme,1000);
	$scope.$on("$destroy",function(event){
		if(timer){$timeout.cancel(timer)};
	});
})

```
##  $broadcast $emit $on的处理 
在一个controller里面通过事件触发一个方法，在方法里面通过$broadcast或$emit来定义一个变量，在父，子controller里面通过$on来获取。
二，实例说明angularjs $broadcast $emit $on的用法
1，html代码
```

<div ng-controller="ParentCtrl">                  //父级  
    <div ng-controller="SelfCtrl">                //自己  
        <a ng-click="click()">click me</a>  
        <div ng-controller="ChildCtrl"></div>     //子级  
    </div>  
    <div ng-controller="BroCtrl"></div>           //平级  
</div> 

``` 
2，js代码
```

phonecatControllers.controller('SelfCtrl', function($scope) {  
    $scope.click = function () {  
        $scope.$broadcast('to-child', 'child');  
        $scope.$emit('to-parent', 'parent');  
    }  
});  
  
phonecatControllers.controller('ParentCtrl', function($scope) {  
    $scope.$on('to-parent', function(d,data) {  
        console.log(data);         //父级能得到值  
    });  
    $scope.$on('to-child', function(d,data) {  
        console.log(data);         //子级得不到值  
    });  
});  
  
phonecatControllers.controller('ChildCtrl', function($scope){  
    $scope.$on('to-child', function(d,data) {  
        console.log(data);         //子级能得到值  
    });  
    $scope.$on('to-parent', function(d,data) {  
        console.log(data);         //父级得不到值  
    });  
});  
  
phonecatControllers.controller('BroCtrl', function($scope){  
    $scope.$on('to-parent', function(d,data) {  
        console.log(data);        //平级得不到值  
    });  
    $scope.$on('to-child', function(d,data) {  
        console.log(data);        //平级得不到值  
    });  
}); 

``` 
3，点击Click me的输出结果
child  
parent

总结：
**` $emit为向上冒泡传递事件，可以中断冒泡，而$broadcast向下广播事件，不能被中断广播。 `**
**用$broadcast赋的值，只能子级得到值；$emit赋的值，只能父级得到；而平级的什么都不能得到。**

参考：http://blog.51yip.com/jsjquery/1602.html