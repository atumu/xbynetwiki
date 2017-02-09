title: js中_function_xxx 

#  JS中(function(){xxx})(); 这种写法是什么意思？ 
经常看到别人的JS脚本中有这样的写法:
```

(function(){
function a(){
   alert("a");
}
})();

```
这里的(function(){xxx})(); 是什么意思，为什么这么写，有什么好处？

解答：
这是` 自执行匿名函数： `
常见格式：` (function() { /* code */ })(); `
解释：包围函数（function(){})的**第一对括号向脚本返回未命名的函数**，**随后一对空括号立即执行返回的未命名函数，括号内为匿名函数的参数**。
使用括号包裹定义函数体，解析器将会以**函数表达式的方式**去调用定义函数。

作用：**可以用它创建命名空间**，只要把自己所有的代码都写在这个特殊的函数包装内，那么外部就不能访问，除非你允许(变量前加上window，这样该函数或变量就成为全局)。各JavaScript库的代码也基本是这种组织形式。
用(function(){xxx})()是利用**匿名函数和闭包**用来执行xxx里面的代码，同时所有的定义比如变量的作用域**都在闭包里，不会污染到外部命名空间。**
1. 使这段代码**被载入时候自动执行**。
2. **避免污染全局变量**。

总结一下，**执行函数的作用主要为 ` 匿名 ` 和 ` 自动执行 `**,代码在被解释时就已经在运行了。
因为js是函数作用域，所以如果想实现某个功能又**不想污染全局变量**的时候，会用这个自执行的匿名函数，**常见于jquery插件**

其他写法
  * (function () { /* code */ } ()); 
  * !function () { /* code */ } ();
  * ~function () { /* code */ } ();
  * -function () { /* code */ } ();
  * +function () { /* code */ } ();

##  javascript自执行函数为什么要把windows，jQuery作为参数传进去 
```

(function (window, $, undefined) {
    play=function(){
        $("#demo").val("This is a demo.");
    }
    window.wbLogin = play;
})(window, jQuery);

```
像上边这样的代码为什么要把window, jQuery对象传进去
一句话，**` 使全局变量以参数形式变成自执行函数内部的局部变量。 `**

**为什么要传入 jQuery**
通过定义一个**匿名函数，**创建了一个**“私有”的命名空间**，该命名空间的变量和方法，**不会破坏全局的命名空间**。这点非常有用也是一个** JS 框架必须支持的功能**，jQuery 被应用在成千上万的 JavaScript 程序中，必须确保 jQuery 创建的变量不能和导入他的程序所使用的变量发生冲突。

**为什么要传入 window**
通过传入 window 变量，使得 window 由全局变量变为局部变量，当在 jQuery 代码块中访问 window 时，不需要将作用域链回退到顶层作用域，这样**可以更快的访问** window；这还不是关键所在，更重要的是，将 window 作为参数传入，**可以在压缩代码时进行优化**，看看 jquery-1.6.1.min.js：
```

(function(a,b){})(window); // window 被优化为 a

``` 

**为什么要传入 undefined**
在自调用匿名函数的作用域内，确保 undefined 是真的未定义。因为 undefined 能够被重写，赋予新的值。
全选复制放进笔记
undefined = "now it's defined";
alert( undefined );

总结：一句话，**` 使全局变量以参数形式变成自执行函数内部的局部变量。 `**
**至于为什么这么做，提高程序效率。**为什么能提高效率，得从javascript的机制说起，所谓的` scope chain作用域链 `，在当前作用域中如果没有该属性（局部变量）则向上一层作用域中寻找，一直到最上层，也就是window。也就是说全局变量和下级作用域都是window的一个属性，向下依此类推。
**另外jQuery传入后将参数写成$可以保证在此函数内$为jquery而不是其他类似使用$符号的库。**
undefined同理，由于没有传入第三个参数，自然就是undefined。由于javascript中undefined是一个变量，可以被改变，所以这样可以保证undefined判断时的准确性。有时判断时使用typeof xxx === 'undefined'也是因为这个原因。

参考：
http://segmentfault.com/q/1010000000135703
http://segmentfault.com/q/1010000000311686
http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html