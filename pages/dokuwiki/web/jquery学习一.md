title: jquery学习一 

#  :jQuery学习一 
jQuery语法：
获得内容 -`  text()、html() 以及 val() ` 
  * text() - 设置或返回所选元素的文本内容
  * html() - 设置或返回所选元素的内容（包括 HTML 标记）
  * val() - 设置或返回表单字段的值


jQuery语法是为HTML元素的选取编制，可以对元素执行某些操作。

基础语法：$(selector).action()

$  定义jQuery

选择器(selector)   查询和查找HTML元素

action()   执行对元素的操作

文档就绪函数：防止文档在完全加载（就绪）之前运行 jQuery 代码。

```

$(document).ready(function(){
--- jQuery functions go here ----
});

```
##  jQuery选择器 


```

//jQuery 元素选择器
$("p") //选取 <p> 元素。
$("p.intro") //选取所有 class="intro" 的 <p> 元素。
$("p#demo") //选取 id="demo" 的第一个 <p> 元素
//jQuery 属性选择器
$("[href]") //选取所有带有 href 属性的元素。
$("[href='#']") //选取所有带有 href 值等于 "#" 的元素。
$("[href!='#']") //选取所有带有 href 值不等于 "#" 的元素。
$("[href$='.jpg']") //选取所有 href 值以 ".jpg" 结尾的元素。
//jQuery CSS 选择器
jQuery CSS 选择器可用于改变 HTML 元素的 CSS 属性。
//实例
$("p").css("background-color","red");
复制代码
//更多实例：
$(this) //当前 HTML 元素 
$("p") //所有 <p> 元素 
$("p.intro") //所有 class="intro" 的 <p> 元素 
$(".intro") //所有 class="intro" 的元素 
$("#intro") //id="intro" 的第一个元素 
$("ul li:first") //每个 <ul> 的第一个 <li> 元素 
$("[href$='.jpg']") //所有带有以 ".jpg" 结尾的 href 属性的属性 
$("div#intro .head") //id="intro" 的 <div> 元素中的所有 class="head" 的元素

``` 
##  jQuery CSS函数 
```

$(selector).css(name,value) //为所有匹配元素的给定 CSS 属性设置值
$("p").css("background-color","yellow");
$(selector).css({properties}) //为所有匹配元素的一系列 CSS 属性设置值
$("p").css({"background-color":"yellow","font-size":"200%"});
$(selector).css(name) //返回指定的 CSS 属性的值
$(this).css("background-color");

```
##  jQuery HTML操作 

```

改变HTML内容：

$(selector).html(content) //改变所匹配的 HTML 元素的内容（innerHTML）
 
添加HTML内容：

$(selector).append(content)  //向所匹配的 HTML 元素内部追加内容。
$(selector).prepend(content)  //向所匹配的 HTML 元素内部预置（Prepend）内容。
$(selector).after(content)  //在所有匹配的元素之后插入 HTML 内容。
$(selector).before(content)  //在所有匹配的元素之前插入 HTML 内容。

```

##  第一章认识JQuery 
**使用jQuery库之后，开发者操作的对象不再是DOM元素了，所以不能再使用DOM对象的方法**。而是jQuery对象。而是应该以jQuery对象所支持的属性和方法操作对应的DOM对象。
```


·页面加载事件（可以写多个ready()）
$(document).ready(function(){
alert(“hello world”);
})

·链式操作：JQuery允许你在一句代码中操做任何与其相关联的元素，包括其子元素、父元素等
//选择名称为myDiv的元素，为其自身添加css1的样式，然后再选择其所有子元素a，为其移除css2样式
$(“#myDiv”).addClass(“css1″).children(“a”).removeClass(“css2″);

·JQuery中获得一个对象的所有子元素内容
$(“#myDiv”).html()

·JQuery中的变量 与 DOM中的变量
var $myVar = “”;
var myVar = “”;

·DOM对象 转换成 JQuery对象
var obj = documnet.getElementById(“myDiv”);
var $obj = $(obj);

·JQuery对象 转换成 DOM对象
var $obj = $(“#myDiv”);
var obj = $obj.get(0);  //或者var obj = $obj[0];

·释放JQuery对$符号的控制权
JQuery.noConflict();

```

##  第二章 JQuery选择器 
```


·JQuery完善的处理机制
document.getElementById(“test”).style.color = “red”; //如果test不存在，则页面出现异常
$(“#test”).css(“color”,”red”); //哪怕页面没有名称为test的元素，也不会报错。它是一个JQuery对象

·判断页面是否选择的对象
if( $(“.class”).length > 0 ){
// todo something
}

·基本选择器
$(“#myDiv”)    //根据给定的ID选择匹配的元素，返回：单个元素
$(“.myClass”) //根据给定的样式名称选择匹配的元素，返回：集合元素
$(“div”) //根据给定的元素名称选择匹配的元素，返回：集合元素
$(“#myDiv,div.myClass,span”) //根据给定的规则选择匹配的元素，返回：集合元素
$(“*”) //选择页面所有元素，返回：集合元素

·层次选择器
$(“div span”) //选择所有DIV元素下的所有SPAN元素（所有后代元素），返回：集合元素
$(“div>span”) //选择所有DIV元素下的SPAN子元素（仅子元素），返回：集合元素
$(“.myClass+div”) //选择样式名称为myClass的下一个DIV元素，返回：集合元素
$(“.myClass+div”) //等价于 $(“.myClass”).next(“div”);
$(“.myClass~div”) //选择样式名称为myClass之后的所有DIV元素，返回：集合元素
$(“.myClass~div”) //等价于 $(“.myClass”).nextAll();
$(“.myClass”).siblings(“div”) //选择样式名称为myClass的元素的所有同辈DIV元素（无论前后），返回集合元素

·过滤选择器（index从0开始）
$(“div:first”) //选择所有DIV元素下的第一个DIV元素，返回：单个元素
$(“div:last”) //选择所有DIV元素下的最后一个DIV元素，返回：单个元素
$(“div:not(.myClass)”) //选择所有样式不包括myClass的DIV元素，返回：集合元素
$(“div:even”) //选择所有索引是偶数的DIV元素，返回：集合元素
$(“div:odd”) //选择所有索引是奇数的DIV元素，返回：集合元素
$(“div:eq(index)”) //选择所有索引等于index的DIV元素，返回：集合元素
$(“div:gt(index)”) //选择所有索引大于index的DIV元素，返回：集合元素
$(“div:lt(index)”) //选择所有索引小于index的DIV元素，返回：集合元素
$(“:header”) //选择所有标题元素（h1,h2,h3），返回：集合元素
$(“div:animated”) //选择所有正在执行去画的DIV元素，返回：集合元素

·子元素过滤选择器（index从1开始）
$(“:nth-child(index/even/odd)”) //选择每个父元素下的第index/偶数/奇数个子元素，返回：集合元素
$(“:first-child”) //选择每个父元素下的第一个子元素，返回：集合元素
$(“:last-child”) //选择每个父元素下的最后一个子元素，返回：集合元素
$(“ul li:only-child”) //在UL元素中选择只有一个LI元素的子元素，返回：集合元素

·内容过滤选择器
$(“:contains(text)”) //选择所有内容包含text的元素，返回：集合元素
$(“div:empty”) //选择所有内容为空的DIV元素，返回：集合元素
$(“div:has(span)”) //选择所有含有SPAN子元素的DIV元素，返回：集合元素
$(“div:parent”) //选择所有含有子元素的DIV元素，返回：集合元素

·可见性选择器
$(“:hidden”) //选择所有不可见的元素（type=”hidden” style=”display:none” style=”visibility:none”），返回：集合元素
$(“:visible”) //选择所有可见的元素，返回：集合元素

·属性过滤选择器
$(“[id]“) //选择所有含有id属性的元素，返回：集合元素
$(“[class=myClass]“) //选择所有class属性值是myClass的元素，返回：集合元素
$(“[class!=myClass]“) //选择所有class属性值不是myClass的元素，返回：集合元素
$(“[alt^=begin]“) //选择所有alt属性值以begin开始的元素，返回：集合元素
$(“[alt^=end]“) //选择所有alt属性值以end结束的元素，返回：集合元素
$(“[alt*=some]“) //选择所有alt属性值含有some的元素，返回：集合元素
$(“div[id][class=myClass]“) //选择所有含有id属性的并且class属性值是myClass的元素，返回：集合元素

·表单对象属性选择器
$(“#myForm:enabled”) //选择ID属性为myForm的表单的所有可用元素，返回：集合元素
$(“#myForm:disabled”) //选择ID属性为myForm的表单的所有不可用元素，返回：集合元素
$(“#myForm:checked”) //选择ID属性为myForm的表单的所有所有被选中的元素，返回：集合元素
$(“#myForm:selected”) //选择ID属性为myForm的表单的所有所有被选中的元素，返回：集合元素

·表单选择器
$(“:input”) //选择所有<input> <select> <button> <textarea>元素，返回：集合元素
$(“:text”) //选择所有单行文本框元素，返回：集合元素
$(“:password”) //选择所有密码框元素，返回：集合元素
$(“:radio”) //选择所有单选框元素，返回：集合元素
$(“:checkbox”) //选择所有复选框元素，返回：集合元素
$(“:submit”) //选择所有提交按钮元素，返回：集合元素
$(“:image”) //选择所有图片按钮元素，返回：集合元素
$(“:reset”) //选择所有重置按钮元素，返回：集合元素
$(“:button”) //选择所有按钮元素，返回：集合元素
$(“:file”) //选择所有上传域元素，返回：集合元素
$(“:hidden”) //选择所有不可见域元素，返回：集合元素
$(“:text”) //选择所有单选文本框元素，返回：集合元素

```
##  第三章 JQuery中的DOM操作 
```


·查找元素节点
var str = $(“#myDiv”).text(); //<div id=”myDiv” title=”hello”>123</div>
alert(str); //结果：123

·查找属性节点
var str = $(“#myDiv”).attr(“title”); //<div id=”myDiv” title=”hello”>123</div>
alert(str); //结果：hello

·创建元素节点
var $li1 = $(“<span></span>”); //传入元素标记，自动包装并创建第一个li元素对象
var $li2 = $(“<span></span>”); //第二个,创建时需要遵循XHTML规则（闭合、小写）
$(“#myDiv”).append($li1); //往id为myDiv的元素中添加一个元素
$(“#myDiv”).append($li2); //结果：<div id=”myDiv”><span></span><span></span></div>

$(“#myDIv”).append($li1).append($li2); //客串：传说中的链式写法，省一行代码 ^_^

·创建文本节点
var $li1 = $(“<span>first</span>”);
var $li2 = $(“<span>second</span>”);
$(“#myDIv”).append($li1).append($li2);
// 结果：<div id=”myDiv”><span>first</span><span>second</span></div>

·创建属性节点
var $li1 = $(“<span title=”111″>first</span>”);
var $li2 = $(“<span title=”222″>second</span>”);
$(“#myDIv”).append($li1).append($li2);
// 结果：<div id=”myDiv”><span title=”111″>first</span><span title=”222″>second</span></div>

·插入节点
$(“#myDiv”).append(“<span></span>”); //往id为myDiv的元素插入span元素
$(“<span></span>”).appendTo(“#myDiv”); //倒过来，将span元素插入到id为myDiv的元素

$(“#myDiv”).prepend(“<span></span>”); //往id为myDiv的元素内最前面插入span元素
$(“<span></span>”).prependTo(“#myDiv”); //倒过来，将span元素插入到id为myDiv的元素内的最前面

$(“#myDiv”).after(“<span></span>”); //往id为myDiv的元素后面插入span元素（同级，不是子元素）
$(“<span></span>”).insertAfter(“#myDiv”); //倒过来，将span元素插入到id为myDiv的元素后面（同级，不是子元素）

$(“#myDiv”).before(“<span></span>”); //往id为myDiv的元素前面插入span元素（同级，不是子元素）
$(“<span></span>”).insertBefore(“#myDiv”); //倒过来，将span元素插入到id为myDiv的元素前面（同级，不是子元素）

·删除节点
$(“#myDiv”).remove(); //将id为myDiv的元素移除

·清空节点
$(“#myDiv”).remove(“span”); //将id为myDiv的元素内的所有span元素移除

·复制节点
$(“#myDiv span”).click( function(){ //点击id为myDiv的元素内的span元素，触发click事件
$(this).clone().appendTo(“#myDiv”); //将span元素克隆，然后再添加到id为myDiv的元素内
$(this).clone(true).appendTo(“#myDiv”); //如果clone传入true参数，表示同时复制事件
})

·替换节点
$(“p”).replaceWith(“<strong> 您好</strong>”); //将所有p元素替换成后者 <p>您好</p> –> <strong>您好</strong>
$(“<strong>您好</strong>”).replaceAll(“p”); //倒过来写，同上

·包裹节点
$(“strong”).wrap(“<b></b>”); //用b元素把所有strong元素单独包裹起来 <b><strong>您好</strong>< /b><b><strong>您好< /strong></b>
$(“strong”).wrapAll(“<b></b>”); //用b元素把所有strong元素全部包裹起来 <b><strong>您 好</strong><strong>您好< /strong></b>
$(“strong”).wrapInner(“<b></b>”); //把b元素包裹在strong元素内 <strong><b>您好</b>< /strong>

·属性操作
var txt = $(“#myDiv”).arrt(“title”); //获取id为myDiv的元素的title属性
$(“#myDiv”).attr(“title”,”我是标题内容”); //设置id为myDiv的元素的title属性的值
$(“#myDiv”).attr({“title”:”我是标题内容”, “alt”:”我还是标题”); //一次性设置多个属性的值
$(“#myDiv”).removeArrt(“alt”); //移除id为myDiv的元素的title属性

·样式操作
var txt = $(“#myDiv”).arrt(“class”); //获取id为myDiv的元素的样式
$(“#myDiv”).attr(“class”,”myClass”); //设置id为myDiv的元素的样式
$(“#myDiv”).addClass(“other”); //在id为myDiv的元素中追加样式
$(“#myDiv”).removeClass(“other”); //在id为myDiv的元素中移除other样式
$(“#myDiv”).removeClass(“myClass other”); //在id为myDiv的元素中移除myClass和other多个样式
$(“#myDiv”).removeClass(); //在id为myDiv的元素中移除所有样式
$(“#myDiv”).toggleClass(“other”); //切换样式，在有other样式和没other样式之间切换
$(“#myDiv”).hasClass(“other”); //判断是否有other样式

·设置和获取HTML、文本和值
alert( $(“#myDiv”).html() ); //获取id为myDiv的元素的HTML代码（相当于innerHTML）
$(“#myDiv”).html(“<span>hello</span>”); //设置id为myDiv的元素的HTML代码

alert( $(“#myDiv”).text() ); //获取id为myDiv的元素的HTML代码（相当于innerText）
$(“#myDiv”).text(“hello”); //设置id为myDiv的元素的HTML代码

alert( $(“#myInput”).val() ); //获取id为myDiv的元素的value值（支持文本框、下拉框、单选框、复选框等）
$(“#myInput”).val(“hello”); //设置id为myDiv的元素的value值（下拉框、单选框、复选框带有选中效果）

·遍历节点
var $cList = $(“#myDiv”).children(); //获取id为myDiv的元素的子元素（只考虑子元素，不考虑后代元素）
var $sNext = $(“#myDiv”).next(); //获取id为myDiv的元素的下一个同辈元素
var $sPrev = $(“#myDiv”).prev(); //获取id为myDiv的元素的上一个同辈元素
var $sSibl = $(“#myDiv”).siblings(); //获取id为myDiv的元素的所有同辈元素
var $pClos = $(“#myDiv”).closest(“span”); //获取id为myDiv的元素本身开始，最接近的span元素（向上查找）

·CSS-DOM操作
$(“#myDiv”).css(“color”); //获取id为myDiv的元素的color样式的值
$(“#myDiv”).css(“color”, “blue”); //设置id为myDiv的元素的color样式的值
$(“#myDiv”).css({“color”:”blue”, “fontSize”:”12px”}); //设置id为myDiv的元素的color样式的值（多个）

$(“#myDiv”).css(“opacity”, “0.5″); //设置id为myDiv的元素的透明度（兼容浏览器）

$(“#myDiv”).css(“height”); //获取id为myDiv的元素的高度（单位：px，兼容浏览器）
$(“#myDiv”).height(); //同上（实际高度）

$(“#myDiv”).css(“width”); //获取id为myDiv的元素的宽度（单位：px，兼容浏览器）
$(“#myDiv”).width(); //同上（实际宽度）

var offset = $(“#myDiv”).offset(); //获取id为myDiv的元素在当前窗口的相对偏移量
alert( offset.top + “|” + offset.left );

var offset = $(“#myDiv”).position(); //获取id为myDiv的元素相对于最近一个position设置为relative或absolute的父元素的相对偏移量
alert( offset.top + “|” + offset.left );

$(“#txtArea”).scrollTop(); //获取id为txtArea的元素滚动条距离顶端的距离
$(“#txtArea”).scrollLeft(); //获取id为txtArea的元素滚动条距离左侧的距离
$(“#txtArea”).scrollTop(100); //设置id为txtArea的元素滚动条距离顶端的距离
$(“#txtArea”).scrollLeft(100); //设置id为txtArea的元素滚动条距离左侧的距离

```
##  第四章 JQuery中的事件和动画 
```


·加载DOM
$(window).load() 等价于 window.onload 事件

$(document).ready() 相当于window.onload事件，但有些区别：
(1)执行时机：
window.onload 是在网页中所有元素（包括元素的所有关联文件）完全加载后才执行
$(document).ready() 是在DOM完全就绪时就可以被调用，此时，并不意味着这些元素关系的文件都已经下载完毕

(2)多次使用：可以在同一个页面注册多个$(document).ready()事件
(3)简写方式：可以缩写成 $(function(){ })  或  $().ready()

·事件绑定
当文档装载完成后，可以通过bind()方法，为特定的元素进行事件的绑定，可重复多次使用
bind( type, [data, ] fn );
type：指事件的类型:
blur（失去焦点）、focus（获得焦点）
load（加载完成）、unload（销毁完成）
resize（调整元素大小）、scroll（滚动元素）
click（单击元素事件）、dbclick（双击元素事件）
mousedown（按下鼠标）、mouseup（松开鼠标）
mousemove（鼠标移过）、mouseover（鼠标移入）、mouseout（鼠标移出）
mouseenter（鼠标进入）、mouseleave（鼠标离开）
change（值改变）、select（下拉框索引改变）、submit（提交按钮）
keydown（键盘按下）、keyup（键盘松开）、keypress（键盘单击）
error（异常）
data：指事件传递的属性值，event.data 额外传递给对象事件的值
fn：指绑定的处理函数，在此函数体内，$(this)指携带相应行为的DOM元素

·合并事件
hover(enter,leave)：鼠标移入执行enter、移出事件执行leave
$(“#myDiv”).hover( function(){
$(this).css(“border”, “1px solid black”);0
}, function(){
$(this).css(“border”, “none”);
});

toggle(fn1,fn2,…fnN)：鼠标每点击一次，执行一个函数，直到最后一个后重复
$(“#myDiv”).toggle( function(){
$(this).css(“border”, “1px solid black”);0
}, function(){
$(this).css(“border”, “none”);
});

·事件冒泡
下面的例子，BODY元素下有DIV元素，DIV元素下有SPAN元素，分别将三种元素都注册click事件。
那么，click事件会按照DOM的层次结构，像水泡一样不断向上直到顶端，所以称之为事件冒泡。
<body><div><span> 我是SPAN我怕谁</span></div></body>
$(“span”).bind(“click”, function(){ alert(‘span click’); });
$(“div”).bind(“click”, function(){ alert(‘div click’); });
$(“body”).bind(“click”, function(){ alert(‘body click’); });

·阻止冒泡
解决这个问题的办法是：在SPAN执行完click事件后，停止事件冒泡。
$(“span”).bind(“click”, function(event){
alert(‘span click’);
event.stopPropagation(); //停止冒泡
});

·阻止默认行为
提交按钮在提交前做相应的逻辑判断，当不满足时
$(“#btnSubmit”).bind(“click”, function(event){
event.preventDefault(); //阻止默认行为 相当于return false;
});

·事件对象的属性
$(“#myDiv”).bind(“click”, function(event){ });
event.type() //返回：click
event.target() //获取当前元素
event.relatedTarget() //引发事件的元素
event.pageX()/event.pageY() //获取鼠标相对于页面的X和Y坐标
event.which() //在单击事件中获取到对应的按键 鼠标左中右分别是123
event.metaKey() //获取操作中的相关功能键（ctrl/alt/shift）

·移除事件
$(“#myDiv”).bind(“click”, fn1 = function(){
alert(“function1″);
}).bind(“click”, fn2 = function(){
alert(“function2″);
}).bind(“click”, fn3 = function(){
alert(“function3″);
});
$(“#myDiv”).unbind(); //移除id为myDiv的元素的所有事件
$(“#myDiv”).unbind(“click”); //移除id为myDiv的元素的所有click事件
$(“#myDiv”).unbind(“click”,fn1); //移除id为myDiv的元素的名称为fn1的click事件

·一次性事件：绑定的事件执行一次后自动移除
$(“#myDiv”).one(“click”, [data], function(){
alert(“function1″);
});

·触发事件
$(“#btn”).trigger(“click”, [data]); //代码方式触发click事件
$(“#btn”).click(); //另一种简写方式

·事件命名空间
$(“#myDiv”).bind(“click.hello”, function(){
alert(“function1″);
});
$(“#myDiv”).bind(“click”, function(){
alert(“function1″);
})
$(“div”).unbind(“click”); //两个事件都被移除
$(“div”).unbind(“.hello”); //只移除第一个
$(“div”).unbind(“click!”); //只移除第二个（注意感叹号，指没有名字空间的）

·JQuery中的动画
$(“div”).hide(); //隐藏所有DIV元素，相当于sytle=”display:none”
$(“div”).show(); //显示所有DIV元素

$(“div”).hide(1000); //一秒内隐藏所有DIV元素，其它参数还有：slow(600) normal(400) fast(200)
$(“div”).show(1000); //一秒内显示所有DIV元素

$(“div”).fadeOut(); //降低元素的不透明度，直至消失（支持速度参数，不会改变宽高）
$(“div”).fadeIn(); //升高元素的不透明度，直至显示

$(“div”).slideUp(); //由下至上收缩元素，直至消失（支持速度参数）
$(“div”).slideDown(); //由上至下展开元素，直至显示

·自定义动画animate
$(elem).animate(params, speed, callback);
params：样式属性及值的映射 {protected:”value”, protected:”value”}
speed: 速度参数
callback: 动画完成后执行函数，可选

$(“#myDiv”).animate({left:”500px”}, 2000); //两秒内ID为myDiv的元素移至左边距500px的位置
$(“#myDiv”).animate({left:”+=500px”}, 2000); //同上，支持累加、累减
$(“#myDiv”).animate({top:”200px”, left:”+=500px”}, 2000); //同上，多重动画，同时执行

$(“#myDiv”).animate({opacity:”0.5″}, 1000) //先变成50%透明
.animate({top:”500px”}, 500) //移至离顶端500px
.animate({left:”500px”}, 500) //移至离左边500px
.fadeOut(1000); //显示出来 （四个动作为队列，一步步执行）

$(“#myDiv”).stop([cleanQuene] [,gotuEnd]); //停止动画，参数为boolean

$(“#myDiv”).is(“:animate”) //判断元素是否在执行动画

·其它动画
$(“#myDiv”).toggle(); //显示与隐藏元素
$(“#myDiv”).slideToggle(); //展开与收缩元素
$(“#myDiv”).fadeTo(1000, 0.2); //一秒内将元素透明度调整到20%

```
##  第五章 JQuery对表单、表格的操作及更多应用 
```


·单选文本框应用（获得焦点时，加了个特殊的样式，失去焦点时还原，兼容所有浏览器）
$(“:input”).focus(function(){ this.addClass(“inputFocus”); })
.blur(function(){ this.removeClass(“inputFocus”); });

·多行文本框的应用（放大、缩小多行文本框的高度，限制最大500px，兼容所有浏览器）
var $txt = $(“#textArea”);
$(“.bigger”).click(function(){
if( $txt.height() < 500) $txt.height( $txt.height() + 50 );
//if( $txt.height() < 500) $txt.animate({height:”+=50″}, 500 );
});
$(“.smaller”).click(function(){
if( $txt.height() > 100) $txt.height( $txt.height() – 50 );
//if( $txt.height() < 500) $txt.animate({height:”-=50″}, 500 );
});

·复选框的应用（实现全选、全不选、反选）
$(“#btnCheckedAll”).click(function(){ //全选
$(“[name=items]:checkbox”).attr(“checked”, true);
});
$(“#btnCheckedNone”).click(function(){ //全不选
$(“[name=items]:checkbox”).attr(“checked”, false);
});
$(“#btnCheckedRev”).click(function(){ //反选
$(“[name=items]:checkbox”).each(function(){
$(this).attr(“checked”, !$(this).attr(“checked”));
//this.checked = !this.checked;
}
});

·下拉框的应用（将一个下拉列表的选中项搬至另一个下拉列表）
$(“#btnAdd”).click(function(){ //将选中选项搬过去
$(“#mySelect1 option:selected”).appendTo(“#mySelect2″);
});
$(“#btnAddAll”).click(function(){ //将全部选项搬过去
$(“#mySelect1 option”).appendTo(“#mySelect2″);
});
$(“#mySelect1″).dblclick(function()[ //双击项搬过去
$("#mySelect1 option:selected").appendTo("#mySelect2");
}

·表单验证
<form>
<div>
<label>用户名：</label>
<input type="text" id="txtUid" value="" />
</div>
</form>
$("form :input.required").each(function(){ //往每个class有required样式的input元素后面添加*号
$(this).parent().append( $("<span class='star'>*</span>") );
});
$("form :input.required").blur(function(){ //失去焦点时验证域
if( this.value == "" ){
$(this).parent().append( $("<span class='error'>必填字段</span>") );
}
else{
$(this).parent().append( $("<span class='success'>验证正确</span>") );
$(this).parent().find(".error").remove();
}
}).keyup(function(){ //用户每点一个键触发
$(this).triggerHandler("blur");
}).focus(function(){ //控制有焦点时触发
$(this).triggerHandler("blur");
});
$("#btnSubmit").click(function(){
$("form :input.required").trigger("blur"); //让所有需要验证的域失去焦点
var errNum = $("form .error").length;
if( errNum ){
alert("有验证字段失败，请重新填写");
return false;
}
});

·表格应用
$("tr:odd").addClass("oddTr"); //给奇数行添加oddTr样式
$("tr:even").addClass("evenTr"); //给偶数行添加evenTr样式

$("tr:contains('王五')").addClass("highlightTr"); //查找包含”王五”的行，添加highlightTr样式

$("tr").click(function(){
$(this).addClass("selectedTr") //给当前行添加选中样式
.siblings().removeClass("selectedTr") //反选移除选中样式
.end() //结束，返回$(this)，否则则是反选的行
.find(':radio").attr("checked",true); //在当前行查找单选框，选中它
});

```
##  第六章 JQuery与Ajax的应用 


```


·load( url [,data] [,callback] )方法
url：要请求的页面的地址
data：要发送的相关参数
callback：回调函数

$(“#myDiv”).load(“hello.html”); //向myDiv元素加载hello.html的内容
$(“#myDiv”).load(“hello.html .myClass”); //筛选，只加载hello.html中myClass样式的内容

$(“#myDiv”).load(“hello.html”, function(){} ); //没参数的，使用GET方式
$(“#myDiv”).load(“hello.html”, {id:’123′, name:’dier’}, function(){} ); //有参数的，使用POST方式

$(“#myDiv”).load(“hello.html”, function(responseText, textStatus, XMLHttpRequest){ //回调函数
//responseText : 请求返回的内容
//textStatus : 请求状态 success error notmodified timeout
//XMLHttpRequest : Ajax对象
});
示例：
$(document).ready(function(){
  $("button").click(function(){
    $("#div1").load("/example/jquery/demo_test.txt",function(responseTxt,statusTxt,xhr){
      if(statusTxt=="success")
        alert("外部内容加载成功！");
      if(statusTxt=="error")
        alert("Error: "+xhr.status+": "+xhr.statusText);
    });
  });

--------------
·$.get( url [,data] [,callback] [,type])和$.post( url [,data] [,callback] [,type])方法
url：要请求的页面的地址
data：要发送的相关参数
callback：回调函数
type：指定服务器返回内容的格式 xml html script json text _default

$.get( “test.aspx”, {id:”123″, name:”dier”}, function(data,textStatus){ //回调函数只有当状态是success才触发
//data : 请求返回的内容
//textStatus : 请求状态 success error notmodified timeout

//当data是HTML时，直接加载
$(“#myDiv”).html(data);

//当data是XML时，可筛选 <user id=”123″ name=”dier” age=”27″ />
var age = $(data).find(“user”).attr(“age”);

//当data是JSON时，可直接点出属性来 {id:”123″, name:”dier”, age:”27″}
var age = data.age;
});

·getScript(url [,callback])方法
$(function(){ //动态加载JS脚本
$.getScript(“test.js”);

$.getScript(“test.js”, function(){ //回调函数
//do something..
});
});

·getJSON(url [,callback])方法
$(function(){ //动态加载JS脚本
$.getJSON(“test.js”);

$.getJSON(“test.js”, function(data){ //回调函数
//do something..
//data : 返回的数据
$.each( data, function(index, item){ //遍历，相当于foreach
//index : 索引
//item : 当前项内容
//return false; 退出循环
});
});
});

jQuery $.post() 方法
$.post() 方法通过 HTTP POST 请求从服务器上请求数据。
返回类型jqXHR：参考：http://api.jquery.com/jQuery.ajax/#jqXHR
语法：
jQuery.post(url,data,success(data, textStatus, jqXHR),dataType)
url必需。规定把请求发送到哪个 URL。
data可选。映射或字符串值。规定连同请求发送到服务器的数据。
success(data, textStatus, jqXHR)可选。请求成功时执行的回调函数。
dataType可选。规定预期的服务器响应的数据类型。默认执行智能判断（xml、json、script 或 html）。
jQuery 1.5 中的约定接口同样允许 jQuery 的 Ajax 方法，包括 $.post()，来链接同一请求的多个 .success()、.complete() 以及 .error() 回调函数，甚至会在请求也许已经完成后分配这些回调函数。
示例：
// 请求生成后立即分配处理程序，请记住该请求针对 jqxhr 对象
 var jqxhr = $.post("example.php", function() { 
alert("success"); })
 .success(function() { alert("second success"); })
 .error(function() { alert("error"); }) 
.complete(function() { alert("complete"); }); 
// 在这里执行其他任务 
// 为上面的请求设置另一个完成函数 jqxhr.complete(function(){ alert("second complete"); });


·ajax(options)方法
url : 请求的地址
type : 请求的方式 GET POST 默认为GET
timeout : 请求超时时间(单位：毫秒）
data : 请求时发送的参数（String,Object）
dataType : 预期返回的数据类型 xml html script json jsonp text
bdforeSend : 发送请求前触发事件，如果return false则取消发送 function(XmlHttpRequest){}
complete : 请求完成后触发事件，不管成功与否 function(XmlHttpRequest, textStatus){}
success : 请求完成并且成功时触发事件 function(data, textStatus){}
error : 请求完成并且失败时触发事件 function(XmlHttpRequest, textStatus, errorThrown){}
global : 是否为全局请求，默认为true，可使用AjaxStart、AjaxStop控制各种事件

$.ajax({
url : “test.aspx”,
type : “POST”,
timeout : “3000″,
data : {id:”123″, name:”dier”},
dataType : “HTML”,
success : function(data,textStatus){
$(“#myDiv”).html( data );
}
error : function(XmlHttpRequest, textStatus, errThrown){
$(“#myDiv”).html( “请求失败：” + errThrown );
}
});

·序列化字符串 serialize()
$.get( “test.aspx”, $(“#form1″).serialize(), function(data,textStatus){
//将form1整个表单中的所有域序列化成提交的参数，支持自动编码
});

·序列化数组 serializeArray()
var arr = $(“:checkbox, :radio”).serializeArray();

·对象序列化 param()
var obj = {id:”123″, name:”dier”, age:”27″};
var kv = $.param(obj); //id=123&name=dier&age=27

·JQuery中的全局Ajax事件
ajaxStart(callback) //请求开始时触发
ajaxStop(callback) //请求结束时触发
ajaxComplete(callback) //请求完成时触发
ajaxSuccess(callback) //请求成功时触发
ajaxError(callback) //请求失败时触发
ajaxSend(callback) //请求发送前触发

$(“#loading”).ajaxStart(function(){ //当有AJAX请求时显示，完成时隐藏
$(this).show();
}.ajaxStop(function(){
$(this).hide();
}
);

```

参考：http://blog.163.com/wenchangqing_live/blog/static/17372230920101010299793/
http://www.cnblogs.com/niuniu1985/archive/2010/08/19/1803381.html
http://my.oschina.net/u/129225/blog/29311