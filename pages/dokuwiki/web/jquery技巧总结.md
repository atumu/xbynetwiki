title: jquery技巧总结 

#  jQuery技巧总结 
##   jQuery页面加载初始化的几种方法 
主要看习惯吧，本人觉得第二种方法最好，比较简洁。
```

$(document).ready(function(){  
     alert("第一种方法。");   
}); 

```
```

$(function(){  
    alert("第二种方法。");  
});

```
```

jQuery(function($) {  
    alert("第三种方法。");  
});

```

参考：http://blog.csdn.net/tjcyjd/article/details/6713474

##  jQuery获取当前节点的html包含当前节点的方法 
jQuery 获取当前节点的html` 并且包含当前节点 `的方法
在开发过程中，` jQuery.html() ` 是获取当前节点下的html代码，**并不包含当前节点本身的代码，**然后我们有时候确需要，找遍jQuery api文档也没有任何方法可以拿到。
看到有的人**通过parent().html()，如果当前元素没有兄弟元素还行，如果有那就行不通了。**后台实验发现有一个jQuery的一个方法可以解决，而且非常简便，如下：
` jQuery.prop("outerHTML"); `
```

<div class="test"><p>hello，你好！</p></div>
<script>
$(".test").prop("outerHTML");
</script>
输出结果为：<div class="test"><P>hello,你好！</p></div>

```
因为**原生JS DOM里有一个内置属性 ` outerHTML ` （看清大小写哦，JS是区分大小写的）****用来获取当前节点的html代码(包含当前节点)**，所以**用jQuery的prop()能拿到**。
经过实验attr()方法是拿不到的，不信的话，大家也可以尝试尝试，谢谢。

当然也有人用jQuery的 clone() 函数配合append() 来创建一个只有一个子元素的节点，然后来拿节点的html，这样也是可行的，但是代码繁琐。

##  禁止右键点击 
```

$(document).ready(function(){
    $(document).bind("contextmenu",function(e){
            return false;
    });
});

```
##  在新窗口中打开链接 
```

$(document).ready(function() {
   //Example 1: Every link will open in a new window
   $('a[href^="http://"]').attr("target", "_blank"); 
   //Example 2: Links with the rel="external" attribute will only open in a new window
   $('a[@rel$='external']').click(function(){
         this.target = "_blank";
   });
});// how to use
<a href="http://www.opensourcehunter.com" rel=external>open link</a>

```
##  检测浏览器 
注: 在版本jQuery 1.4中，$.support 替换掉了$.browser 变量
```

$(document).ready(function() {
// Target Firefox 2 and above
if ($.browser.mozilla && $.browser.version >= "1.8" ){
    // do something
}
// Target Safari
if( $.browser.safari ){
    // do something
}
// Target Chrome
if( $.browser.chrome){
    // do something
}
// Target Camino
if( $.browser.camino){
    // do something
}
// Target Opera
if( $.browser.opera){
    // do something
}
// Target IE6 and below
if ($.browser.msie && $.browser.version <= 6 ){
    // do something
}
// Target anything above IE6
if ($.browser.msie && $.browser.version > 6){
    // do something
}
});

```
##   预加载图片 
```

$.preloadImages = function () {  for (var i = 0; i < arguments.length; i++) {
    $('<img>').attr('src', arguments[i]);
  }
};
$.preloadImages('img/hover1.png', 'img/hover2.png');

```
**检查图片是否加载完成**
有时候你需要确保图片完成加载完成以便执行后面的操作：
```

$('img').load(function () {
  console.log('image load successful');
});

```
你可以把 img 替换为其他的 ID 或者 class 来检查指定图片是否加载完成。
##   页面样式切换 
```

$(document).ready(function() {
    $("a.Styleswitcher").click(function() {
        //swicth the LINK REL attribute with the value in A REL attribute
        $('link[rel=stylesheet]').attr('href' , $(this).attr('rel'));
    });
// how to use
// place this in your header
<LINK rel=stylesheet type=text/css href="default.css">
// the links
<A class=Styleswitcher href="#" rel=default.css>Default Theme</A>
<A class=Styleswitcher href="#" rel=red.css>Red Theme</A>
<A class=Styleswitcher href="#" rel=blue.css>Blue Theme</A>

```
##   获得鼠标指针ＸＹ值 
```

$(document).ready(function() {
   $().mousemove(function(e){
     //display the x and y axis values inside the div with the id XY
    $('#XY').html("X Axis : " + e.pageX + " | Y Axis " + e.pageY);
  });
// how to use
<DIV id=XY></DIV>
});

```
##  返回顶部按钮 
你可以利用 animate 和 scrollTop 来实现返回顶部的动画，而不需要使用其他插件。
```

// Back to top
$('a.top').click(function () {
  $(document.body).animate({scrollTop: 0}, 800);
  return false;
});<!-- Create an anchor tag --><a class="top" href="#">Back to top</a>

```
改变 scrollTop 的值可以调整返回距离顶部的距离，而 animate 的第二个参数是执行返回动作需要的时间(单位：毫秒)。
##  阻止链接加载 
**有时你不希望链接到某个页面或者重新加载它**，你可能希望它来做一些其他事情或者触发一些其他脚本，你可以这么做：
```

$('a.no-link').click(function (e) {
  e.preventDefault();
});

```
##  让两个 DIV 高度相同 
有时你需要让两个 div 高度相同，而不管它们里面的内容多少。可以使用下面的代码片段：
```

var $columns = $('.column');var height = 0;
$columns.each(function () {
  if ($(this).height() > height) {
    height = $(this).height();
  }
});
$columns.height(height);

```
这段代码会循环一组元素，并设置它们的高度为元素中的最大高。
##  验证元素是否为空 
This will allow you to check if an element is empty.
```

$(document).ready(function() {
  if ($('#id').html()) {
     // do something
   }
});

```
##  替换元素或重新加载某元素 
Want to replace a div, or something else?
```

$(document).ready(function() {
   $('#id').replaceWith('
<DIV>I have been replaced</DIV>
 
');
//或者这样
 $("#id").empty();
  $("#id").append('<DIV>I have been replaced</DIV>');
});

```

##   jQuery延时加载功能 
Want to delay something?
```

$(document).ready(function() {
   window.setTimeout(function() {
        // do something
   }, 1000);
});

```

##  使元素居屏幕中间位置 
Center an element in the center of your screen.
```

$(document).ready(function() {
  jQuery.fn.center = function () {
        this.css("position","absolute");
              this.css("top", ( $(window).height() - this.height() ) / 2+$(window).scrollTop() + "px");
                    this.css("left", ( $(window).width() - this.width() ) / 2+$(window).scrollLeft() + "px");
                          return this;
  }
  $("#id").center();
});

```
参考：http://www.cnblogs.com/wshiqtb/p/3522257.html
http://www.oschina.net/news/67792/35-jquery-skills
