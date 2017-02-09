title: angularjs表达式与过滤器 

#  AngularJS表达式与过滤器 
##  表达式 
表达式在AngularJS应用中被广泛使用。
&lt;nowiki&gt;用![](/data/dokuwiki}}将一个变量绑定到$scope上的写法本质上就是一个表达式{{expression).当用$watch进行监听时，AngularJS会对表达式或函数进行运算。&lt;/nowiki&gt;
表达式显著的特性：
  * 所有的表达式都在其所属的作用域上下文内执行，并有访问本地$scope的权限。
  * 如果发生类型错误并不会抛出异常
  * 不允许任何流程控制。
  * 可以接受过滤器和过滤器链

###  解析表达式： 
尽管AngularJS在运行$digest循环的过程中自动解析表达式，但有时手动解析表达式也是挺有用的。
通过$parse这个内部服务来进行表达式的运算，这个服务能够访问当前所处的作用域。将$parse4服务注入到控制器中。然后调用它就可以实现手动解析表达式
```

<!doctype html>
<html ng-app="myApp">
<head>
  <title>Parse Expression Example</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0-rc.2/angular.js"></script>
</head>
<body>

  <div ng-controller="MyController">
    <input ng-model="expr"
            type="text"
            placeholder="Enter an expression" />
    <div>![](/data/dokuwiki parsedExpr )</div>
  </div>
  </body>
</html>

```
```

    angular.module('myApp', [])
    .controller('MyController',
    ['$scope', '$parse', function($scope, $parse) {

      $scope.person = {
        name: "Ari Lerner"
      };

      $scope.$watch('expr', function(newVal, oldVal, scope) {
        if (newVal !== oldVal) {
          // Let's set up our parseFun with the expression
          var parseFun = $parse(newVal);
          // Get the value of the parsed expression, set it on the scope for output
          scope.parsedExpr = parseFun(scope);
        }
      });
    }]);

```
###  插值字符串 
在AngularJS中我们还可以手动运行模板编译的能力。例如，插值允许基于作用域上的某些条件实时更新文本字符串。
插值操作，需要你在对象中注入$interpolate服务。
设想这样一个情节：我们希望在电子邮件的正文中进行实时编辑，当文本发生变化时进行字符插值操作并将结果展示出来。
```

<!DOCTYPE html>
<html>
<head>
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.5/angular.min.js"></script>
<meta charset=utf-8 />
<title>JS Bin</title>
</head>
<body ng-app="myApp">
  <div ng-controller="MyController">
    <input ng-model="to" 
          type="email" 
          placeholder="Recipient" />
    <textarea ng-model="emailBody"></textarea>
    <pre>![](/data/dokuwiki previewText )</pre>
  </div>
</body>
</html>

```
```

angular.module('myApp', [])
  .controller('MyController', 
    function($scope, $interpolate) {
      $scope.to = 'ari@fullstack.io';
      $scope.emailBody = 'Hello ![](/data/dokuwiki to ),\n\nMy name is Ari too!';
      // Set up a watch
      $scope.$watch('emailBody', function(body) {
        if (body) {
          var template = $interpolate(body);
          $scope.previewText = 
            template({to: $scope.to});
        }
      });
});

```
$interpolate服务可以接受三个参数：
  * text字符串：一个包含字符插值标记的字符串
  * mustHaveExpression(布尔型):如果将这个参数设为true，当传入的字符串中不含有表达式时会返回null
  * trustedContext(字符串):AngularJS会对已经进行过字符插值操作的字符串通过$sec.getTrusted()方法进行严格的上下文转义。
$interpolate服务返回一个函数，用来在特定的上下文中运算表达式。
` $watch函数会监视$scope上的某个属性。只要属性发生变化就会调用对应的自定义函数。 `
&lt;nowiki&gt;现在在![](/data/dokuwikipreviewText)内部的文本中可以将![](/data/dokuwikito)当作一个变量来使用，并对文本的变化进行实时更新&lt;/nowiki&gt;
&lt;nowiki&gt;提示：如果需要在文本中使用不同于{{}}的符号来标识表达式的开始和结束，可以在$interpolateProvider中进行配置&lt;/nowiki&gt;。如下：
startSymbol()可以修改标识开始的符号; endSymbol()可以修改标识结束的符号。
```

<!doctype html>
<html ng-app="myApp">
<head>
  <title>Interpolate String Template Example</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.5/angular.js"></script>
</head>
<body>

  <div id="emailEditor" ng-controller="MyController">
    <input ng-model="to"
           type="email"
           placeholder="Recipient" />
    <textarea ng-model="emailBody"></textarea>

    <div id="emailPreview">
      <pre>__ previewText __</pre>
    </div>
  </div>


```
```

    angular.module('myApp', ['emailParser'])
      .controller('MyController',
        ['$scope', 'EmailParser',
          function($scope, EmailParser) {
            $scope.to = 'ari@fullstack.io';
            $scope.emailBody = 'Hello __to__';
            // Set up a watch
            $scope.$watch('emailBody', function(body) {
              if (body) {
                $scope.previewText =
                  EmailParser.parse(body, {
                    to: $scope.to
                  });
              }
            });
    }]);
    //config配置服务$interpolateProvider
    angular.module('emailParser', [])
      .config(['$interpolateProvider',
        function($interpolateProvider) {
          $interpolateProvider.startSymbol('__');
          $interpolateProvider.endSymbol('__');
    }])   //factory创建服务EmailParser
    .factory('EmailParser', ['$interpolate',
      function($interpolate) {
        // a service to handle parsing
        return {
          parse: function(text, context) {
            var template = $interpolate(text);
            return template(context);
          }
        };
    }]);

```
##  过滤器 
过滤器用来格式化需要展示给用户的数据。AngularJS有很多实用的内置过滤器。
&lt;nowiki&gt;HTML中的模板绑定符号{{}}内通过|符号来调用过滤器，可以调用多个。通过:符号来传递过滤器参数，可以使用多个:来传递多个参数。&lt;/nowiki&gt;
例如 &lt;nowiki&gt;![](/data/dokuwikiname | uppercase)    ![](/data/dokuwiki12.3244 | number/2 )&lt;/nowiki&gt;
在JS代码中可以通过调用$filter来调用过滤器。例如：
```

app.controller('DemoController',['$scope','$filter',function($scope,$filter){
	$scope.name=$filter('lowercase')('Abc');
}])

```

###  内置过滤器介绍 
1、currency货币格式
2、date时间日期格式  &lt;nowiki&gt;![](/data/dokuwikitoday|date/'yyyyMMdd HH/mm/ss') &lt;/nowiki&gt;
3、filter过滤子集:从数组中过滤一个子集返回一个新数组。可以传递三种类型参数：
  * 字符串：返回包含这个字符串的元素
  * 对象:属性比较。匹配则包含
  * 函数:对每个元素都执行这个函数，返回非假值的元素会出现在新的数组中
 &lt;nowiki&gt;![](/data/dokuwiki['aee','avb','ace'] | filter/'e') &lt;/nowiki&gt;
&lt;nowiki&gt;![](/data/dokuwiki[{'name'/'xby','phone'/'123'},{'name'/'xaa','phone'/'234'}] | filter/{'name'/'xby')} &lt;/nowiki&gt;
&lt;nowiki&gt;![](/data/dokuwiki[{'name'/'Xby','phone'/'123'},{'name'/'xaa','phone'/'234'}] | filter/isCapitalized)&lt;/nowiki&gt;
```

$scope.isCapitalized=function(str){
	return str[0]==str[0].toUpperCase();
}

```

4、json过滤器：将一个JSON或JS对象转化成字符串
5、limitTo过滤长度
6:lowercase/uppercase
7:number
8.orderBy排序。可以接受两个参数，第一个是必须的，第二个是可选的布尔值，控制是否排序反转
第一个参数类型有三种：
  * 函数:该函数被当作排序对象的getter方法
  * 字符串:指定排序放向
  * 数组:略。
 &lt;nowiki&gt;
![](/data/dokuwiki[{'name'/'Xby','phone'/'123'},{'name'/'xaa','phone'/'234'}] | orderBy/'name'/true)
 &lt;/nowiki&gt;
###  自定义过滤器 
格式大致如：
```

app.filter('filter(过滤器)名称',function(){   
           return function(需要过滤的对象,过滤器参数1,过滤器参数2,...){       
                      //...执行业务逻辑代码        
                      return 处理后的对象;    
            }
});   

``` 
```

angular.module('myApp.filters',[]).filter('capitalize',function(){
	return function(input){//input是我们传入的字符串
		if(input){
			return input[0].toUpperCase()+input.slice(1);
		}
	}
})

```
 &lt;nowiki&gt;
![](/data/dokuwiki'ahdsahdsa JKAkfjsdfsd'| lowercase|capitalize)
 &lt;/nowiki&gt;

示例2：带参数版本
```

myAppModule.filter("reverse",function(){
                return function(input,uppercase){
                    var out = "";
                    for(var i=0 ; i<input.length; i++){
                        out = input.charAt(i)+out;
                    }
                    if(uppercase){
                        out = out.toUpperCase();
                    }
                    return out;
                }
            });

```
内部返回的方法包含了两个参数，一个是输入的值，就是我们过滤器接受的值。
如果想要实现下面的过滤器：
name | reverse
则input就是其中name代表的值。
后面的参数是可选的，我们这里接受uppercase这个bool值，判断是否要进行大小写转换。
内部实现的代码，就没必要解释了。最后返回过滤后的字符串即可。
参考：http://www.cnblogs.com/xing901022/p/4290102.html
###  表单验证 
AngularJS的表单验证相比jQuery.validate的表单验证而言，稍显繁琐。故而不推荐使用。
