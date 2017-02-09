title: angularjs入门 

#  AngularJS入门 
快速开始：http://angularjs.cn/A002
Angular Guide中文版：http://docs.ngnice.com/guide
入门教程:http://angularjs.cn/tag/AngularJS
API:http://docs.angularjs.cn/api
360网站卫士CDN:http://libs.useso.com/js.php
附加参考：http://blog.csdn.net/yy374864125/article/details/41349417
http://www.lovelucy.info/understanding-scopes-in-angularjs.html

AngularJS使用了不同的方法，它尝试去补足HTML本身在构建应用方面的缺陷。AngularJS通过使用我们称为标识符(directives)的结构，让浏览器能够识别新的语法。例如：
  使用双大括号{{}}语法进行数据绑定；
  使用DOM控制结构来实现迭代或者隐藏DOM片段；
  支持表单和表单的验证；
  能将逻辑代码关联到相关的DOM元素上；
  能将HTML分组成可重用的组件。
  
**AngularJS的可爱之处**
AngularJS通过为开发者呈现一个更高层次的抽象来简化应用的开发。如同其他的抽象技术一样，这也会损失一部分灵活性。换句话说，**并不是所有的应用都适合用AngularJS来做。AngularJS主要考虑的是构建CRUD应用**。幸运的是，至少90%的WEB应用都是CRUD应用。但是要了解什么适合用AngularJS构建，就得了解什么不适合用AngularJS构建。
如游戏，图形界面编辑器，这种DOM操作很频繁也很复杂的应用，和CRUD应用就有很大的不同，它们不适合用AngularJS来构建。像这种情况用一些更轻量、简单的技术如jQuery可能会更好。

![](/data/dokuwiki/web/pasted/20151009-085540.png)
##  Hello World 
```

<!doctype html>
<html ng-app>
    <head>
        <!-- // <script src="http://code.angularjs.org/angular-1.0.1.min.js"></script> -->
        <script src="http://libs.useso.com/js/angular.js/1.2.9/angular.min.js"></script>
    </head>
    <body>
        Hello ![](/data/dokuwiki'World')!
    </body>
</html>

```
	html ng-app标记` ng-app `告诉AngularJS处理整个HTML页并引导应用：ng-app指令标记了AngularJS脚本的作用域，在<html/>中添加ng-app属性即说明整个<html/>都是AngularJS脚本作用域。开发者也可以在局部使用ng-app指令，如<div ng-app/>，则AngularJS脚本仅在该<div/>中运行。通过ngApp指令来自动引导AngularJS应用是一种简洁的方式，适合大多数情况。
在高级开发中，例如使用脚本装载应用，**您也可以使用bootstrap手动引导AngularJS应用。**

	Hello ![](/data/dokuwiki'World')!注意，使用双大括号标记{{}}的内容是问候语中**绑定的表达式**，这个表达式是一个简单的字符串‘World’。
##  演示AngularJS的双向数据绑定 
```

<!doctype html>
<html ng-app>
    <head>
       <!-- // <script src="http://code.angularjs.org/angular-1.0.1.min.js"></script> -->
        <script src="http://libs.useso.com/js/angular.js/1.2.9/angular.min.js"></script>
    </head>
    <body>
        Your name: <input type="text" ng-model="yourname" placeholder="World">
        <hr>
        Hello ![](/data/dokuwikiyourname || 'World')!
    </body>
</html>

```
**该示例有一下几点重要的注意事项：**
  * 文本输入指令<input ng-model="yourname" />绑定到一个叫yourname的模型变量。
  * 双大括号标记将yourname模型变量添加到问候语文本。
  * 你不需要为该应用另外注册一个事件侦听器或添加事件处理程序！
现在试着在输入框中键入您的名称，您键入的名称将立即更新显示在问候语中。 这就是**AngularJS双向数据绑定**的概念。 输入框的任何更改会立即反映到模型变量（一个方向），模型变量的任何更改都会立即反映到问候语文本中（另一方向）。

**AngularJS应用引导过程有3个重要点：**
  * 注入器(injector)将用于创建此应用程序的依赖注入(dependency injection)；
  * 注入器将会创建根作用域作为我们应用模型的范围；
  * AngularJS将会链接根作用域中的DOM，从用ngApp标记的HTML标签开始，逐步处理DOM中指令和绑定。

一旦AngularJS应用引导完毕，它将继续侦听浏览器的HTML触发事件，如鼠标点击事件、按键事件、HTTP传入响应等改变DOM模型的事件。这类事件一旦发生，AngularJS将会自动检测变化，并作出相应的处理及更新。
##  视图和模板 
对于AngularJS应用，我们鼓励使用模型-视图-控制器（MVC）模式解耦代码和分离关注点。考虑到这一点，我们用AngularJS来为我们的应用添加一些模型、视图和控制器。
**视图和模板**
在AngularJS中，一个视图是模型通过HTML模板渲染之后的映射。这意味着，不论模型什么时候发生变化，AngularJS会实时更新结合点，随之更新视图。
```

<html ng-app>
<head>
  ...
  <script src="lib/angular/angular.js"></script>
  <script src="js/controllers.js"></script>
</head>
<body ng-controller="PhoneListCtrl">

  <ul>
    <li ng-repeat="phone in phones">
      ![](/data/dokuwikiphone.name)
    <p>![](/data/dokuwikiphone.snippet)</p>
    </li>
  </ul>
</body>
</html>

```
	因为这里我们使用ngRepeat指令和两个用花括号包裹起来的AngularJS表达式——![](/data/dokuwikiphone.name)和![](/data/dokuwikiphone.snippet)——能达到同样的效果。
* 在<li>标签里面的` ng-repeat="phone in phones" `语句是一个**AngularJS迭代器**。这个迭代器告诉AngularJS用第一个<li>标签作为模板为列表中的每一部手机创建一个<li>元素。
* 包裹在phone.name和phone.snippet周围的花括号标识着**数据绑定**。和常量计算不同的是，这里的表达式实际上是我们应用的一个数据模型引用，这些我们在PhoneListCtrl控制器里面都设置好了。

**模型和控制器**
在PhoneListCtrl控制器里面初始化了数据模型
app/js/controller.js:
```


function PhoneListCtrl($scope) {
  $scope.phones = [
    {"name": "Nexus S",
     "snippet": "Fast just got faster with Nexus S."},
    {"name": "Motorola XOOM™ with Wi-Fi",
     "snippet": "The Next, Next Generation tablet."},
    {"name": "MOTOROLA XOOM™",
     "snippet": "The Next, Next Generation tablet."}
  ];
}

```
控制器允许我们建立模型和视图之间的数据绑定。我们是这样把表现层，数据和逻辑部件联系在一起的：
PhoneListCtrl——控制器方法的名字（在JS文件controllers.js中）和<body>标签里面的` ngController `指令的值相匹配。
数据此时与注入到我们控制器函数的` 作用域（$scope） `相关联。当应用启动之后，会有一个**根作用域**被创建出来，而控制器的作用域是根作用域的一个典型后继。这个控制器的作用域对所有<body ng-controller="PhoneListCtrl">标记内部的数据绑定有效。
**AngularJS的作用域**理论非常重要：一个作用域可以视作模板、模型和控制器协同工作的粘接器。AngularJS使用作用域，同时还有模板中的信息，数据模型和控制器。这些可以帮助模型和视图分离，但是他们两者确实是同步的！任何对于模型的更改都会即时反映在视图上；任何在视图上的更改都会被立刻体现在模型中。
##  迭代器与全文搜索 
```

<div class="container-fluid">
  <div class="row-fluid">
    <div class="span2">
      <!--Sidebar content-->

      Search: <input ng-model="query">

    </div>
    <div class="span10">
      <!--Body content-->

      <ul class="phones">
        <li ng-repeat="phone in phones | filter:query">
          ![](/data/dokuwikiphone.name)
        <p>![](/data/dokuwikiphone.snippet)</p>
        </li>
      </ul>

       </div>
  </div>
</div>

```
我们现在添加了一个<input>标签，并且使用AngularJS的$filter函数来处理ngRepeat指令的输入。

这样允许用户输入一个搜索条件，立刻就能看到对电话列表的搜索结果。我们来解释一下新的代码：
**数据绑定**： 这是AngularJS的一个核心特性。当页面加载的时候，AngularJS会根据输入框的属性值名字，将其与数据模型中相同名字的变量绑定在一起，以确保两者的同步性。
在这段代码中，用户在输入框中输入的数据名字称作query，会立刻作为列表迭代器` （phone in phones | filter:query`） `其过滤器的输入。当数据模型引起迭代器输入变化的时候，迭代器可以高效得更新DOM将数据模型最新的状态反映出来。
**使用filter过滤器**：filter函数使用query的值来创建一个只包含匹配query记录的新数组。
**ngRepeat会根据filter过滤器生成的手机记录数据数组来自动更新视图。整个过程对于开发者来说都是透明的。**
###  绑定输入到页面标题 
第一步<html ng-app ` ng-controller="PhoneListCtrl" `>
第二步<title`  ng-bind-template `="Google Phone Gallery: ![](/data/dokuwikiquery)">Google Phone Gallery</title>
##  双向绑定与排序demo 
```

Search: <input ng-model="query">
Sort by:
<select ng-model="orderProp">
  <option value="name">Alphabetical</option>
  <option value="age">Newest</option>
</select>


<ul class="phones">
  <li ng-repeat="phone in phones | filter:query | orderBy:orderProp">
    ![](/data/dokuwikiphone.name)
    <p>![](/data/dokuwikiphone.snippet)</p>
  </li>
</ul>

```
我们在index.html中做了如下更改：

首先，我们增加了一个叫做orderProp的<select>标签，这样我们的用户就可以选择我们提供的两种排序方法。
然后，在filter过滤器后面添加一个orderBy过滤器用其来处理进入迭代器的数据。orderBy过滤器以一个数组作为输入，复制一份副本，然后把副本重排序再输出到迭代器。
AngularJS在select元素和orderProp模型之间创建了一个**双向绑定**。而后，orderProp会被用作orderBy过滤器的输入。
正如我们在步骤3中讨论数据绑定和迭代器的时候所说的一样，无论什么时候数据模型发生了改变（比如用户在下拉菜单中选了不同的顺序），AngularJS的数据绑定会让视图自动更新。没有任何笨拙的DOM操作！
controllers.js：
```


function PhoneListCtrl($scope) {
  $scope.phones = [
    {"name": "Nexus S",
     "snippet": "Fast just got faster with Nexus S.",
     "age": 0},
    {"name": "Motorola XOOM™ with Wi-Fi",
     "snippet": "The Next, Next Generation tablet.",
     "age": 1},
    {"name": "MOTOROLA XOOM™",
     "snippet": "The Next, Next Generation tablet.",
     "age": 2}
  ];

  $scope.orderProp = 'age';
}

```
我们修改了phones模型—— 手机的数组 ——为每一个手机记录其增加了一个age属性。我们会根据age属性来对手机进行排序。
我们在控制器代码里加了一行让orderProp的默认值为age。如果我们不设置默认值，这个模型会在我们的用户在下拉菜单选择一个顺序之前一直处于未初始化状态。
##  模块与服务 
invoice2.js
```

angular.module('invoice2', ['finance2'])
  .controller('InvoiceController', ['currencyConverter', function(currencyConverter) {
    this.qty = 1;
    this.cost = 2;
    this.inCurr = 'EUR';
    this.currencies = currencyConverter.currencies;

    this.total = function total(outCurr) {
      return currencyConverter.convert(this.qty * this.cost, this.inCurr, outCurr);
    };
    this.pay = function pay() {
      window.alert("谢谢！");
    };
  }]);

```
finance2.js
```

angular.module('finance2', [])
  .factory('currencyConverter', function() {
    var currencies = ['USD', 'EUR', 'CNY'],
        usdToForeignRates = {
      USD: 1,
      EUR: 0.74,
      CNY: 6.09
    };
    return {
      currencies: currencies,
      convert: convert
    };

    function convert(amount, inCurr, outCurr) {
      return amount * usdToForeignRates[outCurr] * 1 / usdToForeignRates[inCurr];
    }
  });

```
index.html
```

<!doctype html>
<html ng-app="invoice2">
  <head>
    <script src="http://code.angularjs.org/1.2.25/angular.min.js"></script>
    <script src="finance2.js"></script>
    <script src="invoice2.js"></script>
  </head>
  <body>
    <div ng-controller="InvoiceController as invoice">
      <b>订单:</b>
      <div>
        数量: <input type="number" ng-model="invoice.qty" required >
      </div>
      <div>
        单价: <input type="number" ng-model="invoice.cost" required >
        <select ng-model="invoice.inCurr">
          <option ng-repeat="c in invoice.currencies"></option>
        </select>
      </div>
      <div>
        <b>总价:</b>
        <span ng-repeat="c in invoice.currencies">
          
        </span>
        <button class="btn" ng-click="invoice.pay()">支付</button>
      </div>
    </div>
  </body>
</html>

```
我们把服务convertCurrency函数和所支持的币种的定义独立到一个新的文件：finance.js。但是**控制器怎样才能找到这个独立的函数呢？**
这下该“依赖注入(Dependency Injection)”出场了。依赖注入(DI)是一种设计模式(Design Pattern)，它用于解决下列问题：我们创建了对象和函数，但是它们怎么得到自己所依赖的对象呢？** Angular中的每一样东西都是用依赖注入(DI)的方式来创建和使用的**，比如指令(Directive)、过滤器(Filter)、控制器(Controller)、服务(Service)。 在Angular中，**依赖注入(DI)的容器(container)叫做"注入器(injector)"。**
**` 要想进行依赖注入，你必须先把这些需要协同工作的对象和函数注册(Register)到某个地方。在Angular中，这个地方叫做“模块(module)”。 `**

在上面这个例子中： 模板包含了一个ng-app="invoice2"指令。这告诉Angular使用invoice2模块作为该应用程序的**主模块**。 像angular.module('invoice', ['finance'])这样的代码告诉Angular：**invoice模块依赖于finance模块。** 这样一来，Angular就能同时使用InvoiceController这个控制器和currencyConverter这个服务了。

上面这个例子中，我们通过一个返回currencyConverter函数的函数作为创建currencyConverter服务的工厂。 （译注：js中的“工厂(factory)”是指一个以函数作为“返回值”的函数）

回到刚才的问题：InvoiceController该怎样获得这个currencyConverter函数的引用呢？ 在Angular中，这非常简单，**只要在构造函数中定义一些具有特定名字的参数就可以了**。 这时，注入器(injector)就可以按照正确的依赖关系创建这些对象，并且根据名字把它们传入那些依赖它们的对象工厂中。 在我们的例子中，InvoiceController有一个叫做currencyConverter的参数。 根据这个参数，Angular就知道InvoiceController依赖于currencyConverter，取得currencyConverter服务的实例，并且把它作为参数传给InvoiceController的构造函数。

这次改动中的最后一点是我们现在把一个数组作为参数传入到module.controller函数中，而不再是一个普通函数。 这个数组前面部分的元素包含这个控制器所依赖的一系列服务的名字，最后一个元素则是这个控制器的构造函数。 **Angular可以通过这种数组语法来定义依赖，以免js代码压缩器(Minifier)破坏这个“依赖注入”的过程**。 因为这些js代码压缩器通常都会把构造函数的参数重命名为很短的名字，比如a，而常规的依赖注入是需要根据参数名来查找“被注入对象”的。 （译注：因为字符串不会被js代码压缩器重命名，所以数组语法可以解决这个问题。）
##  XHR和依赖注入 
现在我们**使用AngularJS一个内置服务**` $http `来获取一个更大的手机记录数据集。我们将使用AngularJS的依赖注入（dependency injection (DI)）功能来为PhoneListCtrl控制器提供这个AngularJS服务。
我们在控制器中使用AngularJS服务$http向你的Web服务器发起一个HTTP请求，以此从app/phones/phones.json文件中获取数据。$http仅仅是AngularJS众多内建服务中之一，这些服务可以处理一些Web应用的通用操作。AngularJS能将这些服务注入到任何你需要它们的地方。
controllers.js

```

function PhoneListCtrl($scope, $http) {
  $http.get('phones/phones.json').success(function(data) {
    $scope.phones = data;
  });

  $scope.orderProp = 'age';
}

//PhoneListCtrl.$inject = ['$scope', '$http'];

```
###  关于JS压缩带来的问题 
由于AngularJS是通过控制器构造函数的参数名字来推断依赖服务名称的。所以如果你要压缩)PhoneListCtrl控制器的JS代码，它所有的参数也同时会被压缩，这时候依赖注入系统就不能正确的识别出服务了。
为了克服压缩引起的问题，只要在控制器函数里面给$inject属性赋值一个依赖服务标识符的数组，就像被注释掉那段最后一行那样：
` PhoneListCtrl.$inject = ['$scope', '$http']; `
另一种方法也可以用来指定依赖列表并且避免压缩问题——使用Javascript数组方式构造控制器：把要注入的服务放到一个字符串数组（代表依赖的名字）里，数组最后一个元素是控制器的方法函数：
` var PhoneListCtrl = ['$scope', '$http', function($scope, $http) { /* constructor body */ }]; `
上面提到的两种方法都能和AngularJS可注入的任何函数完美协作，要选哪一种方式完全取决于你们项目的编程风格，**建议使用数组方式。**
##  链接与图片模板 
```

  <ul class="phones">
          <li ng-repeat="phone in phones | filter:query | orderBy:orderProp" class="thumbnail">
            <a href="#/phones/![](/data/dokuwikiphone.id)" class="thumb"><img ng-src="![](/data/dokuwikiphone.imageUrl)"></a>
            <a href="#/phones/![](/data/dokuwikiphone.id)">![](/data/dokuwikiphone.name)</a>
            <p>![](/data/dokuwikiphone.snippet)</p>
          </li>
        </ul>
...

```
这些链接将来会指向每一部电话的详细信息页。不过现在为了产生这些链接，我们在href属性里面使用我们早已熟悉的双括号数据绑定。在步骤2，我们添加了` ![](/data/dokuwikiphone.name) `绑定作为元素内容。在这一步，我们在元素属性中使用` ![](/data/dokuwikiphone.id) `绑定。
我们同样为每条记录添加手机图片，只需要**使用ngSrc指令代替<img>的src属性标签**就可以了。如果我们仅仅用一个正常src属性来进行绑定（` <img class="diagram" src="![](/data/dokuwikiphone.imageUrl)"> `），浏览器会把AngularJS的![](/data/dokuwiki 表达式 )标记直接进行字面解释，并且发起一个向非法url`  http:/ /localhost:8000/app/![](/data/dokuwikiphone.imageUrl) `的请求。
**因为浏览器载入页面时，同时也会请求载入图片，AngularJS在页面载入完毕时才开始编译**——浏览器请求载入图片时` ![](/data/dokuwikiphone.imageUrl) `还没得到编译！有了这个ngSrc指令会避免产生这种情况，使用ngSrc指令防止浏览器产生一个指向非法地址的请求。

##  路由与多视图 
AngularJS中应用的路由通过` $routeProvider `来声明，它是` $route `服务的提供者。**这项服务使得控制器、视图模板与当前浏览器的URL可以轻易集成**。应用这个特性我们就可以实现深链接，它允许我们使用浏览器的历史(回退或者前进导航)和书签。
**App 模块**
app/js/app.js
```

angular.module('phonecat', []).
  config(['$routeProvider', function($routeProvider) {
  $routeProvider.
      when('/phones', {templateUrl: 'partials/phone-list.html',   controller: PhoneListCtrl}).
      when('/phones/:phoneId', {templateUrl: 'partials/phone-detail.html', controller: PhoneDetailCtrl}).
      otherwise({redirectTo: '/phones'});
}]);

```
为了给我们的应用配置路由，我们需要给应用创建一个模块。我们管这个模块叫做phonecat，并且通过使用configAPI，我们请求把` $routeProvider `注入到我们的配置函数并且使用$routeProvider.whenAPI来定义我们的路由规则。
当URL 映射段为/phone/:phoneId时，手机详细信息视图被显示出来。这里:phoneId是URL的变量部分。
:phoneId参数的使用。` $route `服务使用路由声明/phones/:phoneId作为一个匹配当前URL的模板。所有以:符号声明的变量（此处变量为phones）都会被提取，然后存放在` $routeParams `对象中。
为了让我们的应用引导我们新创建的模块，**我们同时需要在ngApp指令的值上指明模块的名字：**
app/index.html
<!doctype html>
<html lang="en"** ng-app="phonecat"**>
**控制器**
app/js/controllers.js
```

...
function PhoneDetailCtrl($scope, $routeParams) {
  $scope.phoneId = $routeParams.phoneId;
}
//PhoneDetailCtrl.$inject = ['$scope', '$routeParams'];

```
**模板**
` $route服务通常和ngView指令 `一起使用。**ngView指令的角色是为当前路由把对应的视图模板载入到布局模板中**。
app/index.html
```


<html lang="en" ng-app="phonecat">
<head>
...
  <script src="lib/angular/angular.js"></script>
  <script src="js/app.js"></script>
  <script src="js/controllers.js"></script>
</head>
<body>

  <div ng-view></div>

</body>
</html>

```
注意，我们把index.html模板里面大部分代码移除，我们只放置了一个<div>容器，这个<div>具有ng-view属性。我们删除掉的代码现在被放置在phone-list.html模板中：
app/partials/phone-list.html
```


<div class="container-fluid">
  <div class="row-fluid">
    <div class="span2">
      <!--Sidebar content-->

      Search: <input ng-model="query">
      Sort by:
      <select ng-model="orderProp">
        <option value="name">Alphabetical</option>
        <option value="age">Newest</option>
      </select>

    </div>
    <div class="span10">
      <!--Body content-->

      <ul class="phones">
        <li ng-repeat="phone in phones | filter:query | orderBy:orderProp" class="thumbnail">    
          <a href="#/phones/![](/data/dokuwikiphone.id)" class="thumb"><img ng-src="![](/data/dokuwikiphone.imageUrl)"></a>
          <a href="#/phones/![](/data/dokuwikiphone.id)">![](/data/dokuwikiphone.name)</a>
          <p>![](/data/dokuwikiphone.snippet)</p>
        </li>
      </ul>

    </div>
  </div>
</div>

```
同时我们为手机详细信息视图添加一个占位模板。

app/partials/phone-detail.html

TBD: detail view for ![](/data/dokuwikiphoneId)
**注意到我们的布局模板中没再添加PhoneListCtrl或PhoneDetailCtrl控制器属性！**
**问题说明**
试着在index.html上增加一个![](/data/dokuwikiorderProp)绑定，当你在手机列表视图上时什么也没变。**这是因为orderProp模型仅仅在PhoneListCtrl管理的作用域下才是可见的，**这与<div ` ng-view `>元素相关。如果你在phone-list.html模板中加入同样的绑定，那么这个绑定会按你设想的那样被渲染出来。
##  定制过滤器 
**ng的内置过滤器**
ng内置了九种过滤器，使用方法都非常简单，看文档即懂。不过为了以后不去翻它的文档，我在这里还是做一个详细的记录。
**currency(货币)、date(日期)、filter(子串匹配)、json(格式化json对象)、limitTo(限制个数)、lowercase(小写)、uppercase(大写)、number(数字)、orderBy(排序)**
为了创建一个新的过滤器，先创建一个phonecatFilters模块，并且将定制的过滤器注册给这个模块。
app/js/filters.js
```

angular.module('phonecatFilters', []).filter('checkmark', function() {
  return function(input) {
    return input ? '\u2713' : '\u2718';
  };
});

```
我们的过滤器命名为checkmark。**它的输入要么是true，要么是false，****并且我们返回两个表示true或false的unicode字符（\u2713和\u2718）。**
现在我们的过滤器准备好了，**我们需要将我们的phonecatFilters模块作为一个依赖注册到我们的主模块phonecat上。**
app/js/app/js
```

...
angular.module('phonecat', ['phonecatFilters']).
...

```
在AngularJS模板中使用过滤器的语法是：
```

![](/data/dokuwiki expression | filter )

```
**练习**

现在让我们来练习一下AngularJS内置过滤器，在index.html中加入如下绑定：

```

![](/data/dokuwiki "lower cap string" | uppercase )
![](/data/dokuwiki {foo/ "bar", baz/ 23} | json )
![](/data/dokuwiki 1304375948024 | date )
![](/data/dokuwiki 1304375948024 | date/"MM/dd/yyyy @ h/mma" )

```
我们也可以用一个输入框来创建一个模型，并且将之与一个过滤后的绑定结合在一起。在index.html中加入如下代码：
```

Uppercased: ![](/data/dokuwiki userInput | uppercase )

```
##  事件处理器 
控制器
app/js/controllers.js
```


...
function PhoneDetailCtrl($scope, $routeParams, $http) {
  $http.get('phones/' + $routeParams.phoneId + '.json').success(function(data) {
    $scope.phone = data;
    $scope.mainImageUrl = data.images[0];
  });

 $scope.setImage = function(imageUrl) {
    $scope.mainImageUrl = imageUrl;
  }
}

//PhoneDetailCtrl.$inject = ['$scope', '$routeParams', '$http'];

```
在PhoneDetailCtrl控制器中，我们创建了mainImageUrl模型属性，并且把它的默认值设为第一个手机图片的URL。

模板
app/partials/phone-detail.html
```


<img ng-src="![](/data/dokuwikimainImageUrl)" class="phone">

...

<ul class="phone-thumbs">
  <li ng-repeat="img in phone.images">
    <img ng-src="![](/data/dokuwikiimg)" ng-click="setImage(img)">
  </li>
</ul>
...

```
我们把大图片的ngSrc指令绑定到mainImageUrl属性上。
同时我们注册一个` ng-click `处理器到缩略图上。当一个用户点击缩略图的任意一个时，这个处理器会使用setImage事件处理函数来把mainImageUrl属性设置成选定缩略图的URL。
##  REST和定制服务 
代表RESTful客户端的定制服务。有了这个客户端我们可以用一种更简单的方式来发送XHR请求，而不用去关心更底层的$http服务（API、HTTP方法和URL）。
定制的服务被定义在app/js/services，所以我们需要在布局模板中引入这个文件。另外，我们也要加载` angularjs-resource.js `这个文件，它**包含了ngResource模块以及其中的$resource服务**，我们一会就会用到它们：
app/index.html
```

...
  <script src="js/services.js"></script>
  <script src="lib/angular/angular-resource.js"></script>
...

```
**服务**
app/js/services.js
```

angular.module('phonecatServices', ['ngResource']).
    factory('Phone', function($resource){
      return $resource('phones/:phoneId.json', {}, {
        query: {method:'GET', params:{phoneId:'phones'}, isArray:true}
      });
    });

```
我们使用模块API通过一个工厂方法**注册了一个定制服务**。我们传入服务的名字Phone和工厂函数。工厂函数和控制器构造函数差不多，它们都通过函数参数声明依赖服务。Phone服务声明了它依赖于$resource服务。
` $resource `服务使得用短短的几行代码就可以创建一个**RESTful客户端。**我们的应用使用这个客户端来代替底层的$http服务。
app/js/app.js
```

...
angular.module('phonecat', ['phonecatFilters', 'phonecatServices']).
...

```
我们需要把phonecatServices添加到phonecat的依赖数组里。

**控制器**
AngularJS的$resource相比于$http更加适合于与RESTful数据源交互。而且现在我们更容易理解控制器这些代码在干什么了。
app/js/controllers.js
```


...

function PhoneListCtrl($scope, Phone) {
  $scope.phones = Phone.query();
  $scope.orderProp = 'age';
}

//PhoneListCtrl.$inject = ['$scope', 'Phone'];

function PhoneDetailCtrl($scope, $routeParams, Phone) {
  $scope.phone = Phone.get({phoneId: $routeParams.phoneId}, function(phone) {
    $scope.mainImageUrl = phone.images[0];
  });

  $scope.setImage = function(imageUrl) {
    $scope.mainImageUrl = imageUrl;
  }
}

//PhoneDetailCtrl.$inject = ['$scope', '$routeParams', 'Phone'];

```
注意到，在PhoneListCtrl里我们把：
```

$http.get('phones/phones.json').success(function(data) {
  $scope.phones = data;
});
换成了：
$scope.phones = Phone.query();


```
##  指令（Directives）定制HTML标签 

指令是我个人最喜欢的特性。你是不是也希望浏览器可以做点儿有意思的事情？那么AngularJS可以做到。
**指令可以用来创建自定义的标签**。它们可以用来装饰元素或者操作DOM属性。可以作为标签、属性、注释和类名使用。
这里是一个例子，它监听一个事件并且针对的更新它的$scope ，如下：
```

myModule.directive('myComponent', function(mySharedService) {
    return {
        restrict: 'E',
        controller: function($scope, $attrs, mySharedService) {
            $scope.$on('handleBroadcast', function() {
                $scope.message = 'Directive: ' + mySharedService.message;
            });
        },
        replace: true,
        template: '<input>'
    };
});

```
然后，你可以使用这个自定义的directive来使用：
<my-component ng-model="message"></my-component>
使用一系列的组件来创建你自己的应用将会让你更方便的添加，删除和更新功能。

##  访问后端 
```

angular.module('finance3', [])
  .factory('currencyConverter', ['$http', function($http) {
    var YAHOO_FINANCE_URL_PATTERN =
          'http://query.yahooapis.com/v1/public/yql?q=select * from '+
          'yahoo.finance.xchange where pair in ("PAIRS")&format=json&'+
          'env=store://datatables.org/alltableswithkeys&callback=JSON_CALLBACK',
        currencies = ['USD', 'EUR', 'CNY'],
        usdToForeignRates = {};
    refresh();
    return {
      currencies: currencies,
      convert: convert,
      refresh: refresh
    };

    function convert(amount, inCurr, outCurr) {
      return amount * usdToForeignRates[outCurr] * 1 / usdToForeignRates[inCurr];
    }

    function refresh() {
      var url = YAHOO_FINANCE_URL_PATTERN.
                 replace('PAIRS', 'USD' + currencies.join('","USD'));
      return $http.jsonp(url).success(function(data) {
        var newUsdToForeignRates = {};
        angular.forEach(data.query.results.rate, function(rate) {
          var currency = rate.id.substring(3,6);
          newUsdToForeignRates[currency] = window.parseFloat(rate.Rate);
        });
        usdToForeignRates = newUsdToForeignRates;
      });
    }

```