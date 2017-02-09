title: 深入理解requirejs 

#  深入理解RequireJS 
之前有篇[[web:js模块化requirejs]]
RequireJs已经流行很久了，我们在项目中也打算使用它。它提供了以下功能：
  * 声明不同js文件之间的依赖
  * 可以按需、并行、延时载入js库
  * 可以让我们的代码以模块化的方式组织


` 判断一个第三方库是否直接支持RequireJS模块加载，采用的办法就是查看其代码是否支持AMD规范. `
` RequireJS加载模块或这模块依赖时，会像将该模块代码执行一般，并将返回值保存为变量待以后使用。 `

**总结注意事项（重要**
  * 1、使用RequireJS配置第三方库时需要查看其代码` 是否遵循AMD规范 `，如果遵循` 是否内部给本模块命名 `了。然后再配置。
  * 2、jquery遵循AMD规范，其内部进行` 了命名"jquery" `,所以配置时使用jquery这个名字。
  * 3、jquery插件不一定遵循AMD规范。
  * 4、在全局配置js中通常命名为config.js**中配置requireJS并加载主模块。**

也就是代码中是否有如下定义了
```

define( 可选的名称, [依赖数组], function () { } );
或者
if ( typeof define === "function" && define.amd ) {
 define( "", [], function() {
  return ;
 });
}

```

##  在html中引入requirejs 

在HTML中，添加这样的 <script> 标签：
```

<script src="/path/to/require.js" data-main="/path/to/app/config.js"></script>

```
通常使用requirejs的话，我们只需要导入requirejs即可，不需要显式导入其它的js库，因为这个工作会交给requirejs来做。
属性** data-main 是告诉requirejs：你下载完以后，马上去载入真正的入口文件。****它一般用来对requirejs进行配置，并且载入真正的程序模块。**
###  在config.js中配置requirejs 
config.js 中通常用来` 做两件事 `：
  * 配置requirejs 比如项目中**用到哪些模块，文件路径是什么**
  * **载入程序主模块**
```

requirejs.config({
  baseUrl: '/public/js',
  paths: {
    app: 'app'
  }
});
requirejs(['app'], function(app) {
  app.hello();
});

```
app.js 的内容，应该使用requirejs的方式即AMD的方式来定义模块：
```

define([], function() {
  return {
    hello: function() {
      alert("hello, app~");
    }
  }
});

```

这里的 define 是requirejs提供的函数。**requirejs一共提供了两个全局变量**：
  * ` require `: 用来配置requirejs及**载入入口模块**。如果其中一个命名被其它库使用了，我们可以用另一个
  * ` define `: **定义一个模块**

##  AMD规范返回值的几种写法 
第一种，暴露变量或函数引用给reuqireJS供后续调用：
```

define([], function() {
  return {
    hello: function() {
      alert("hello, app~");
    }
  }
});

```
```

define([], function() {
  function initMy(){
    ...
  }
 function init2(){
   
 }
  return {       //return会将暴露的两个函数引用返回以便后续调用。
    initMy:initMy,
    init2:init2
  }
});

```
第二种 直接返回一个函数调用，并在该函数调用中进行初始化
```

define([], function() {
  function init(){
    ...
  }
  return function() { //return会将这个匿名函数执行完成之后再返回。而这个匿名函数中进行了初始化调用init();
	   init();
	}
｝

```
第三种：直接返回一个引用
```

define([], function() {
  function a(){
    ...
  }
  return a //return只是返回这个函数引用，以便后续被调用。
｝

```
##  以jQuery为例进行分析 
没有requireJS框架之前，如果我们想使用jquery框架，会在HTML页面中通过<script>标签加载，这个时候**jquery框架生成` 全局变量$和jQuery等全局变量 `**。
现在问题来了**，虽然jquery框架已经开始支持AMD规范，但是jquery的众多插件还是不支持AMD**，仍然像以前一样需要使用全局变量$。jquery插件大多都是如下结构：
```

(function( $, undefined ) {

})( jQuery );

```
如果我们项目中使用了jquery插件，但是jquery框架是通过requireJS加载的（不会添加全局变量$），那怎么完成jquery插件的加载呢？使用传统的方，在HTML页面中通过<script>加载jquery插件，肯定是不行的。
**这个时候我们需要使用到requireJS的` shim参数 `，来完成jquery插件的加载。**下面我们以加载jquery-ui的slider插件为例：
```

requirejs.config({
    paths : {
    jquery : 'jquery-2.1.1/jquery',
    domReady : 'require-2.1.11/domReady',
    'jquery.ui.core' : 'jquery-ui-1.10.4/development-bundle/ui/jquery.ui.core',
    'jquery.ui.widget' : 'jquery-ui-1.10.4/development-bundle/ui/jquery.ui.widget',
    'jquery.ui.mouse' : 'jquery-ui-1.10.4/development-bundle/ui/jquery.ui.mouse',
    'jquery.ui.slider' : 'jquery-ui-1.10.4/development-bundle/ui/jquery.ui.slider'
  }，
  shim: {
  'jquery.ui.core': ['jquery'],
  'jquery.ui.widget': ['jquery'],
  'jquery.ui.mouse': ['jquery'],
  'jquery.ui.slider':['jquery']
    }
});

require([ 'jquery', 'domReady','jquery.ui.core','jquery.ui.widget','jquery.ui.mouse','jquery.ui.slider'],
    function($) {
      
      $("#slider" ).slider({
           value:0,
           min: 0,
           max: 4,
           step: 1,
           slide: function( event, ui ) {}	   
      });		
                
    });

```
在` path参数 `中，我们设置了模块名称(可以随意指定)和js文件路径的映射**，然后在shim参数中，指定了模块名称和它的依赖数组，**上面我们的jquery插件只依赖于jquery框架。通过这种方式，就可以使用requireJS完成jquery和其插件的加载，不会有全局变量污染问题。


注意：**shim参数**的` exports属性 `可以把某个` **非requirejs方式的代码中的某一个全局变量或全局函数暴露出去** `，当作该模块以引用。` 而不是简单的对模块名重命名。 `
正确理解，请看下面:
前面的代码是理想的情况，即依赖的js文件，里面用了 define(...) 这样的方式来组织代码的。如果没用这种方式，会出现什么情况？
比如这个 hello.js :
```

function hello() {
  alert("hello, world~");
}

```
它就按最普通的方式定义了一个函数，我们能在requirejs里使用它吗？
在这种情况下，我们要使用 shim ，将` **某个依赖中的某个全局变量暴露给requirejs** `，**当作这个模块本身的引用。**(注意理解这句话)
```

requirejs.config({
  baseUrl: '/public/js',
  paths: {
    hello: 'hello'
  },
  shim: {
    hello: { exports: 'hello' }
  }
});

requirejs(['hello'], function(hello) {
  hello();
});

```
再运行就正常了。

上面代码 **exports: 'hello' 中的 hello ，是我们在 hello.js 中定义的 hello 函数。**当我们使用 function hello() {} 的方式定义一个函数的时候，它就是全局可用的。如果我们**选择了把它 export 给requirejs**，那当我们的代码依赖于 hello 模块的时候，就可以拿到这个 hello 函数的引用了。
所以：** exports 可以把某个非requirejs方式的代码中的某一个全局变量暴露出去（一般为这个文件中的初始化函数），当作该模块以引用。**

##  非标准js库暴露多个变量：init 
但如果我要同时暴露多个全局变量呢？比如， hello.js 的定义其实是这样的：
```

function hello() {
  alert("hello, world~");
}
function hello2() {
  alert("hello, world, again~");
}

```
它定义了两个函数，而我两个都想要。
这时就不能再用 exports 了，必须换成`  init 函数： `
```

requirejs.config({
  baseUrl: '/public/js',
  paths: {
    hello: 'hello'
  },
  shim: {
    hello: {
      init: function() {
        return {
          hello: hello,
          hello2: hello2
        }
      }
    }
  }
});

requirejs(['hello'], function(hello) {
  hello.hello1();
  hello.hello2();
});

```
**当 exports 与 init 同时存在的时候， exports 将被忽略。**

##  无主的与有主的模块 
我遇到了一个折腾我不少时间的问题：**为什么我` 只能使用 jquery名字 来依赖jquery `, 而不能用其它的名字？**
比如下面这段代码：
```

requirejs.config({
  baseUrl: '/public/js',
  paths: {
    myjquery: 'lib/jquery/jquery'
  }
});

requirejs(['myjquery'], function(jq) {
  alert(jq);
});
它会提示我：

jq is undefined
但我仅仅改个名字：

requirejs.config({
  baseUrl: '/public/js',
  paths: {
    jquery: 'lib/jquery/jquery'
  }
});

requirejs(['jquery'], function(jq) {
  alert(jq);
});

```
就一切正常了，能打印出 jq 相应的对象了。
为什么？我始终没搞清楚问题在哪儿。
###  有主的模块（内部定义了名字的模块） 
经常研究，发现原来在jquery中已经定义了：
```

define('jquery', [], function() { ... });

```
它这里的 define 跟我们前面看到的 app.js 不同，` 在于它多了第一个参数 'jquery' ，表示给当前这个模块起了名字 jquery ，它已经是有主的了，只能属于 jquery . `
所以当我们使用另一个名字：
myjquery: 'lib/jquery/jquery'
去引用这个库的时候，它会发现，**在 jquery.js 里声明的模块名 jquery 与我自己使用的模块名 myjquery 不能，便不会把它赋给 myjquery ，所以 myjquery 的值是 undefined 。**
<note tip>所以我们在使用一个第三方的时候，一定要注意它` 是否声明了一个确定的模块名 `。</note>
###  无主的模块（内部没有定义名字的模块） 
如果我们不指明模块名，就像这样：
```

define([...], function() {
  ...
});

```
那么它就是无主的模块。我们可以在 requirejs.config 里，使用任意一个模块名来引用它。这样的话，就让我们的命名非常自由，大部分的模块就是无主的。
###  为什么有的有主，有的无主 
可以看到，无主的模块使用起来非常自由，为什么某些库（jquery, underscore）要把自己声明为有主的呢？
按某些说法，这么做是出于性能的考虑。因为像 jquery , underscore 这样的基础库，经常被其它的库依赖。如果声明为无主的，那么其它的库很可能起不同的模块名，这样当我们使用它们时，就可能会多次载入jquery/underscore。
而把它们声明为有主的，那么所有的模块只能使用同一个名字引用它们，这样系统就只会载入它们一次。
##  如何完全不让jquery污染全局的$ 
在前面引用jquery的这几种方式中，我们虽然可以以模块的方式拿到jquery模块的引用，**但是还是可以在任何地方使用全局变量 jQuery 和 $** 。有没有办法让jquery完全不污染这两个变量？
如果我们有办法能让在继续使用 jquery 这个模块名的同时，有机会调用 jQuery.noConflict(true) 就好了。
我们可以再定义一个模块，仅仅为了执行这句代码：
jquery-private.js
```

define(['jquery'], function(jq) {
  return jQuery.noConflict(true);
});


```
我们这时可以引入 map 配置，一劳永逸地解决这样问题：
```

requirejs.config({
  baseUrl: '/public/js',
  paths: {
    jquery: 'lib/jquery/jquery',
    'jquery-private': 'jquery-private'
  },
  map: {
    '*': { 'jquery': 'jquery-private'},
    'jquery-private': { 'jquery': 'jquery'}
  }
});

requirejs(['jquery'], function(jq) {
  alert($);
});

```
这样做，就解决了前面的问题：**在除了jquery-private之外的任何依赖中，还可以直接使用 jqurey 这个模块名，并且总是被替换为对 jquery-private 的依赖，使得jQuery.noConflict(true);最先被执行。**

##  RequireJS下setInterval与setTimeout失效问题 
参考：http://blog.csdn.net/aitangyong/article/details/40865703
javascript中与定时相关的API有setTimeout()和setInterval()，这2个函数功能不同，但是使用方式是一样的。
以前我写javascript，都是使用setTimeout("say('aty');",1000);这种方式，由于say是全局函数，所以这样写能够正确运行。最近一个项目使用了requireJS框架，这要求我们要用模块化的方式编写javascript。用之前的方式，使用setTimeout就行不通了。
```

(function(){  
    function say(msg)  
    {  
        alert(msg);  
    }        
    // 第1种方式  
    //setTimeout("say('aty');",1000);        
    // 第2种方式  
    //setTimeout(say("aty"),1000);       
    // 第3种方式  
    setInterval(function(){  
        say("aty");  
    },1000);      
})();  

```
第一种方式会报错，因为say函数仅仅在模块内部可见，setTimeout看不见；
第二种方式：代码会立即执行，没有到达setTimeout的延时效果；
第三种方式：通过这种匿名函数调用，能够满足我们的需要，即解决了延时的问题，也解决了变量可见域的问题。

下面是本人的解决思路:
```

<script>
/**解决模版编辑时间
 * 此处没有采用RequireJS的原因是因为在RequireJS下存在setInterval()与setTimeout()函数均失效问题。
 */
function sendEmptyRequest()
{
	var url=getServer()+"/sword/cms/template/empty";
	var xhr = new XMLHttpRequest();
	// 发送POST请求
	xhr.open("GET", url, true);
	// 发送请求,，不发送任何参数
	xhr.send(null);
	setTimeout("sendEmptyRequest()" , 3000);
	console.log("haha");
}
sendEmptyRequest();
</script>

```

##  总结注意事项（重要)
  * 1、使用RequireJS配置第三方库时需要查看其代码是否遵循AMD规范，如果遵循是否内部给本模块命名了。然后再配置。
  * 2、jquery遵循AMD规范，其内部进行了命名"jquery",所以配置时使用jquery这个名字。
  * 3、jquery插件不一定遵循AMD规范。
  * 4、在全局配置js中通常命名为config.js中配置requireJS并加载主模块。


参考：http://www.tuicool.com/articles/vMZBnyr
http://www.tuicool.com/articles/jam2Anv
http://www.jb51.net/article/50724.htm