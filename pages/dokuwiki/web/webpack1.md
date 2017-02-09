title: webpack1 

#  webpack基础 
Webpack 中文指南：http://webpackdoc.com/index.html
https://github.com/webpack/webpack

当前版本1.13.2 ，2.0版本还在beta版。

Webpack 是当下最热门的前端资源模块化管理和打包工具。
它可以将许多松散的模块按照依赖和规则打包成符合生产环境部署的前端资源。
还可以将按需加载的模块进行代码分隔，等到实际需要的时候再异步加载。
通过 loader 的转换，任何形式的资源都可以视作模块，比如 CommonJs 模块、 AMD 模块、 ES6 模块、CSS、图片、 JSON、Coffeescript、 LESS 等。

**模块系统的演进：**
* 多个script标签
* CommonJS/Browserify：适用于NodeJS服务端模块加载，是同步加载的。
* AMD/requireJS：适用于浏览器端模块加载，是异步加载的。
* ES6 模块：EcmaScript6 标准增加了 JavaScript 语言层面的模块体系定义。
（ES6 模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。但是存在的问题是原生浏览器端还没有实现该标准
import "jquery";
export function doStuff() {}
module "localModule" {}）


**期望的模块系统**
可以兼容多种模块风格，尽量可以利用已有的代码，不仅仅只是 JavaScript 模块化，还有 CSS、图片、字体等资源也需要模块化。

**前端模块加载**
前端模块要在客户端中执行，所以他们需要增量加载到浏览器中。
模块的加载和传输，我们首先能想到两种极端的方式，一种是每个模块文件都单独请求，另一种是把所有模块打包成一个文件然后只请求一次。
显而易见，每个模块都发起单独的请求造成了请求次数过多，导致应用启动速度慢；一次请求加载所有模块导致流量浪费、初始化过程慢。这两种方式都不是好的解决方案，它们过于简单粗暴。

分块传输，按需进行懒加载，在实际用到某些模块的时候再增量更新，才是较为合理的模块加载方案。
要实现模块的按需加载，就需要一个对整个代码库中的模块进行静态分析、编译打包的过程。


**所有资源都是模块**
require("./style.css");
require("./style.less");
require("./template.jade");
require("./image.png");

**Webpack 的特点**
* 代码拆分：组织**模块**依赖的方式，提供**按需加载**的能力
* Loader：Webpack 本身只能处理原生的 JavaScript 模块，但是 loader 转换器可以将各种类型的资源转换成 JavaScript 模块。这样，**任何资源都可以成为 Webpack 可以处理的模块**。
* 智能解析：Webpack 有一个智能解析器，几乎可以处理任何第三方库，无论它们的模块形式是 CommonJS、 AMD 还是普通的 JS 文件。甚至在加载依赖的时候，允许使用动态表达式 require("./templates/" + name + ".jade")。
* 插件系统：Webpack 还有一个功能丰富的插件系统。
* 快速运行：Webpack 使用异步 I/O 和多级缓存提高运行效率，这使得 Webpack 能够以令人难以置信的速度快速增量编译。

##  安装
先安装node.js
然后全局安装:
```

# 安装指定版本的 webpack
$ npm install webpack@1.13.x -g
$ npm install webpack-dev-server -g

```

初始化项目:
```

npm init  #生成package.json文件

```

本地安装webpack
```

# 查看 webpack 版本信息
$ npm info webpack
# 安装指定版本的 webpack
$ npm install webpack@1.13.x --save-dev
#安装webpack loader
 $ npm install style-loader --save-dev
 $ npm install css-loader --save-dev
 $ npm install less-loader --save-dev
 $ npm install sass-loader --save-dev
#将我们的CSS代码自动添加-webkit-等前缀，提高CSS代码的兼容性 
 $ npm install --save-dev postcss-loader autoprefixer

 $ npm install json-loader --save-dev
 $ npm install url-loader --save-dev
 $ npm install file-loader --save-dev
$ npm install raw-loader --save-dev

$ npm install vue-loader --save-dev
$ npm install vue-html-loader --save-dev

$ npm install imports-loader -save-dev
$ npm install exports-loader --save-dev

#实现对 es6 语法的编译解析
$ npm install  babel-loader --save-dev

```
插件安装：
```

$ npm install  extract-text-webpack-plugin --save-dev

```
##  WebPack的配置 
每个项目下都必须配置有一个** webpack.config.js**
```

var path = require('path');
var webpack = require('webpack')

module.exports = {
 //页面入口文件配置
  entry: './entry.js',
//入口文件输出配置
  output: {
    path: path.resolve(__dirname), //使用Webpack-dev-server时，如果路径不是当前文件夹，比如./dist,那么就必须配置publicPath:'/dist/',或者指定cli选项--content-base ./dist
    //publicPath:'/dist/',
    filename: 'bundle.js'
  },
//用于调试生成js source-map
  devtool:"cheap-module-eval-source-map",
  module: {
    //加载器配置,如上，"-loader"其实是可以省略不写的，多个loader之间用“!”连接起来。
        loaders: [
            { test: /\.css$/,exclude: /node_modules/,loader: ExtractTextPlugin.extract("style", "css!postcss")} //loader: 'style!css?sourceMap!postcss' }, //需要排除添加exclude: /node_modules/,  
            { test: /\.js$/, exclude: /node_modules/,loader: 'jsx?harmony' },
            { test: /\.scss$/,exclude: /node_modules/, loader: 'style!css!sass?sourceMap'},
 	    //图片文件使用 url-loader 来处理，小于8kb的直接转为base64
            { test: /\.(png|jpg)$/,exclude: /node_modules/, loader: 'url?limit=8192'},
           { test: /\.vue$/, exclude: /node_modules/,loader: 'vue'}

        ],
  },
  postcss: [
    require('autoprefixer')//调用autoprefixer插件
  ],
//插件项
  plugins: [
	//new webpack.BannerPlugin('This file is created by xby')
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"dev"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    }),
    new ExtractTextPlugin("[name].css"),//new ExtractTextPlugin("[name]-[hash].css")
    //为需要加载的资源生成hash
    //new HtmlWebpackPlugin(), // Generates default index.html 
    new HtmlWebpackPlugin({  // Also generate a test.html 
      filename: 'test.html',
      template: 'src/assets/test.html'
    }),
    new webpack.optimize.OccurenceOrderPlugin() //模块id优化插件
	  ],
	//其它解决方案配置,配置查找模块的路径和扩展名和别名。
  resolve: {
		//查找module的话从这里开始查找
	      root: 'E:/github/flux-example/src', //绝对路径
		//自动扩展文件后缀名，意味着我们require模块可以省略不写后缀名
	      extensions: ['', '.js', '.json', '.scss'],
		 //模块别名定义，方便后续直接引用别名，无须多写长长的地址
	      alias: {
	            AppAction : 'js/actions/AppAction.js'
	      }
	}
}

```
入口可以有多个
```

{
	entry: {
		page1: "./page1",
		//支持数组形式，将加载数组中的所有模块，但以最后一个模块作为输出
		page2: ["./entry1", "./entry2"]
	},
	output: {
		path: "dist/js/page",
		filename: "[name].bundle.js"
	}
}
//该段代码最终会生成一个 page1.bundle.js 和 page2.bundle.js，并存放到 ./dist/js/page 文件夹下。

```

更详细的配置参考官方文档

Webpack 会分析入口文件，解析包含依赖关系的各个文件。这些文件（模块）都打包到 bundle.js 。Webpack 会给每个模块分配一个唯一的 id 并通过这个 id 索引和访问模块。在页面启动时，会先执行 entry.js 中的代码，其它模块会在运行 require 的时候再执行。
entry.js 中的 style.css 加载方式：
require('./style.css')

##  模块开发 
模块开发可以使用 AMD/CMD ,commonJS ,ES6模块等方式进行。在开发新项目时还是推荐CommonJs或ES2015的Module。
下面我们就来看一下它是否真的支持CommonJs和AMD、ES6模块模块机制呢？下面我们新建多几个js文件吧！
```

// 修改module1.js
require(["./module3"], function(){
  console.log("Hello Webpack!");
});
下一个文件
// module2.js，使用的是CommonJs机制导出包
module.exports = function(a, b){
  return a + b;
}
下一个文件
// module3.js，使用AMD模块机制
define(['./module2.js'], function(sum){
  return console.log("1 + 2 = " + sum(1, 2));
})

```

##  Loader 
Loader列表：http://webpack.github.io/docs/list-of-loaders.html
Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。
  * Loader 可以通过管道方式链式调用，每个 loader 可以把资源转换成任意格式并传递给下一个 loader ，但是最后一个 loader 必须返回 JavaScript。
  * Loader 可以同步或异步执行。
  * **Loader 运行在 node.js 环境中**，所以可以做任何可能的事情。
  * Loader 可以通过**文件扩展名（或正则表达式**）绑定给不同类型的文件。

按照惯例，而非必须，loader 一般以 xxx-loader 的方式命名，xxx 代表了这个 loader 要做的转换功能，比如 json-loader。
在引用 loader 的时候可以使用全名 json-loader，或者使用短名 json。

更多参考：https://segmentfault.com/a/1190000005742111
##  插件 
插件列表：http://webpack.github.io/docs/list-of-plugins.html
插件可以完成更多 loader 不能完成的功能。
Webpack 本身内置了一些常用的插件，还可以通过 npm 安装第三方插件。
例如BannerPlugin 内置插件的作用是给输出的文件头部添加注释信息。
插件的使用一般是在 webpack 的配置信息 plugins 选项中指定。

我用到了两个： 
DefinePlugin：在全局中添加变量 
ProvidePlugin：自动require我们经常需要的包，如jQuery
```

plugins: [
        new webpack.DefinePlugin({
            __DEV__: JSON.stringify("true"),
            VERSION: '1.1',

        }),
        new webpack.ProvidePlugin({
            $: 'jquery',
            Mock: 'mockjs',
        }),
    ],

```
更多参考：https://segmentfault.com/a/1190000005106383
##  运行 webpack 
webpack 的执行也很简单，直接执行
```

$ webpack --display-error-details
其他主要的参数有：
$ webpack --config XXX.js   //使用另一份配置文件（比如webpack.config2.js）来打包
$ webpack --watch   //监听变动并自动打包
$ webpack -p    //压缩混淆脚本，这个非常非常重要！
$ webpack -d    //生成map映射文件，告知哪些模块被最终打包到哪里了

```

开发环境
如果不想每次修改模块后都重新编译，那么可以**启动监听模式**。开启监听模式后，没有变化的模块会在编译后缓存到内存中，而不会每次都被重新编译，所以监听模式的整体速度是很快的。
```

$ webpack --progress --colors --watch
#也可以通过--port来改变端口
$ webpack --progress  --colors --watch

```

当然，**使用 webpack-dev-server 开发服务**是一个更好的选择。
```

$ webpack-dev-server --colors 
$ webpack-dev-server --port 8888 --colors

``` 
它将在 localhost:8080 启动一个 express 静态资源 web 服务器，并且会以监听模式自动运行 webpack，
服务器可以自动生成和刷新，修改代码保存后自动更新画面
**http://localhost:8080/webpack-dev-server/bundle**

##  故障处理 
Webpack 的配置比较复杂，很容出现错误，下面是一些通常的故障处理手段。
通过参数** - -display-error-details** 来打印错误详情。
**Webpack 中涉及路径配置最好使用绝对路径，建议通过` var path = require('path'); ` path.resolve(dirname, "app/folder") 或 path.join(dirname, "app", "folder") 的方式来配置，以兼容 Windows 环境。**

##  CommonJS 规范 
CommonJS 规范是为了解决 JavaScript 的作用域问题而定义的模块形式，可以使每个模块它自身的命名空间中执行。
**该规范的主要内容是，模块必须通过 module.exports 导出对外的变量或接口，通过 require() 来导入其他模块的输出到当前模块作用域中。**
一个直观的例子：
```

// moduleA.js
module.exports = function( value ){
	return value * 2;
}
// moduleB.js
var multiplyBy2 = require('./moduleA');
var result = multiplyBy2(4);

```
**CommonJS 是同步加载模块**,主要用于Node服务端，但也有浏览器端的实现。

##  AMD 规范 
**AMD（异步模块定义）是为浏览器环境设计的。**
AMD 定义了一套 JavaScript 模块依赖异步加载标准，来解决同步加载的问题。
模块通过 define 函数定义在闭包中，格式如下：
```

define(id?: String, dependencies?: String[], factory: Function|Object);

```
factory 是最后一个参数，它包裹了模块的具体实现，它是一个函数或者对象。如果是函数，那么它的返回值就是模块的输出接口或值。
**如果没有指定 dependencies，那么它的默认值是 ["require", "exports", "module"]。**
```

define(function(require, exports, module) {}）

```

一些用例：
定义一个名为 myModule 的模块，它依赖 jQuery 模块：
```

define('myModule', ['jquery'], function($) {
	// $ 是 jquery 模块的输出
	$('body').text('hello world');
	 var HelloWorldize = function(selector){
		$(selector).text('hello world');
	};
	// HelloWorldize 是该模块输出的对外接口
	return HelloWorldize;
});
// 使用
define(['myModule'], function(myModule) {});

```
注意：在 webpack 中，模块名只有局部作用域，在 Require.js 中模块名是全局作用域，可以在全局引用。
**在模块定义内部引用依赖：**
```

define(function(require) {
	var $ = require('jquery');
	$('body').text('hello world');
});

```

**模块引入**
**HTML直接在页面引入 webpack 最终生成的页面脚本即可**，不用再写什么 data-main 或 seajs.use 了
JS各脚本模块可以直接使用 commonJS 或者AMD来书写，并可以直接引入未经编译的模块，比如 JSX、sass、coffee等（只要你在 webpack.config.js 里配置好了对应的加载器）。
##  Shimming模块 
**这节是针对非AMD/CMD/CommonJS标准的js需要全局变量依赖时而言的。**
```

npm install exports-loader -save-dev
npm install imports-loader -save-dev

```
在 AMD/CMD 中,我们需要对不符合规范的模块（比如一些直接返回全局变量的插件）进行 shim 处理，
**imports-loader或者使用插件ProvidePlugin来处理插件的全局变量依赖
exports-loader则用来导出模块内的变量到外面。**
参考：
https://segmentfault.com/q/1010000004633039/a-1020000004634576
http://webpack.github.io/docs/shimming-modules.html
###  ProvidePlugin-定义所有模块依赖的全局变量
Example: Make $ and jQuery available in every module without writing require("jquery").
```

new webpack.ProvidePlugin({
    $: "jquery",
    jQuery: "jquery",
    "window.jQuery": "jquery"
})

```

###  imports-loader 
imports-loader允许你require一个依赖于某个全局变量的模块，比如jquery插件依赖于全局变量$,或者某些插件this指向window
比如example.js是一个jquery插件，定义了一个$.fn.doExamplePlugin.那么我们可以这样
```

require("imports?$=jquery!./example.js");

``` 
这样就把example.js这个jquery插件引进来了，同时告诉它,$=require('jquery')。
```

$("img").doExamplePlugin();

```
常用形式:
```

  * angular	 var angular = require("angular");
  * $=jquery	var $ = require("jquery");
  * define=>false	var define = false;
  * config=>{size:50}	var config = {size:50};
  * this=>window	(function () { ... }).call(window);

```

**依赖多个全局变量时：**
```

require("imports?$=jquery,angular,config=>{size:50}!./file.js");

```

通过配置webpack.config.js方式来实现上面的同样效果：
```

// ./webpack.config.js
module.exports = {
    ...
    module: {
        loaders: [
            {
                test: require.resolve("some-module"),
                loader: "imports?this=>window"
            }
        ]
    }
};

```
###  exports-loader 
用来导出模块内的变量到外面
Examples:
**比如file.js中文件最外层定义了var XModule = ....，或者function XModule(){} ,这时我们想在require之后使用该变量，我们就必须到出它。
var XModule = require("exports?XModule!./file.js");使用就通过XModule来用，如果是函数则XModule()
到处多个变量，如 XParser, Minimizer.
var XModule = require("exports?XParser&Minimizer!./file.js"); 使用通过XModule.XParser; XModule.Minimizer,注意，这里只有一个&而不是&&**
The file sets a global variable with XModule = ....
require("imports?XModule= >undefined!exports?XModule!./file.js") (import to not leak to the global context)
The file sets a property on window window.XModule = ....
require("imports?window= >{}!exports?window.XModule!./file.js


##  自定义公共模块提取 
在文章开始我们使用了 **CommonsChunkPlugin** 插件来提取多个页面之间的公共模块，并将该模块打包为 common.js 。
但有时候我们希望能更加个性化一些，我们可以这样配置：
```

var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
	entry: {
		p1: "./page1",
		p2: "./page2",
		p3: "./page3",
		ap1: "./admin/page1",
		ap2: "./admin/page2"
	},
	output: {
		filename: "[name].js"
	},
	plugins: [
		new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
		new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
	]
};
// <script>s required:
// page1.html: commons.js, p1.js
// page2.html: commons.js, p2.js
// page3.html: p3.js
// admin-page1.html: commons.js, admin-commons.js, ap1.js
// admin-page2.html: commons.js, admin-commons.js, ap2.js

```

##  独立打包样式文件 
有时候可能希望项目的样式能不要被打包到脚本中，而是独立出来作为.css，然后在页面中以<link>标签引入。这时候我们需要 extract-text-webpack-plugin 来帮忙：
```

var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
	module: {
		loaders: [
			{ test: /\.css$/, loader: ExtractTextPlugin.extract("style", "css?sourceMap") },
		{test: /\.less$/i, loader: extractLESS.extract(["style",'css?sourceMap','less'])}
		]
	},
	plugins: [
		new ExtractTextPlugin("[name].css")
	]
}

```
最终 webpack 执行后会乖乖地把样式文件提取出来.

##  使用CDN/远程文件 
有时候我们希望某些模块走CDN并以<script>的形式挂载到页面上来加载，但又希望能在 webpack 的模块中使用上。
这时候我们可以在配置文件里使用 externals 属性来帮忙：
```

{
	externals: {
		// require("jquery") 是引用自外部模块的
		// 对应全局变量 jQuery
		"jquery": "jQuery"
	}
}

```
需要留意的是，得确保 CDN 文件必须在 webpack 打包文件引入之前先引入。

##  与 grunt/gulp 配合 
以 gulp 为示例，我们可以这样混搭：
```

gulp.task("webpack", function(callback) {
	// run webpack
	webpack({
		// configuration
	}, function(err, stats) {
		if(err) throw new gutil.PluginError("webpack", err);
		gutil.log("[webpack]", stats.toString({
			// output options
		}));
		callback();
	});
});

```
当然我们只需要把配置写到 webpack({ .. . }) 中去即可，无须再写 webpack.config.js 了。

##  webpack调试 
https://segmentfault.com/a/1190000004280859
http://stackoverflow.com/questions/32211649/debugging-with-webpack-es6-and-babel
http://www.jianshu.com/p/c0492554b33c

js调试，即生成source-map.结合chrome devtools调试
sourcemap 有 7 种，**cheap-module-eval-source-map** 绝大多数情况下都会是最好的选择
添加选项：
` devtool:"cheap-module-eval-source-map", `

##  webpack-dev-server CLI选项 
http://gold.xitu.io/entry/57b94d8879bc44005ba13b4c
http://webpack.github.io/docs/webpack-dev-server.html#webpack-dev-server-cli
问题：
webpack-dev-server好像是只监听webpack.config.js中entry入口下文件（如js、css等等）的变动，只有这些文件的变动才会触发实时编译打包与页面刷新，而对于不在entry入口下的html文件，却不进行监听与页面自动刷新。
请问该如何才能使html的修改也能触发页面的自动刷新呢？
https://segmentfault.com/q/1010000004707022/a-1020000004710399
https://github.com/AriaFallah/WebpackTutorial/tree/master/part1/html-reload
参考：
Webpack 中文指南：http://webpackdoc.com/index.html
http://www.jianshu.com/p/b95bbcfc590d
http://www.w2bc.com/Article/50764

在package.json中配置scripts选项
```

"scripts": {
    "dev": "webpack-dev-server --devtool eval-source-map --progress --colors --hot --inline --content-base ./dist",
    "build": "webpack --progress --colors"
  },

``` 
然后通过 **npm run dev**来运行。

##  彻底解决 webpack 打包文件体积过大 
http://www.jianshu.com/p/a64735eb0e2b
webpack 把我们所有的文件都打包成一个 JS 文件，这样即使你是小项目，打包后的文件也会非常大。

**去除不必要的插件**
刚开始用 webpack 的时候，开发环境和生产环境用的是同一个 webpack 配置文件，导致生产环境打包的 JS 文件包含了一大堆没必要的插件，比如 HotModuleReplacementPlugin, NoErrorsPlugin... 这时候不管用什么优化方式，都没多大效果。所以，如果你打包后的文件非常大的话，先检查下是不是包含了这些插件。

###  提取第三方库 
像 react 这个库的核心代码就有 627 KB，这样和我们的源代码放在一起打包，体积肯定会很大。所以可以在 webpack 中设置
```

{
  entry: {
   bundle: 'app'
   vendor: ['react']
  }

  plugins: {
    new webpack.optimize.CommonsChunkPlugin('vendor',  'vendor.js')
  }
}

```
这样打包之后就会多出一个 vendor.js 文件，之后在引入我们自己的代码之前，都要先引入这个文件。比如在 index.html 中
 <script src="/build/vendor.js"></script>
 <script src="/build/bundle.js"></script>

**除了这种方式之外，还可以通过引用外部文件的方式引入第三方库**，比如像下面的配置
```

{
  externals: {
     'react': 'React'
  }
}

```
externals 对象的 key 是给 require 时用的，比如 require('react')，**对象的 value 表示的是如何在 global 中访问到该对象，这里是 window.React**。这时候 index.html 就变成下面这样
```

<script src="//cdn.bootcss.com/react/0.14.7/react.min.js"></script>
<script src="/build/bundle.js"></script>

```

###  代码压缩 
webpack 自带了一个压缩插件 **UglifyJsPlugin**，只需要在配置文件中引入即可。**它既可以压缩js代码也可以压缩css代码。**
```

{
  plugins: [
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
}

```
加入了这个插件之后，编译的速度会明显变慢，所以一般只在生产环境启用。
另外，服务器端还可以开启 gzip 压缩，优化的效果更明显。

##  Webpack + vue-loader构建单文件vue组件 
http://blog.csdn.net/a1104258464/article/details/51984721
http://www.jianshu.com/p/a5361bff1cd8

示例项目：https://github.com/vuejs/vue-loader-example

使用vue-loader并进行相关配置，略。请看前面。还需要一个vue-html-loader
配置es6模块语法支持babel-loader处理js
```

module: {
    loaders: [
      {
        // 让webpack去验证文件是否是.js结尾将其转换
        test: /\.js$/,
        // 通过babel转换
        loader: 'babel',
        // 不用转换的node_modules文件夹
        exclude: /node_modules/
      }
    ],  
  babel: {  
    presets: ['es2015', 'stage-0'],  
    plugins: ['transform-runtime']  
  } 
  }

```

在Vue根目录下创建index.html：
```

<html>  
<head>  
    <meta charset="UTF-8">  
    <title>Document</title>  
</head>  
<body>  
    <App></App>   
    <script src="dist/build.js"></script>  
</body>  
</html>

```
在src目录下创建main.js文件：
```

//es6加载模块 ,需要babbel-loader支持
import Vue from 'vue'  
import App from './views/index.vue' 

//require('vue')
//require('./views/index.vue')
  
//创建一个vue实例,挂载在body上面  
new Vue({   
    el: 'body',  
    components: { App }  
})

```
在src/views目录下创建index.vue:
```

<pre name="code" class="javascript"><template>  
    <p>![](/data/dokuwiki message )</p>  
    <input v-model="message">  
</template>  
  
  
<script> 
  //module.exports={ 
  //es6使用 
  export default{  }
      
        data(){   
            return {  
                message: 'Hello Vue.js!'  
            }  
        }  
    }  
</script>

```
  
##  配置package.json npm命令 
package.json中配置scripts选项
```

  "scripts": {
    "dev": "webpack-dev-server --progress --colors --inline --hot --port 8888 --content-base .",
    "build": "webpack --progress -p -d"
  },

```
    
##  分不同环境配置webpack思路 
先配置一个webpack.base.config,这个作为dev与product环境的基础，例如：
```

var webpack = require('webpack')
var path = require('path');
module.exports = {
  entry: './src/main.js',
  output: {
    path: './dist', // path: path.resolve(__dirname),
    publicPath: 'dist/',
    filename: 'build.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css'}
    ]
  }
}

```
然后配置webpack.dev.config,对应开发环境
```

var config = require('./webpack.base.config')
config.devtool = 'cheap-module-eval-source-map'
module.exports = config

```

配置webpack.product.config，对应生产环境
```

var webpack = require('webpack')
var config = require('./webpack.base.config')
var config = require('./webpack.base.config')
//此处先判断是否为null，然后用concat拼接数组，为什么不用push? , push 与 concat 区别:push()方法会修改原有数组，这就会影响其他环境。concat不会改变现有的数组，而仅仅会返回被连接数组的一个副本。
config.plugins = (config.plugins || []).concat([
  // this allows uglify to strip all warnings
  // from Vue.js source code.
  new webpack.DefinePlugin({
    'process.env': {
      NODE_ENV: '"production"'
    }
  }),
  // This minifies not only JavaScript, but also
  // the templates (with html-minifier) and CSS (with cssnano)!
  new webpack.optimize.UglifyJsPlugin({
    compress: {
      warnings: false
    }
  })
])

```


然后配置package.json的script，这样就可以使用npm run [环境标志]运行不同命令了。
```

"scripts": {
    "dev": "webpack-dev-server --inline --hot --config build/webpack.dev.config.js",
    "build": "webpack --progress --hide-modules --config build/webpack.product.config.js",
  },

```