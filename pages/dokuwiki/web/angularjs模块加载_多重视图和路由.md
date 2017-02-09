title: angularjs模块加载_多重视图和路由 

#  AngularJS模块加载、多重视图和路由 
**AngularJS模块可以在被加载和执行之前对其自身进行配置**，这样我们就可以在应用的加载阶段应用不同的逻辑组
##  应用启动配置config 
在模块加载阶段，AngularJS会在提供者Provider注册和配置的过程中通过` angular.module的config()方法 `对模块进行配置。在这个AngularJS的工作流中，**这个阶段是唯一能够在应用启动前进行修改的部分**。
```

angular.module('myApp',[]).config(function($provider){});

```
当对模块进行配置的时候需要格外注意：**只有少数几种类型的对象可以被注入到config()函数中：` 提供者和常量 `**
也可以定义多个配置块，他们会按照顺序执行的。
config()接受一个参数。参数可以是依赖注入形式的数组或函数。
```

angular.module('myApp',['ngRouter'])
 .config(function($routeProvider){
	//配置路由
	$routeProvider.when('/',{
		controller:'welcomeCtrl',
		templateUrl:'home/views/sidebarLI.html'
	});
})
 .config(['ConnectionProvider',function(ConnectionProvider){}]);

```
##  运行块run 
**和配置块不同，运行块在注入器创建之后被执行，它是所有AngularJS应用中第一个被执行的方法**
**运行块比较接近main函数的概念。
运行块通常用来注册全局的事件监听器**，例如设置路由事件的监听器以及过滤未经授权的请求。
例如：我们需要在每次路由发生变化时，都执行一个函数来验证用户的权限。
```

angular.module('myApp',['ngRouter']).run(function($rootScope,AuthService){
	$rootScope.$on('$routeChangeStart',function(evt,next,current){
		//如果用户为登录
		if(!AuthService.isLogged()){
			if(next.templateUrl==='login.html'){
				//已经转向登录路由则无需重定向
			}else{
				$location.path('/login');
			}
		}
	}
});

```
##  多重视图和路由 
能够从页面的一个视图跳转到另一个视图，对于单页应用来说是至关重要的。URL中包含应用的状态信息。
我们一般将视图分解成布局和模板，并根据用户当前访问的URL来展示对应的视图。
我们会将这些模板分解到视图，并在布局模板中进行组装，AngularJS允许我们在$route服务的提供者$routeProvider中通过声明路由来实现这个功能。

从1.2版本开始，AngularJS将ngRouters从核心代码中剥离出来成为独立的模块。我们需要安装并应用它。
首先下载：https://code.angularjs.org/
之后引用:
```

<script  src="angular.js"/>
<script  src="angular-route.js"/>

```
最后要把ngRoute模块在我们的应用中当作依赖加载进来：
```

 var HomeApp = angular.module('HomeApp', ["ngRoute"]);

```
###  布局模板 
通过ng-view指令和路由组合，我们可以指定当前路由对应的模板在DOM中渲染的位置。
如
```

<div ng-view></div>

```
**ng-view**是由ngRoute模块提供的一个特殊指令，它的独特作用在于HTML中的给$route对应的视图内容占位。
**它会创建自己的作用域并将模板嵌套在内部**

**ngView指令遵循以下规则：**
每次出发` $routeChangeSuccess `事件，视图都会更新。例如：
```

HomeApp.run(['$rootScope','$location',function($rootScope,$location){
        $rootScope.$on('$routeChangeSuccess',function(evt,next,prev){
            require(['UtilDir/dialog','UtilDir/grid'], function (Dialog,Grid) {
                Grid.dropCache();
                Dialog.dropCache();
            })
        });
    }]);

```
**如果某个模板同当前路由相关联：**
  * 创建一个新的作用域
  * 移除上一个视图，同时上一个作用域也会被清除，这一点需要注意
  * 将新作用域同当前模板关联在一起
  * 如果路由中有相关定义，则把对应的控制器和当前作用域关联起来
  * 触发$viewContentLoaded事件
  * 如果提供了onload属性，调用该属性所指定的函数

###  路由 
用config函数在特定的模块或应用中定义路由：
```

 var app = angular.module('HomeApp', ["ngRoute"]);
app.config(['$routeProvider',function($routeProvider) {
                        $routeProvider.when('/home', {
                            templateUrl : 'static/home.html',
                            controller : 'homeCtrl'
                        }).when('/home', {
                            templateUrl : 'static/home.html',
                            controller : 'homeCtrl'
                        }).otherwise({
                           redirectTo：'/'
                        });
                    }
                ]);

```
上面的代码很容易理解,即使不会看意思猜都能猜出来,这里介绍一些其他的.when()的第二个参数是一个配置对象,**通常我们只需要配置一下templateUrl和controller就行了**,有些时候我们需要更多.
(1) controller 
它的作用是将指定的控制器与路由所创建的新作用域关联在一起,如果参数是字符串,那么就从所有注册过的控制器中寻找对应的内容进行关联,如果参数是函数,那么就会直接关联这个函数. 
(2) template 和 templateUrl 
这两个其实都是HTML模板,区别只是一个是String形式写在route中,另一个则会通过XHR读取指定文件(或者从$templateCache).这里面的内容都会被渲染到具有ng-view指令的DOM中. 
(3) resolve 
这个感觉可能不是太常用,不过非常有用.**它通常是一个对象,key-value的形式.key是注入到控制器中的依赖名称.**如果你还不明白它做了什么,比较通俗易懂的说法就是**讲一个服务或者一个值在控制器加载以及$routeChangeSuccess触发之前,被设置并注入到控制器.**比如这样: 
```

resolve: {
  'flag': ['$someService', function($someService) {
      if($someService.getFlag()){
        return true;
      }else{
        return false;
      }
  }];
}

```
上面代码可以看到,在**flag是通过一个服务判断后设置的,它会被注入到控制器,所以你在控制器可以使用它了**.补充一点就是,这里"key-value"格式中,value可以是服务的名字或者返回值,函数或者可以被resolve的promise对象,**它会被注入到控制器**.

(4) redirectTo 
这个看名字就知道干嘛的了...不解释 。可以是一个值或一个函数
(5) reloadOnSearch 
这个参数基本上大家都没用过,因为通常都使用默认值(true).默认在$location.search()变化时会重新加载路由,如果这个参数为false,那么当URL查询条件变化时就不会重新加载路由.这个功能当你使用本地分页时对URL是更友好的.

###  路由参数 
路由里可以有参数,只要将when()的第一次参数写成类似` '/users/:user_id `'这样就行了,AngularJS会把它解析出来并传递给` $routeParams. ` 
你在从中就可以通过**$routeParams.user_id的方式获取这个值**. 
```

angular.module('phonecat', []).
  config(['$routeProvider', function($routeProvider) {
  $routeProvider.
      when('/phones', {templateUrl: 'partials/phone-list.html',   controller: PhoneListCtrl}).
      when('/phones/:phoneId', {templateUrl: 'partials/phone-detail.html', controller: PhoneDetailCtrl}).
      otherwise({redirectTo: '/phones'});
}]);

```
当URL 映射段为/phone/:phoneId时，这里:phoneId是URL的变量部分。
:phoneId参数的使用。$route服务使用路由声明/phones/:phoneId作为一个匹配当前URL的模板。所有以:符号声明的变量（此处变量为phones）都会被提取，然后存放在` $routeParams `对象中。

**对URL的控制 - $location**
一般情况下我们对地址栏中的URL的操作要通过` window.location `对象,AngularJS提供了一个服务用以解析地址栏中的URL,也就是` $location `.通过它你可以访问应用当前路径所对应的路由,以及修改路径和处理导航. 
应用需要在内部进行跳转时是使用$location进行的,**注意的是它并没有刷新整个页面的能力.如果要刷新整个页面可以使用` $window.location `对象.** 
下面简单介绍一下 常用的 API,推荐还是去看官网文档. 
(1) path() 
获取以及修改当前路径,可以和HTML5的历史API直接进行交互,所以前进后退按钮可以生效. 
 (2) replace() 
如果**希望跳转后不能后退回去,可以这么写:$location.path('/').replace()** 
(3) search()
获取或者修改URL中的查询串(也就是查询参数),设置可以是对象或者字符串均可. 
(4) url() 
和path()类似,只不过操作的是URL而不是路径. 

###  两种路由模式 
如果你看过官网的路由Demo,应该会发现在**URL中包含#符号**.之所以有这个东西是因为AngularJS一些**默认情况下的Demo都是基于路由的标签模式**.不同路由模式在浏览器地址栏中会使用不同的URL格式.**$location默认是标签模式.** 

 (1) 标签模式 
**标签模式就利用内部链接的技巧,URL路径以#符号开头**.AngularJS本身不会重写<a>标签,也不需要进行任何配置或者服务器支持.它的URL看起来类似这样: http://angular.app.com/#/users
默认的AngularJS配置就是这样的,不要任何设置.**通常标签模式是HTML5模式的一种降级方案.** 

(2) HTML5模式
HTML5模式就接近于我们正常的URL,同一个地址它看起来是这样的: http://angular.app.com/users .   在AngularJS内部,$location服务通过HTML5历史API让应用能够使用普通的URL路径来访问路由,当浏览器不支持HTML5历史API时,$location服务会自动使用标签模式的URL作为替代 方案.在HTML5模式中,AngularJS会负责重写<a href=""></a>中的链接.也就是说AngularJS会根据浏览器的能力在编译时决定是否要重写href=""中的链接.
后端服务器也需要支持URL重写,服务器需要确保所有请求都返回index.html,以支持HTML5模式.这样才能确保由AngularJS应用来处理路由.
当在HTML5模式的AngularJS中写链接时,永远都不要使用相对路径.如果你的应用是在根路径中加载的,这不会有什么问题,但如果是在其他路径中,AngularJS应用就无法正确处理路由了.

###  路由的事件 
**$route服务在路由过程中的每个阶段都会触发不同的事件,可以为这些不同的路由事件设置监听器并做出响应.**
一共有4个事件用来监听路由的状态变化:` $routeStartChange, $routeChangeSuccess, $routeChangeError, $routeUpdate. `
其中最常用的是前两个,这里稍微解释一下.
(1) $routeStartChange
看名字就能猜出来它表示的是**路由开始变化**的事件,在浏览器地址栏发生变化之前AngularJS会先广播一下这个事件.路由会开始加载所有需要的依赖,模板和resolve部分的内容也会注入.
```

angular.module('myApp', [])
  .run(['$rootScope', '$location', function($rootScope, $location){
    $rootScope.$on('$routeChangeStart', function(evt, next, current){
    console.log('route begin change');
  }); 
}]);

```
解释一下事件的参数**,evt是事件对象,可以从中读取到一些route的信息.next是将要导航到的路由,current是当前的URL.**
可以看见在这个时期我们可以做很多有用的事,因为此时仅仅是路由开始变化,对应的内容都还没来得及发生改变**.这里我们可进行permission的校验,loading画面的加载,对应路由信息的读取等等.**

(2) $routeChangeSuccess
在路由的所有依赖都被注入加载后,AngularJS会对外广播路由跳转成功的事件. 
```

angular.module('myApp', [])
  .run(['$rootScope', '$location', function($rootScope, $location) {
    $rootScope.$on('$routeChangeSuccess', function(evt, current, previous) {
      console.log('route have already changed');
    }); 
}])

```
这里也稍微解释下三个参数,**evt是AngularJS事件对象,current是当前所处路由,previous是上一个路由 .**
剩下两个不太常用的事件,大家去看官方API说明吧,这里不介绍了.

应用场景以及可扩展性
对于**SPA（Simple Page Application）应用**尤其是AngularJS这种胖客户端的应用,路由的部分一直可以大做文章的.我们可以把一些页面的基本信息配置到路由中.上面说过了when()的第二个参数是一个对象,出了AngularJS为我们定义好的一些属性之外,我们完全可以自己进行扩展,这些扩展出来的信息,又可以通过注入的方式,加载到每一个路由对应的控制器中. 
通过这样方式我们就可以解耦出来一些每个页面都有,但是能抽象成公共信息的这么一类东西.比如之前我一篇博客中提到的**通过路由控制页面访问权限**,**又或者每个页面都有一个说明当前页面主题的title导航之类的东西**,这种信息也可以淡出抽到route中.又或者在视图切换的间隙中,加入loading动画都可以在路由中实现. 
复杂的SPA通过路由进行视图和视图之间的管理和联系,路由的规范往往是牵一发而动全身的.通常文件的视图结构就映射了路由的URL结构,这些结构又在一定程度上符合restful定位资源的含义,好的路由让你可以更快速的寻找到对应的文件. 

###  关于搜索引擎索引 
web爬虫对于javascript胖客户端应用无能为力。为了在应用的运行过程中给爬虫提供支持，我们需要在头部添加meta标签。
```

<meta name="fragment" content="!"/>

```
###  页面重新加载 
$location服务并没有刷新整个页面的能力.如果要刷新整个页面可以使用` $window.location `对象.
$window.location.href="/pahe/new"

###  异步地址变化 
如果我们想要在作用域的生命周期外使用$location服务，必须用$apply函数将变化抛到应用外部。

参考：http://www.tuicool.com/articles/AvmaYn