title: angularjs指令 

#  AngularJS指令 
指令的本质就是AngularJS扩展具有自定义功能的HTML元素的途径。
1、HTML引导
```

<html ng-app="myApp"></html>

```
ng-app指令标记出AngularJS应用的根节点。但是一个HTML页面中也可以有多个ng-app出现即多个AngularJS应用。不过这种情形下需要自己手动启动应用。
2、第一个自定义指令
AngularJS编译HTML与DOM时就会调用指令。调用指令意味着执行指令背后与之相关的JS代码。
` AngularJS在构造自定义指令的同时也会通过原型继承机制创建新的子作用域。 `
```

angular.module('myApp',[])
	.directive('myDire',function(){
		retutn{
			restrict:'E', //指令适用范围类型
			replace : true //是否只保留模板内容，而移除标签体。
			template:'<a href="http://wiki.xby1993.net"'>click me </a>' //模板内容
		}	
	});

```
使用
```

<myDire></myDire>

```
**其中restrict指定指令的使用范围： 元素(E),属性(A),类(C)或注释(M).**
也可以同时使用多个，如restrict: 'EAC',不过，一般主要使用restrict : 'A'

当前作用域与隔离作用域的概念

##  内置指令 
AngularJS提供了一系列内置指令，其中一些指令重载了原生的HTML元素如&lt;nowiki&gt;&lt;form&gt;&lt;/nowiki&gt;.有些指令有对应的HTML标签，有些则没有。
###  基础的ng属性指令 
` ng-src:告诉浏览器在ng-src对应的表达式生效之前不要加载图像 `
**ng-href：动态创建URL时使用**
ng-disabled
ng-checked
ng-readonly
ng-selected
ng-class
ng-style

##  布尔属性：(动态绑定) 
1、ng-disabled：
支持的表单输入字段:&lt;nowiki&gt;&lt;input&gt;家族(text,checkbox,radio,number,url,email,submit), &lt;textarea&gt; ,&lt;select&gt; , &lt;button&gt;&lt;/nowiki&gt;
```

<!doctype html>
<html ng-app="myApp">
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0-rc.3/angular.js"></script>
</head>
<body>

  <h1>Demo 1:</h1>
  <input type="text" ng-model="someProperty" placeholder="Type to Enable">
  <button ng-model="button" ng-disabled="!someProperty">A Button</button>

  <h1>Demo 2:</h1>
  <textarea ng-disabled="isDisabled">Wait 1 second</textarea>

  <script>
    // JS for demo 2:
    angular.module('myApp', [])
    .run(function($rootScope, $timeout) {
      $rootScope.isDisabled = true;
      $timeout(function() {
        $rootScope.isDisabled = false;
      }, 1000);
    });
  </script>

</body>
</html>

```
###  在指令中使用子作用域 
**指令会以父级作用域为原型生成子作用域。
ng-app和ng-controller是特殊的指令，因为他们会修改嵌套在他们内部指令的作用域。
ng-app为AngularJS应用创建$rootScope。ng-Controller则会以$rootScope或另外一个ng-controller的作用域为原型创建新的子作用域。**

###  作用域与数据绑定的最佳实践 
` 通过在$scope添加模型对象，然后通过模型对象来访问值属性，而不要直接在$scope上添加访问值属性。这点非常重要。 `否则可能存在当ng-controller嵌套时，子属性覆盖父属性的情形。进而导致数据绑定出现不一致的情形。
原因分析：因为JS中对于值属性（字符串，布尔，数据类型）使用的是**值复制**，而对于对象则使用**引用复制**。通过添加模型对象，进而在模型对象内部访问属性。由于模型对象是引用复制传递，所以就不会出现上述情形问题。

```

<!doctype html>
<html ng-app="myApp">
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0-rc.3/angular.js"></script>
</head>
<body>

  <div ng-controller="SomeCtrl">
    ![](/data/dokuwiki someModel.someValue )
    <button ng-click="someAction()">Communicate to child</button>
    <div ng-controller="ChildCtrl">
      ![](/data/dokuwiki someModel.someValue )
      <button ng-click="childAction()">Communicate to parent</button>
    </div>
  </div>

  <script>
    angular.module('myApp', [])
    .controller('SomeCtrl', function($scope) {
      // best practice, always use a model
      $scope.someModel = {
        someValue: 'hello computer'
      }
      $scope.someAction = function() {
        $scope.someModel.someValue = 'hello human, from parent';
      };
    })
    .controller('ChildCtrl', function($scope) {
      $scope.childAction = function() {
        $scope.someModel.someValue = 'hello human, from child';
      };
    });
  </script>

</body>
</html>

```

##  ng-include 
**使用ng-include可以加载、编译并包含外部HTML片段到当前应用中。**
```

<div ng-include="/myTemplate.html"
	ng-controller="MyCtrl">
</div>

```
##  ng-switch 
略
##  ng-view 
**ng-view指令用来设置将被路由管理的视图位置。**即相当于路由的占位符。
##  ng-if 
**如果赋值给ng-if的表达式的值是false,那对应的元素将会从DOM中移除。否则对应元素的一个克隆将被重新插入DOM中**。` 重新插入的状态不是上次被移除的状态，而是原始状态。这个需要特别注意 `。
##  ng-repeat 
ng-repeat用来遍历一个集合或为集合中的每个元素生成一个模板实例。集合中的每个元素都会被赋予自己的模板和作用域。同时会暴露一些属性：
$index,$first,$last,$middle,$even,$odd
```

<!doctype html>
<html ng-app="myApp">
<head>
  <link rel="stylesheet" href="//cdn.jsdelivr.net/foundation/4.3.2/css/foundation.min.css">
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0-rc.2/angular.js"></script>
</head>
<body>
  
<ul ng-controller="PeopleController">
  <li ng-repeat="person in people" ng-class="{even: !$even, odd: !$odd}">
    ![](/data/dokuwikiperson.name) lives in ![](/data/dokuwikiperson.city)
  </li>
</ul>
  
</body>
</html>

```
```

angular.module('myApp', [])
.controller('PeopleController', function($scope) {
  $scope.people = [
    {name: "Ari", city: "San Francisco"},
    {name: "Erik", city: "Seattle"}
  ];
});

```
##  ng-model 
**ng-model指令用来将input,select,textarea或自定义表单控件同包含他们作用域中的属性进行绑定。**
##  ng-show/ng-hide 
根据所给表达式的值来显示或隐藏HTML元素。
##  ng-change 
和ng-model结合使用，会在表单输入发生变化时计算给定表达式的值
```

<!doctype html>
<html ng-app="myApp">
<head>
  <link rel="stylesheet" href="//cdn.jsdelivr.net/foundation/4.3.2/css/foundation.min.css">
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0-rc.2/angular.js"></script>
</head>
<body>
  
<!-- type any number into the text field -->

<div ng-controller="EquationController">
  <input type="text" ng-model="equation.x" ng-change="change()" />
  
  ![](/data/dokuwiki equation.output )
</div>
  
</body>
</html>

```
```

angular.module('myApp', [])
.controller('EquationController', function($scope) {
  $scope.equation = {};
  $scope.change = function() {
    $scope.equation.output = Number($scope.equation.x) + 2;
  };
});

```
##  ng-form，ng-submit,ng-click 
ng-form用于嵌套表单
在form标签` 没有指定action的情况下 `，**Angular不会将表单提交到服务器**，除非它指定了action属性。要指定提交表单调用哪个JS方法，使用下面两个指令中的一个：
  * ng-submit:在form标签上使用。
  * ng-click:在submit类型的按钮或submit类型的输入框上使用.
` 注意不能混用。 `
```

<form name="my-form" ng-submit="submitForm()">...</form>

```
##  ng-select 
结合ng-model,ng-options指令一起使用。
两种形式数据源：数组与对象
```

<!doctype html>
<html ng-app="myApp">
<head>
  <link rel="stylesheet" href="//cdn.jsdelivr.net/foundation/4.3.2/css/foundation.min.css">
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.0-rc.2/angular.js"></script>
</head>
<body>

<div ng-controller="CityController">
  <select ng-model="city" ng-options="city.name for city in cities">
    <option value="">Choose City</option>
  </select>
  Best City: ![](/data/dokuwiki city.name )
</div>
  
</body>
</html>

```
```

angular.module('myApp', [])
.controller('CityController', function($scope) {
    $scope.cities = [
      {name: 'Seattle'},
      {name: 'San Francisco'},
      {name: 'Chicago'},
      {name: 'New York'},
      {name: 'Boston'}
    ];
});

```

##  自定义指令 
下面的代码是一个简单的 Hello World 指令。
```

var app = angular.module('myapp', []);
app.directive('helloWorld', function() {
  return {
      restrict: 'AE',
      replace: 'true',
      template: '<h3>Hello World!!</h3>'
  };
});

```
在上面的代码中，app.directive()方法在模块中注册了一个新的指令。这个方法的第一个参数是这个指令的名字。**第二个参数是一个返回指令定义对象的函数**。如果你的指令依赖于其他的对象或者服务，比如 $rootScope, $http, 或者$compile，他们可以在这个时间被注入。这个指令在HTML中以一个元素使用，如下：
```

<hello-world/>
//OR
<hello:world/>

```
或者，以一个属性的方式使用：
```

<div hello-world></div>
//OR
<div hello:world/>

```
如果你想要符合HTML5的规范，你可以在元素前面添加 x- 或者 data-的前缀。所以下面的标记也会匹配 helloWorld 指令：
```

<div data-hello-world></div>
//OR
<div x-hello-world></div>

```
注意： **在匹配指令的时候**，Angular会在元素或者属性的名字中剔除 x- 或者 data- 前缀。 然后将 – 或者 : **连接的字符串转换成驼峰(camelCase)表现形式，然后再与注册过的指令进行匹配。**
我们在指令定义过程中**使用了三个属性来配置指令**。我们来一一介绍他们的作用。
  * restrict – 这个属性用来指定指令在HTML中如何使用。在上面的例子中，我们使用了 ‘AE’。所以这个指令可以被当作新的HTML元素或者属性来使用。如果要允许指令被当作class来使用，我们将 restrict 设置成 ‘AEC’。
  * template – 这个属性规定了指令被Angular编译和链接（link）后生成的HTML标记。这个属性值不一定要是简单的字符串。template 可以非常复杂，而且经常包含其他的指令，以及表达式(![](/data/dokuwiki ))等。更多的情况下你可能会见到 **templateUrl**， 而不是 template。所以，**理想情况下，你应该将模板放到一个特定的HTML文件中，然后将 templateUrl 属性指向它。**
  * replace – 这个属性指明生成的HTML内容是否会替换掉定义此指令的HTML元素。在我们的例子中，我们用 <hello-world></hello-world>的方式使用我们的指令，并且将 replace 设置成 true。所以，在指令被编译之后，生成的模板内容替换掉了 <hello-world></hello-world>。最终的输出是 <h3>Hello World!!</h3>。如果你将 replace 设置成 false，也就是默认值，那么生成的模板会被插入到定义指令的元素中。

###  Link函数和Scope 
指令生成出的模板其实没有太多意义，除非它在特定的scope下编译。` 默认情况下，指令并不会创建新的子scope。 `**更多的，它使用父scope。**也就是说，如果指令存在于一个controller下，它就会使用这个controller的scope。 **如何运用scope，我们要用到一个叫做 link 的函数。它由指令定义对象中的link属性配置。**
让我们来改变一下我们的 helloWorld 指令，当用户在一个输入框中输入一种颜色的名称时，Hello World 文字的背景色自动发生变化。同时，当用户在 Hello World 文字上点击时，背景色变回白色。 相应的HTML标记如下：
```

<body ng-controller="MainCtrl">
  <input type="text" ng-model="color" placeholder="Enter a color" />
  <hello-world/>
</body>

```
修改后的 helloWorld 指令如下：
```

app.directive('helloWorld', function() {
  return {
    restrict: 'AE',
    replace: true,
    template: '<p style="background-color:![](/data/dokuwikicolor)">Hello World',
    link: function(scope, elem, attrs) {
      elem.bind('click', function() {
        elem.css('background-color', 'white');
        scope.$apply(function() {
          scope.color = "white";
        });
      });
      elem.bind('mouseover', function() {
        elem.css('cursor', 'pointer');
      });
    }
  };
});

```
我们注意到指令定义中的** link 函数。 它有三个参数**：
  * scope – **指令的scope。在我们的例子中，指令的scope就是父controller的scope。**
  * elem – 指令的jQLite(jQuery的子集)**包装DOM元素**。**如果你在引入AngularJS之前引入了jQuery，那么这个元素就是jQuery元素**，而不是jQLite元素。由于这个元素已经被jQuery/jQLite包装了，所以我们就在进行DOM操作的时候就**不需要再使用 $()来进行包装。**
  * attr – **一个包含了指令所在元素的属性的标准化的参数对象。**举个例子，你给一个HTML元素添加了一些属性：，那么可以在 link 函数中通过 attrs.someAttribute 来使用它。
**link函数主要用来为DOM元素添加事件监听、监视模型属性变化、以及更新DOM。**在上面的指令代码片段中，我们添加了两个事件， click，和 mouseover。click 处理函数用来重置 <p> 的背景色，而 mouseover 处理函数改变鼠标为 pointer。在模板中有一个表达式 ![](/data/dokuwikicolor)，当父scope中的 color 发生变化时，它用来改变 Hello World 文字的背景色。 

###  compile函数 
**compile 函数在 link 函数被执行` 之前 `用来做一些DOM改造**。它接收下面的参数：
  * tElement – 指令所在的元素
  * attrs – 元素上赋予的参数的标准化列表
要注意的是 **compile 函数不能访问 scope，并且必须返回一个 link 函数**。但是如果没有设置 compile 函数，你可以正常地配置 link 函数，（` 有了compile，就不能用link，link函数由compile返回 `）。compile函数可以写成如下的形式：
```

app.directive('test', function() {
  return {
    compile: function(tElem,attrs) {
      
//do optional DOM transformation here
      return function(scope,elem,attrs) {
        
//linking function here
      };
    }
  };
});

```
**大多数的情况下，你只需要使用 link 函数**。这是因为大部分的指令只需要考虑注册事件监听、监视模型、以及更新DOM等，这些都可以在 link 函数中完成。 但是对于像 ng-repeat 之类的指令，需要克隆和重复 DOM 元素多次，在 link 函数执行之前由 compile 函数来完成。

###  指令是如何被编译的 
当应用引导启动的时候，Angular开始使用 $compile 服务遍历DOM元素。这个服务基于注册过的指令在标记文本中搜索指令。一旦所有的指令都被识别后，Angular执行他们的 compile 方法。如前面所讲的，compile 方法返回一个 link 函数，**被添加到稍后执行的 link 函数列表中**。这被称为编译阶段。如果一个指令需要被克隆很多次（比如 ng-repeat），compile函数只在编译阶段被执行一次，复制这些模板，但是link 函数会针对每个被复制的实例被执行。所以分开处理，让我们在性能上有一定的提高。**这也说明了为什么在 compile 函数中不能访问到scope对象。 在编译阶段之后，就开始了链接（linking）阶段。**在这个阶段，所有收集的 link 函数将被一一执行。指令创造出来的模板会在正确的scope下被解析和处理，然后返回具有事件响应的真实的DOM节点。

###  改变指令的Scope 
**默认情况下，指令获取它父节点的controller的scope。但这并不适用于所有情况**。如果将父controller的scope暴露给指令，那么他们可以随意地修改 scope 的属性。在某些情况下，**你的指令希望能够添加一些仅限内部使用的属性和方法。如果我们在父的scope中添加，会污染父scope。** 其实我们还有两种选择：
  * 一个子scope – 这个scope原型继承子父scope。
  * 一个隔离的scope – 一个孤立存在不继承自父scope的scope。这样的scope可以**通过指令定义对象中 scope 属性来配置。**
下面的代码片段是一个例子：
```

app.directive('helloWorld', function() {
  return {
    scope: true,  //创建一个继承自父scope的新的子scope
// use a child scope that inherits from parent
    restrict: 'AE',
    replace: 'true',
    template: '<h3>Hello World!!</h3>'
  };
});

```
上面的代码，让Angular给指令创建一个继承自父socpe的新的子scope。** 另外一个选择，隔离的scope：**
```

app.directive('helloWorld', function() {
  return {
    scope: {},  //创建一个隔离scope
// use a new isolated scope
    restrict: 'AE',
    replace: 'true',
    template: '<h3>Hello World!!</h3>'
  };
});

```
这个指令使用了一个隔离的scope。**隔离的scope在我们想要创建可重用的指令的时候是非常有好处的**。**通过使用隔离的scope，我们能够保证我们的指令是自包含的，可以被很容易的插入到HTML应用中**。 ` 它内部不能访问父的scope，所保证了父scope不被污染。 ` 在我们的 helloWorld 指令例子中，如果我们将 scope 设置成 {}，那么上面的代码将不会工作。 它会创建一个新的隔离的scope，那么相应的表达式 ![](/data/dokuwikicolor) 会指向到这个新的scope中，它的值将是 undefined. 
**但是使用隔离的scope并不意味着我们完全不能访问父scope的属性。其实有一些技术可以允许我们访问父scope的属性，甚至监视他们的变化。**我们会在指令这个系列的第二部分中讨论这些技术.

###  隔离scope和父scope之间的数据绑定 
通常，隔离指令的scope会带来很多的便利，尤其是在你要操作多个scope模型的时候。但有时为了使代码能够正确工作，你也需要从指令内部访问父scope的属性。好消息是Angular给了你足够的灵活性让你能够有选择性的通过绑定的方式传入父scope的属性。让我们重温一下我们的 helloWorld 指令，它的背景色会随着用户在输入框中输入的颜色名称而变化。还记得当我们对这个指令使用隔离scope的之后，它不能工作了吗？现在，我们来让它恢复正常。
假设我们已经初始化完成app这个变量所指向的Angular模块。那么我们的 helloWorld 指令如下面代码所示：
```

app.directive('helloWorld', function() {
  return {
    scope: {},
    restrict: 'AE',
    replace: true,
    template: '<p style="background-color:![](/data/dokuwikicolor)">Hello World</p>',
    link: function(scope, elem, attrs) {
      elem.bind('click', function() {
        elem.css('background-color','white');
        scope.$apply(function() {
          scope.color = "white";
        });
      });
      elem.bind('mouseover', function() {
        elem.css('cursor', 'pointer');
      });
    }
  };
});

```
```

<body ng-controller="MainCtrl">
  <input type="text" ng-model="color" placeholder="Enter a color"/>
  <hello-world/>
</body>

```
**上面的代码现在是不能工作的。因为我们用了一个隔离的scope，**指令内部的 ![](/data/dokuwikicolor) 表达式被隔离在指令内部的scope中(不是父scope)。但是外面的输入框元素中的 ng-model 指令是指向父scope中的 color 属性的。所以，**我们需要一种方式来绑定隔离scope和父scope中的这两个参数。**在Angular中，**这种数据绑定可以通过为指令所在的HTML元素添加属性和并指令定义对象中配置相应的 scope 属性来实现。**

**建立隔离scope和父scope之间的数据绑定的几种方式。**
###  选择一：使用 @ 实现单向文本绑定 
在下面的指令定义中，我们指定了隔离scope中的属性 color 绑定到指令所在HTML元素上的参数 colorAttr。在HTML标记中，你可以看到 ![](/data/dokuwikicolor)表达式被指定给了 color-attr 参数。当表达式的值发生改变时，color-attr 参数也跟着改变。隔离scope中的 color 属性的值也相应地被改变。
```

app.directive('helloWorld', function() {
  return {
    scope: {
      color: '@colorAttr'
    },
    ....
    
// the rest of the configurations
  };
});

```
```

<body ng-controller="MainCtrl">
  <input type="text" ng-model="color" placeholder="Enter a color"/>
  <hello-world color-attr="![](/data/dokuwikicolor)"/>
</body>

```
**我们称这种方式为单向绑定**，是因为在这种方式下，你只能将字符串(使用表达式{{}})传递给参数。当父scope的属性变化时，你的隔离scope模型中的属性值跟着变化。你甚至可以在指令内部监控这个scope属性的变化，并且触发一些任务。**然而，反向的传递并不工作**。你不能通过对隔离scope属性的操作来改变父scope的值。
注意点：
**当隔离scope属性和指令元素参数的名字一样是，你可以更简单的方式设置scope绑定：**
```

app.directive('helloWorld', function() {
  return {
    scope: {
      color: '@'
    },
    ....
    
// the rest of the configurations
  };
});

```
```

<hello-world color="![](/data/dokuwikicolor)"/>

```

###  选择二：使用 = 实现双向绑定 
让我们将指令的定义改变成下面的样子：
```

app.directive('helloWorld', function() {
  return {
    scope: {
      color: '='
    },
    ....
    
// the rest of the configurations
  };
});

```
```

<body ng-controller="MainCtrl">
  <input type="text" ng-model="color" placeholder="Enter a color"/>
  <hello-world color="color"/>
</body>

```
与 @ 不同，这种方式让你能够**给属性指定一个真实的scope数据模型**，而不是简单的字符串。这样你就可以传递简单的字符串、数组、甚至复杂的对象给隔离scope。**同时，还支持双向的绑定**。每当父scope属性变化时，相对应的隔离scope中的属性也跟着改变，反之亦然。和之前的一样，你也可以监视这个scope属性的变化。

###  选择三：使用 & 在父scope中执行函数 
**有时候从隔离scope中调用父scope中定义的函数是非常有必要的**。为了能够访问外部scope中定义的函数，我们使用 &。比如我们想要从指令内部调用 sayHello() 方法。下面的代码告诉我们该怎么做：
```

app.directive('sayHello', function() {
  return {
    scope: {
      sayHelloIsolated: '&'
    },
    ....
    
// the rest of the configurations
  };
});

```
```

<body ng-controller="MainCtrl">
  <input type="text" ng-model="color" placeholder="Enter a color"/>
  <say-hello sayHelloIsolated="sayHello()"/>
</body>

```
其中sayHello()是我们的父scope中定义的函数。这样就实现了子scope调用父scope中的函数了。

###  Transclusion（嵌入） 
**Transclusion是让我们的指令包含任意内容的方法**。我们可以延时提取并在正确的scope下编译这些嵌入的内容，最终将它们放入指令模板中指定的位置。 如果你在指令定义中**设置 transclude:true**，一个新的嵌入的scope会被创建，它原型继承子父scope。 如果你想要你的指令使用隔离的scope，但是它所包含的内容能够在父scope中执行，transclusion也可以帮忙。
假设我们注册一个如下的指令：
```

app.directive('outputText', function() {
  return {
    transclude: true,
    scope: {},
    template: '<div ng-transclude></div>'
  };
});

```
```

<div output-text>
  <p>Hello ![](/data/dokuwikiname)</p>
</div>

```
ng-transclude 指明在哪里放置被嵌入的内容。在这个例子中DOM内容 &lt;nowiki&gt;&lt;p&gt;Hello ![](/data/dokuwikiname)&lt;/p&gt;&lt;/nowiki&gt; 被提取和放置到 &lt;nowiki&gt;&lt;div ng-transclude=""&gt;&lt;/div&gt;&lt;/nowiki&gt; 内部。有一个很重要的点需要注意的是，**表达式&lt;nowiki&gt;![](/data/dokuwikiname)&lt;/nowiki&gt;所对应的属性是在父scope中被定义的，而非子scope。**

###  transclude:’element’ 和 transclude:true的区别 
有时候我我们要嵌入指令元素本身，而不仅仅是它的内容。在这种情况下，我们需要使用 transclude:’element’。它和 transclude:true 不同，它将标记了 ng-transclude 指令的元素一起包含到了指令模板中。
##  父scope、子scope以及隔离scope的区别 
**默认情况下，指令不会创建一个新的scope，而是沿用父scope。（ng-controller除外，它会创建一个新的scope）**但是在很多情况下，这并不是我们想要的。如果你的指令重度地使用父scope的属性、甚至创建新的属性，**会污染父scope**。让所有的指令都使用同一个父scope不会是一个好主意，因为任何人都可能修改这个scope中的属性。因此，下面的这个原则也许可以帮助你为你的指令选择正确的scope。
1.父scope(scope: false) – **这是默认情况**。如果你的指令不操作父scoe的属性，你就不需要一个新的scope。这种情况下是可以使用父scope的。
2.子scope(**scope: true**) – **这会为指令创建一个新的scope，并且原型继承自父scope**。如果你的指令scope中的属性和方法与其他的指令以及父scope都没有关系的时候，你应该创建一个新scope。在这种方式下，你同样拥有父scope中所定义的属性和方法。
3.隔离scope(**scope:{}**) – 这就像一个沙箱！当你创建的指令是自包含的并且可重用的，你就需要使用这种scope。你在指令中会创建很多scope属性和方法，它们仅在指令内部使用，永远不会被外部的世界所知晓。如果是这样的话，隔离的scope是更好的选择。隔离的scope不会继承父scope。


##  controller 函数和 require 
**如果你想要允许其他的指令和你的指令发生交互时，你需要使用 controller 函数。**比如有些情况下，你需要通过组合两个指令来实现一个UI组件。那么你可以通过如下的方式来给指令添加一个 controller 函数。
```

app.directive('outerDirective', function() {
  return {
    scope: {},
    restrict: 'AE',
    controller: function($scope, $compile, $http) {
      
// $scope is the appropriate scope for the directive
      this.addChild = function(nestedDirective) { 
// this refers to the controller
        console.log('Got the message from nested directive:' + nestedDirective.message);
      };
    }
  };
});

```
这个代码为指令添加了一个名叫 outerDirective 的controller。当另一个指令想要交互时，它需要声明它对你的指令 controller 实例的引用(require)。可以通过如下的方式实现：
```

app.directive('innerDirective', function() {
  return {
    scope: {},
    restrict: 'AE',
    require: '^outerDirective',
    link: function(scope, elem, attrs, controllerInstance) {
      
//the fourth argument is the controller instance you require
      scope.message = "Hi, Parent directive";
      controllerInstance.addChild(scope);
    }
  };
});

```
相应的HTML代码如下：
```

<outer-directive>
  <inner-directive></inner-directive>
</outer-directive>

```
require: ‘^outerDirective’ 告诉Angular在元素以及它的父元素中搜索controller。这样被找到的** controller 实例会作为第四个参数被传入到 link 函数中。**在我们的例子中，我们将嵌入的指令的scope发送给父亲指令。
参考：http://blog.jobbole.com/62249/
http://blog.jobbole.com/62999/
