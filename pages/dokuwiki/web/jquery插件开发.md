title: jquery插件开发 

#  jQuery插件开发 
参考：http://www.cnblogs.com/Wayou/p/jquery_plugin_tutorial.html
示例：https://github.com/wayou/SlipHover/blob/master/src/jquery.sliphover.js
学会使用jQuery并不难，因为它简单易学，并且相信你接触jQuery后肯定也使用或熟悉了不少其插件。如果要将能力上升一个台阶，编写一个属于自己的插件是个不错的选择。
##  jQuery插件开发模式 
软件开发过程中是需要一定的设计模式来指导开发的，有了模式，我们就能更好地组织我们的代码，并且从这些前人总结出来的模式中学到很多好的实践。
根据《jQuery高级编程》的描述，jQuery插件开发方式主要有三种：
  * 通过$.extend()来扩展jQuery
  * 通过$.fn 向jQuery添加新的方法
  * 通过$.widget()应用jQuery UI的部件工厂方式创建
通常我们使用第二种方法来进行简单插件开发，说简单是相对于第三种方式。第三种方式是用来开发更高级jQuery部件的，该模式开发出来的部件带有很多jQuery内建的特性，比如插件的状态信息自动保存，各种关于插件的常用方法等
而第一种方式又太简单，仅仅是在jQuery命名空间或者理解成jQuery身上添加了一个静态方法而以。所以我们调用通过$.extend()添加的函数时直接通过$符号调用（$.myfunction()）而不需要选中DOM元素($('#example').myfunction())。请看下面的例子。
```

$.extend({
    sayHello: function(name) {
        console.log('Hello,' + (name ? name : 'Dude') + '!');
    }
})
$.sayHello(); //调用
$.sayHello('Wayou'); //带参调用

```
**这种方式用来定义一些辅助方法是比较方便的**。比如一个自定义的console，输出特定格式的信息，定义一次后可以通过jQuery在程序中任何需要的地方调用它。
```

$.extend({
    log: function(message) {
        var now = new Date(),
            y = now.getFullYear(),
            m = now.getMonth() + 1, //！JavaScript中月分是从0开始的
            d = now.getDate(),
            h = now.getHours(),
            min = now.getMinutes(),
            s = now.getSeconds(),
            time = y + '/' + m + '/' + d + ' ' + h + ':' + min + ':' + s;
        console.log(time + ' My App: ' + message);
    }
})
$.log('initializing...'); //调用

```
要处理DOM元素以及将插件更好地运用于所选择的元素身上，还是需要使用第二种开发方式。你所见到或使用的插件也大多是通过此种方式开发。
##  插件开发 
下面我们就来看第二种方式的jQuery插件开发。
```

$.fn.pluginName = function() {
    //your code goes here
}

```
基本上就是往$.fn上面添加一个方法，名字是我们的插件名称。然后我们的插件代码在这个方法里面展开。
比如我们将页面上所有链接颜色转成红色，则可以这样写这个插件：
```

$.fn.myPlugin = function() {
    //在这里面,this指的是用jQuery选中的元素
    //example :$('a'),则this=$('a')
    this.css('color', 'red');
}

```
在插件名字定义的这个函数内部，this指代的是我们在调用该插件时，用jQuery选择器选中的元素，**一般是一个jQuery类型的集合。比如$('a')返回的是页面上所有a标签的集合，且这个集合已经是jQuery包装类型了，也就是说，在对其进行操作的时候可以直接调用jQuery的其他方法而不需要再用美元符号来包装一下。**
所以在上面插件代码中，我们在this身上调用jQuery的css()方法，也就相当于在调用 $('a').css()。
` 理解this在这个地方的含义很重要。 `这样你才知道为什么可以直接商用jQuery方法同时在其他地方this指代不同时我们又需要用jQuery重新包装才能调用。
```

<ul>
	<li>
		<a href="http://www.webo.com/liuwayong">我的微博</a>
	</li>
	<li>
		<a href="http://http://www.cnblogs.com/Wayou/">我的博客</a>
	</li>
	<li>
		<a href="http://wayouliu.duapp.com/">我的小站</a>
	</li>
</ul>

<p>这是p标签不是a标签，我不会受影响</p>
<script src="jquery-1.11.0.min.js"></script>
<script src="jquery.myplugin.js"></script>
<script type="text/javascript">
	$(function(){
		$('a').myPlugin();
	})
</script>

```
下面进一步，在插件代码里处理每个具体的元素，而不是对一个集合进行处理，这样我们就**可以针对每个元素进行相应操作。**
我们已经知道**this指代jQuery选择器返回的集合**，那么通过调用**jQuery的.each()方法就可以处理合集中的每个元素了**，但此刻要` 注意的是，在each方法内部，this指带的是普通的DOM元素了 `，如果需要调用jQuery的方法那就` 需要用$来重新包装 `一下。
比如现在我们要在每个链接显示链接的真实地址，首先通过each遍历所有a标签，然后获取href属性的值再加到链接文本后面。
更改后我们的插件代码为：
```

$.fn.myPlugin = function() {
    //在这里面,this指的是用jQuery选中的元素
    this.css('color', 'red');
    this.each(function() {
        //对每个元素进行操作
        $(this).append(' ' + $(this).attr('href'));
    }))
}

```
<note important>注意理解上面三个this的区别</note>
调用代码还是一样的，我们通过选中页面所有的a标签来调用这个插件
###  支持链式调用 
我们都知道jQuery一个时常优雅的特性是支持链式调用。要让插件不打破这种链式调用，**只需return一下即可**。
```

$.fn.myPlugin = function() {
    //在这里面,this指的是用jQuery选中的元素
    this.css('color', 'red');
    return this.each(function() {
        //对每个元素进行操作
        $(this).append(' ' + $(this).attr('href'));
    }))
}

```
###  让插件接收参数 
一个强劲的插件是可以让使用者**随意定制的**，这要求我们提供在编写插件时就要考虑得全面些，**尽量提供合适的参数**。
比如现在我们不想让链接只变成红色，我们让插件的使用者自己定义显示什么颜色，要做到这一点很方便，只需要使用者在调用的时候传入一个参数即可。同时我们在插件的代码里面接收。另一方面，为了灵活，使用者可以不传递参数，**插件里面会给出参数的默认值**。

` 在处理插件参数的接收上，通常使用jQuery的extend方法， `上面也提到过，但那是给extend方法传递单个对象的情况下，这个对象会合并到jQuery身上，所以我们就可以在jQuery身上调用新合并对象里包含的方法了，像上面的例子。当给extend方法传递一个以上的参数时，它会将所有参数对象合并到第一个里。同时，如果对象中有同名属性时，**合并的时候后面的会覆盖前面的。**

利用这一点，我们可以在插件里定义一个保存插件参数默认值的对象，同时将接收来的参数对象合并到默认对象上，最后就实现了用户指定了值的参数使用指定的值，未指定的参数使用插件默认值。
为了演示方便，再指定一个参数fontSize，允许调用插件的时候设置字体大小。
```

$.fn.myPlugin = function(options) {
    var defaults = {
        'color': 'red',
        'fontSize': '12px'
    };
    var settings = $.extend(defaults, options);
    return this.css({
        'color': settings.color,
        'fontSize': settings.fontSize
    });
}

```
现在，我们调用的时候指定颜色，字体大小未指定，会运用插件里的默认值12px。
```

$('a').myPlugin({
    'color': '#2C9929'
});

```
###  保护好默认参数 
注意到上面代码调用extend时会将defaults的值改变，这样不好，**因为它作为插件因有的一些东西应该维持原样，另外就是如果你在后续代码中还要使用这些默认值的话**，当你再次访问它时它已经被用户传进来的参数更改了。
一个好的做法是**将一个新的空对象做为$.extend的第一个参数`  $.extend({},defaults, options); `**，defaults和用户传递的参数对象紧随其后，这样做的好处是所有值被合并到这个空对象上，保护了插件里面的默认值。
```

$.fn.myPlugin = function(options) {
    var defaults = {
        'color': 'red',
        'fontSize': '12px'
    };
    var settings = $.extend({},defaults, options);//将一个空对象做为第一个参数
    return this.css({
        'color': settings.color,
        'fontSize': settings.fontSize
    });
}

```
所以将插件的所有方法属性包装到一个对象上，用面向对象的思维来进行开发，无疑会使工作轻松很多。
##  面向对象的插件开发 
为什么要有面向对象的思维，因为如果不这样，你可能需要一个方法的时候就去定义一个function，当需要另外一个方法的时候，再去随便定义一个function，同样，需要一个变量的时候，毫无规则地定义一些散落在代码各处的变量。
```

//定义Beautifier的构造函数
var Beautifier = function(ele, opt) {
    this.$element = ele,
    this.defaults = {
        'color': 'red',
        'fontSize': '12px',
        'textDecoration':'none'
    },
    this.options = $.extend({}, this.defaults, opt)
}
//定义Beautifier的方法
Beautifier.prototype = {
    beautify: function() {
        return this.$element.css({
            'color': this.options.color,
            'fontSize': this.options.fontSize,
            'textDecoration': this.options.textDecoration
        });
    }
}
//在插件中使用Beautifier对象
$.fn.myPlugin = function(options) {
    //创建Beautifier的实体
    var beautifier = new Beautifier(this, options);
    //调用其方法
    return beautifier.beautify();
}

```
通过上面这样一改造，我们的代码变得更面向对象了，也更好维护和理解，以后要加新功能新方法，只需向对象添加新变量及方法即可，然后在插件里实例化后即可调用新添加的东西。
插件的调用还是一样的，我们对代码的改动并不影响插件其他地方，只是将代码的组织结构改动了而以。
```

$(function() {
    $('a').myPlugin({
        'color': '#2C9929',
        'fontSize': '20px'
    });
})

```
##  关于命名空间与自调用匿名函数 
不仅仅是jQuery插件的开发，我们在写任何JS代码时都应该注意的一点是**不要污染全局命名空间**。
一个好的做法是始终**用` 自调用匿名函数 `包裹你的代码**，这样就可以完全放心，安全地将它用于任何地方了，绝对没有冲突。
**但函数却可以形成一个作用域，域内的代码是无法被外界访问的。如果我们将自己的代码放入一个函数中，那么就不会污染全局命名空间，同时不会和别的代码冲突。**
如上面我们定义了一个Beautifier全局变量，它会被附到全局的window对象上，为了防止这种事情发生，你或许会说，把所有代码放到jQuery的插件定义代码里面去啊，也就是放到$.fn.myPlugin里面。这样做倒也是种选择。但会让我们实际跟插件定义有关的代码变得臃肿，而在$.fn.myPlugin里面我们其实应该更专注于插件的调用，以及如何与jQuery互动。
所以保持原来的代码不变，**我们将所有代码用自调用匿名函数包裹。**
```

(function() {
    //定义Beautifier的构造函数
    var Beautifier = function(ele, opt) {
        this.$element = ele,
        this.defaults = {
            'color': 'red',
            'fontSize': '12px',
            'textDecoration': 'none'
        },
        this.options = $.extend({}, this.defaults, opt)
    }
    //定义Beautifier的方法
    Beautifier.prototype = {
        beautify: function() {
            return this.$element.css({
                'color': this.options.color,
                'fontSize': this.options.fontSize,
                'textDecoration': this.options.textDecoration
            });
        }
    }
    //在插件中使用Beautifier对象
    $.fn.myPlugin = function(options) {
        //创建Beautifier的实体
        var beautifier = new Beautifier(this, options);
        //调用其方法
        return beautifier.beautify();
    }
})();

```
另外还有一个好处就是，` 自调用匿名函数里面的代码会在第一时间执行 `，页面准备好过后，上面的代码就将插件准备好了，以方便在后面的代码中使用插件。
目前为止似乎接近完美了。如果再考虑到其他一些因素，**比如我们将这段代码放到页面后，前面别人写的代码没有用分号结尾，或者前面的代码将window, undefined等这些系统变量或者关键字修改掉了，正好我们又在自己的代码里面进行了使用，那结果也是不可预测的，**这不是 我们想要的。我知道其实你还没太明白，下面详细介绍。

来看下面的代码，你猜他会出现什么结果？
```

var foo=function(){
    //别人的代码
}//注意这里没有用分号结尾

//开始我们的代码。。。
(function(){
    //我们的代码。。
    alert('Hello!');
})();

```
 
本来别人的代码也正常工作，**只是最后定义的那个函数没有用分号结尾而以，然后当页面中引入我们的插件时，报错了，我们的代码无法正常执行**。
所以**好的做法是我们在代码开头加一个分号，这在任何时候都是一个好的习惯。**
```

var foo=function(){
    //别人的代码
}//注意这里没有用分号结尾

//开始我们的代码。。。
;(function(){
    //我们的代码。。
    alert('Hello!');
})();

```
同时，**将系统变量以参数形式传递到插件内部也是个不错的实践。**
当我们这样做之后，window等系统变量在插件内部就有了一个局部的引用，可以提高访问速度，会有些许性能的提升
最后我们得到一个非常安全结构良好的代码：
```

;(function($,window,document,undefined){
    //我们的代码。。
    //blah blah blah...
})(jQuery,window,document);

```
 
而至于这个undefined，稍微有意思一点，为了得到没有被修改的undefined，我们并没有传递这个参数，但却在接收时接收了它，因为实际并没有传，所以‘undefined’那个位置接收到的就是真实的'undefined'了。是不是有点hack的味道，值得细细体会的技术，当然不是我发明的，都是从前人的经验中学习。

所以最后我们的插件成了这样：
```

;(function($, window, document,undefined) {
    //定义Beautifier的构造函数
    var Beautifier = function(ele, opt) {
        this.$element = ele,
        this.defaults = {
            'color': 'red',
            'fontSize': '12px',
            'textDecoration': 'none'
        },
        this.options = $.extend({}, this.defaults, opt)
    }
    //定义Beautifier的方法
    Beautifier.prototype = {
        beautify: function() {
            return this.$element.css({
                'color': this.options.color,
                'fontSize': this.options.fontSize,
                'textDecoration': this.options.textDecoration
            });
        }
    }
    //在插件中使用Beautifier对象
    $.fn.myPlugin = function(options) {
        //创建Beautifier的实体
        var beautifier = new Beautifier(this, options);
        //调用其方法
        return beautifier.beautify();
    }
})(jQuery, window, document);

```
一个安全，结构良好，组织有序的插件编写完成。

##  让插件支持AMD规范与RequireJS 
```

;(function (factory) {
    if (typeof define === "function" && define.amd) {
        // AMD模式
        define([ "jquery" ], factory);
    } else {
        // 全局模式
        factory(jQuery);
    }
}(function ($) {
    $.fn.jqueryPlugin = function () {
        //插件代码
    };
}));

```
##  代码混淆与压缩 
或许你很早就注意到了，你下载的插件里面，一般都会提供一个压缩的版本一般在文件名里带个'min'字样。也就是minified的意思，压缩浓缩后的版本。并且平时我们使用的jQuery也是官网提供的压缩版本，jquery.min.js。
压缩的好处
源码经过混淆压缩后，体积大大减小，使代码变得轻量级，同时加快了下载速度，两面加载变快。比如正常jQuery v1.11.0的源码是276kb，而压缩后的版本仅94.1kb！体积减小一半还多。这个体积的减小对于文件下载速度的提升不可小觑。
所使用的工具推崇的是Google开发的**Closure Compiler**。该工具需要Java环境的支持，所以使用前你可能需要先在机子上装JRE, 然后再获取Closure进行使用。
同时也有很朋在线的代码混淆压缩工具，用起来也很方便。这些工具都是一搜一大把的。

##  插件发布 

##  插件开发实战 
jQuery分页插件.
**插件开发` 不能用ID去识别插件内容标签元素 `。切记。。。。否则多个地方同时使用插件时会造成混乱**
**` 自启动闭包函数方式，回调函数中不能调用this.，因为这个时候对象引用已经是undefine了 `**
```

/**
 * jQuery分页插件
 * */
;(function (factory) {
    if (typeof define === "function" && define.amd) {
        // AMD模式
        define([ "jquery" ], factory);
    } else {
        // 全局模式
        factory(jQuery);
    }
}(function ($) {
    
     //定义MyPagePlugin的构造函数
    MyPagePlugin = function(ele, option) {
         //   this.viewHtml="<nav><ul class='pagination'><li><a id='firstPageli'>&laquo;</a></li><li><a id='prevPageli'>&lsaquo;</a></li><li class='active'><a>第<span id='curPageNoSpan'></span>页,共<span id='allPageCountSpan'></span>页</a></li><li><a id='nextPageli'>&rsaquo;</a></li><li><a id='lastPageli'>&raquo;</a></li></ul></nav>";
   this.viewHtml= "<div class='pageplugin'><a  class='first firstPageli'>&laquo;</a><a class='previous prevPageli'>&lsaquo;</a><a class='present'><input type='number' class='inputPageNo' >第<span class='curPageNoSpan'></span>页,共<span class='allPageCountSpan'></span>页</a><a class='next nextPageli'>&rsaquo;</a><a class='last lastPageli'>&raquo;</a></div>"

        this.$element = ele;
   		/**参数：page:当前页,pageCount:总共页数,onPaged回调函数,回调函数会传入页数*/
        this.defaults = {
            page:1,
            pageCount:1,
            onPaged:function(pageNo){}
        };
        this.options = $.extend({}, this.defaults, option);

    }
    //定义MyPagePlugin的方法
    MyPagePlugin.prototype = {
        initPlugin:function(){
        	this.$element.empty();
             this.$element.append(this.viewHtml);
             if(this.options.page>this.options.pageCount){
            	 this.options.pageCount=this.options.page;
             }
             this.options.onPaged(this.options.page);//初始化
             this.$element.find(".curPageNoSpan").text(this.options.page);
             this.$element.find(".inputPageNo").val(this.options.page);
             this.$element.find(".curPageNoSpan").data("options",this.options);
             this.$element.find(".allPageCountSpan").text(this.options.pageCount);
             
             this.$element.find(".firstPageli").on("click",function(e){
            	 
                var curNo=$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text();
                curNo=parseInt(curNo);
                if(curNo==1){
                	 return false;
                }else{
                	
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").data("options").onPaged(1);
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text(1);
                	$(e.currentTarget).parent("div.pageplugin").find(".inputPageNo").val(1);
                }
                return false;
             });
             this.$element.find(".prevPageli").on("click",function(e){
                var curNo=$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text();
                curNo=parseInt(curNo);
                if(curNo==1){
                    return false;
                }else{
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").data("options").onPaged(curNo-1);
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text(curNo-1);
                	$(e.currentTarget).parent("div.pageplugin").find(".inputPageNo").val(curNo-1);
                }
                return false;
             });

             this.$element.find(".inputPageNo").on("click keypress",function(e){
            	 var isReturn=true; 
            	 if(e.type=="keypress" && e.keyCode == "13")    
                 {
                    isReturn =false;
                 }
                 if(e.type=="click"){
            		 isReturn =false;
            	 }
            	 if(isReturn){
            		 return true;
            	 }
            	 var curNo=$(e.currentTarget).parent().parent("div.pageplugin").find(".curPageNoSpan").text();
                 curNo=parseInt(curNo);
                  var pageCount=$(e.currentTarget).parent().parent("div.pageplugin").find(".allPageCountSpan").text();
                  pageCount=parseInt(pageCount);
                  var inV=parseInt($(e.currentTarget).val());
                  if(inV>pageCount){
                	  inV=pageCount;
                  }else if(inV<1){
                	  inV=1;
                  }
                  $(e.currentTarget).val(inV);
                  if(curNo!=inV){
                	  $(e.currentTarget).parent().parent("div.pageplugin").find(".curPageNoSpan").data("options").onPaged(inV);
                	  $(e.currentTarget).parent().parent("div.pageplugin").find(".curPageNoSpan").text(inV);
                  }
                  return false;
                  
             });
             this.$element.find(".nextPageli").on("click",function(e){
                var curNo=$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text();
                curNo=parseInt(curNo);
                var pageCount=$(e.currentTarget).parent("div.pageplugin").find(".allPageCountSpan").text();
                pageCount=parseInt(pageCount);
                if(curNo==pageCount){
                    return false;
                }else{
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").data("options").onPaged(curNo+1);
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text(curNo+1);
                	$(e.currentTarget).parent("div.pageplugin").find(".inputPageNo").val(curNo+1);
                }
                return false;
             });
             this.$element.find(".lastPageli").on("click",function(e){
                var curNo=$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text();
                curNo=parseInt(curNo);
                var pageCount=$(e.currentTarget).parent("div.pageplugin").find(".allPageCountSpan").text();
                pageCount=parseInt(pageCount);
                if(curNo==pageCount){
                	 return false;
                }else{
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").data("options").onPaged(pageCount);
                	$(e.currentTarget).parent("div.pageplugin").find(".curPageNoSpan").text(pageCount);
                	$(e.currentTarget).parent("div.pageplugin").find(".inputPageNo").val(pageCount);
                }
                return false;
             });
             
        }


    }
    $.fn.pagePlugin = function (option) {
        var pagePlugin=new MyPagePlugin(this,option);
        pagePlugin.initPlugin();
    };
}));

```
```

.pageplugin {
  display: inline-block;
  border: 1px solid #CDCDCD;
  border-radius: 3px; }

.pageplugin a {
  cursor: pointer;
  display: block;
  float: left;
  width: 20px;
  height: 25px;
  outline: none;
  border-right: 1px solid #CDCDCD;
  border-left: 1px solid #CDCDCD;
  color: #767676;
  vertical-align: middle;
  text-align: center;
  text-decoration: none;
  font-weight: bold;
  font-size: 16px;
  font-family: Times, 'Times New Roman', Georgia, Palatino;
    background-color: #f7f7f7;
  /* ATTN: need a better font stack 
  background-color: #f7f7f7;
  background-image: -webkit-gradient(linear, left top, left bottom, color-stop(0%, #f3f3f3), color-stop(100%, lightgrey));
  background-image: -webkit-linear-gradient(#f3f3f3, lightgrey);
  background-image: linear-gradient(#f3f3f3, lightgrey); */}
  .pageplugin a:hover, .pageplugin a:focus, .pageplugin a:active {
    color:#0099CC;
    background-color: #cecece;
    background-image: -webkit-gradient(linear, left top, left bottom, color-stop(0%, #e4e4e4), color-stop(100%, #cecece));
    background-image: -webkit-linear-gradient(#e4e4e4, #cecece);
    background-image: linear-gradient(#e4e4e4, #cecece); }
  .pageplugin a.disabled, .pageplugin a.disabled:hover, .pageplugin a.disabled:focus, .pageplugin a.disabled:active {
    background-color: #f3f3f3;
    background-image: -webkit-gradient(linear, left top, left bottom, color-stop(0%, #f3f3f3), color-stop(100%, lightgrey));
    background-image: -webkit-linear-gradient(#f3f3f3, lightgrey);
    background-image: linear-gradient(#f3f3f3, lightgrey);
    color: #A8A8A8;
    cursor: default; }

.pageplugin a:first-child {
  border: none;
  border-radius: 2px 0 0 2px; }

.pageplugin a:last-child {
  border: none;
  border-radius: 0 2px 2px 0; }

 .pageplugin .present {
  float: left;
  margin: 0;
  padding: 0;
  width: 170px;
  height: 25px;
  outline: none;
  border: none;
  vertical-align: middle;
  text-align: center; }
 .inputPageNo{
    width: 45px;
    height: 25px;
 }

```
![](/data/dokuwiki/web/pasted/20151113-142348.png)
请参考http://www.cnblogs.com/Wayou/p/jquery_plugin_tutorial.html
http://www.cnblogs.com/xiaoruoen/archive/2012/01/11/2318199.html