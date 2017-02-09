title: angularjs模块 

#  AngularJS模块 
AngularJS有几大特性，比如：
　　1 MVC
　　2 模块化
　　3 指令系统
　　4 双向数据绑定
那么本篇就来看看AngularJS的模块化。
首先先说一下为什么要实现模块化：
　　1 增加了模块的可重用性
　　2 通过定义模块，实现加载顺序的自定义
　　3 在单元测试中，不必加载所有的内容
　　之前做的几个例子，控制器的代码直接写在script标签里面，这样声明的函数都是全局的，显然不是一个最好的选择。
　　下面看看如何进行模块化：
```

        <script type="text/javascript">
            var myAppModule = angular.module('myApp',[]);
            
            myAppModule.filter('test',function(){
                return function(name){
                    return 'hello, '+name+'!';
                };
            });

            myAppModule.controller('myAppCtrl',['$scope',function($scope){
                $scope.name='xingoo';
            }]);
        </script>

```
首先，**通过` 全局变量angular `创建模块myAppModule**
angular.module('myApp',[]);
　　第一个参数是绑定的应用app名称，这个app标识了页面中angular的入口点，类似main函数的作用。
　　第二个参数[]里面标识了依赖的模块。
　　下面看看如何使用模块吧！
```

<!doctype html>
<html ng-app="myApp">
    <head>
         <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
         <script src="http://apps.bdimg.com/libs/angular.js/1.2.16/angular.min.js"></script>
    </head>
    <body>
        <div ng-controller="myAppCtrl">
            ![](/data/dokuwikiname | test )
        </div>
        <script type="text/javascript">
            var myAppModule = angular.module('myApp',[]);
            
            myAppModule.filter('test',function(){
                return function(name){
                    return 'hello, '+name+'!';
                };
            });

            myAppModule.controller('myAppCtrl',['$scope',function($scope){
                $scope.name='xingoo';
            }]);
          /**
          *也可以自己手动方式启动而不采用ng-app指令。
          angular.element(document).ready(function () {
            angular.bootstrap(document, ['myApp']);//第一个参数只带angularJS应用的覆盖范围，第二个参数就是指定入口模块。
            ace.init();
        });
        */
        </script>
    </body>
</html>

```
直接绑定myApp到ng-app上，就可以了。
在script中，我们通过模块创建了一个filter和一个控制器。
filter的作用是 添加字符串修饰。
控制器的作用则是初始化变量。
程序的运行结果如下：
![](/data/dokuwiki/web/pasted/20151117-235328.png)
参考：http://www.cnblogs.com/xing901022/p/4287724.html