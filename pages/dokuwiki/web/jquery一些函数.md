title: jquery一些函数 

#  jQuery一些函数 
##  $.each() 

一个通用的迭代函数，它可以用来无缝**迭代对象和数组**。数组和类似数组的对象通过一个长度属性（如一个函数的参数对象）来迭代数字索引，从0到length - 1。其他对象通过其属性名进行迭代。
jQuery.each(array, callback )
jQuery.each( object, callback )
```

$.each([52, 97], function(index, value) {
  alert(index + ': ' + value);
});

var obj = {
  "flammable": "inflammable",
  "duh": "no duh"
};
$.each( obj, function( key, value ) {
  alert( key + ": " + value );
});

```

##  $("").each() 

注意与$.each的区别。遍历一个jQuery对象，为每个匹配元素执行一个函数。
.each( function(index, Element) )
```

<ul>
    <li>foo</li>
    <li>bar</li>
</ul>

$( "li" ).each(function( index ) {
  console.log( index + ": "" + $(this).text() );
});
  
列表中每一项会显示在下面的消息中：
0: foo 
1: bar

```

##  $("").find() 
通过一个选择器，jQuery对象，或元素过滤，得到当前匹配的元素集合中**每个**元素的**后代**。
.find( selector )
**如果一个jQuery对象表示一个DOM元素的集合**， ` .find() `方法允许我们能够通过查找DOM树中的这些元素的**后代元素**，匹配的元素将**构造一个新的jQuery对象**。.find()和.children()方法是相似的，但后者只是再DOM树中向下遍历一个层级（愚人码头注：就是只查找子元素，而不是后代元素）。
```

<ul class="level-1">
  <li class="item-i">I</li>
  <li class="item-ii">II
    <ul class="level-2">
      <li class="item-a">A</li>
      <li class="item-b">B
        <ul class="level-3">
          <li class="item-1">1</li>
          <li class="item-2">2</li>
          <li class="item-3">3</li>
        </ul>
      </li>
      <li class="item-c">C</li>
    </ul>
  </li>
  <li class="item-iii">III</li>
</ul>
  
如果我们从item II 开始，我们可以找到它里面的清单项目：
$('li.item-ii').find('li').css('background-color', 'red');


```
该调用的结果是II项的A，B，1，2，3，和C的背景变为红色，尽管item II匹配选择表达式，它不包括在结果中; **只有它的` 后代 `被认为是匹配的候选元素。**

##  $("").data() 
.data( key, value )返回: jQuery
描述: 在匹配元素上存储任意相关数据.
.data( obj )
一个用于更新数据的 键/值对
.data() 方法允许我们在DOM元素上绑定任意类型的数据,避免了循环引用的内存泄漏风险。
我们可以在一个元素上设置不同的值，之后获取这些值：
```

$("body").data("foo", 52);
$("body").data("bar", { myType: "test", count: 40 });
$("body").data({ baz: [ 1, 2, 3 ] });
 
$("body").data("foo"); // 52
$("body").data(); // { foo: 52, bar: { myType: "test", count: 40 }, baz: [ 1, 2, 3 ] }

```
示列：从div元素储存然后找回一个值。
```

<!DOCTYPE html>
<html>
<head>
  <style>
  div { color:blue; }
  span { color:red; }
  </style>
  <script src="http://code.jquery.com/jquery-latest.js"></script>
</head>
<body>
  <div>
    The values stored were
    <span></span>
    and
    <span></span>
  </div>
<script>
$("div").data("test", { first: 16, last: "pizza!" });
$("span:first").text($("div").data("test").first);
$("span:last").text($("div").data("test").last);
</script>
 
</body>
</html>

```
##  $.extend() 
jQuery.extend( target [, object1 ] [, objectN ] )返回: Object
描述: 将两个或更多对象的内容合并到第一个对象。
  * target
  * 类型: Object
  * 一个对象，如果附加的对象被传递给这个方法将那么它将接收新的属性，如果它是唯一的参数将扩展jQuery的命名空间。
  * object1
  * 类型: Object
  * 一个对象，它包含额外的属性合并到第一个参数
  * objectN
  * 类型: Object
  * 包含额外的属性合并到第一个参数

jQuery.extend( [deep ], target, object1 [, objectN ] )
  * deep
  * 类型: Boolean
  * 如果是true，合并成为递归（又叫做深拷贝）。

**Example: 合并两个对象，并修改第一个对象。**
```

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>jQuery.extend demo</title>
  <script src="https://code.jquery.com/jquery-1.10.2.js"></script>
</head>
<body>
 
<div id="log"></div>
 
<script>
var object1 = {
  apple: 0,
  banana: { weight: 52, price: 100 },
  cherry: 97
};
var object2 = {
  banana: { price: 200 },
  durian: 100
};
 
// Merge object2 into object1
$.extend( object1, object2 );
 
// Assuming JSON.stringify - not available in IE<8
$( "#log" ).append( JSON.stringify( object1 ) );
</script>
 
</body>
</html>

```
输出：{"apple":0,` "banana":{"price":200} `,"cherry":97,"durian":100}
如果脚本改为
```

// Merge object2 into object1, recursively
$.extend( true, object1, object2 );
 
// Assuming JSON.stringify - not available in IE<8
$( "#log" ).append( JSON.stringify( object1 ) );

```
输出：{"apple":0,` "banana":{"weight":52,"price":200} `,"cherry":97,"durian":100}

##  $.when() 
` jQuery.when( deferreds ) `返回: Promise
描述: 提供一种方法来执行一个或多个对象的回调函数， Deferred(延迟)对象通常表示异步事件。
deferreds 类型: Deferred
**一个或多个延迟对象**，或者普通的JavaScript对象。

如果向 jQuery.when() 传入一个单独的延迟对象，那么会返回它的 Promise 对象(延迟方法的一个子集)。可以继续绑定 Promise 对象的其它方法，例如， ` defered.then ` 。当延迟对象已经被解决（resolved）或被拒绝(rejected）（通常是由创建延迟对象的最初代码执行的），那么就会调用适当的回调函数。
```

$.when( $.ajax("test.aspx") ).then(function(data, textStatus, jqXHR){
     alert( jqXHR.status ); // alerts 200
});

```
在多个延迟对象传递给` jQuery.when() ` 的情况下，该方法根据一个新的“宿主” Deferred（延迟）对象，**跟踪所有已通过Deferreds聚集状态**，返回一个Promise对象。**当所有的延迟对象被解决（resolve）时，“宿主” Deferred（延迟）对象才会解决（resolved）该方法，或者当其中有一个延迟对象被拒绝（rejected）时，“宿主” Deferred（延迟）对象就会reject（拒绝）该方法。**如果“宿主” Deferred（延迟）对象是（resolved）` 解决状态时 `， “宿主” Deferred（延迟）对象的 ` done() ` （解决回调）将被执行。参数传递给 doneCallbacks提供这解决（resolved）值给每个对应的Deferreds对象，并匹配Deferreds传递给 jQuery.when()的顺序。 例如：
Example: 执行Ajax请求后两个函数是成功的。（见jQuery.ajax()对于一个成功的和错误的案件为AJAX请求的完整描述文档）。
```

$.when($.ajax("/page1.php"), $.ajax("/page2.php")).done(function(a1,  a2){
  /* a1 and a2 are arguments resolved for the
      page1 and page2 ajax requests, respectively */
  var jqXHR = a1[2]; /* arguments are [ data, statusText, jqXHR ] */
  if ( /Whip It/.test(jqXHR.responseText) ) {
    alert("First page has 'Whip It' somewhere.");
  }
});

```
```

Example: 执行函数myFunc当两个Ajax请求是成功的，如果任一或myFailure有一个错误。
$.when($.ajax("/page1.php"), $.ajax("/page2.php"))
  .then(myFunc, myFailure);

```