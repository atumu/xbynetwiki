title: css水平与垂直居中 

#  css水平与垂直居中 
##  水平居中 
方式一、` text-align:center `
方式二、` margin:0 auto `。但对于margin大法也只在子元素宽度小于容器宽度时管用，当子元素宽度大于容器宽度时此法失效。` 注：margin:0 auto;这个必须是要给标签设定宽度，而且不能加浮动， `
方式三、html5元素<center></center>
方式四、translate方式
```

html,body{height:100%;margin:0;padding:0;}
.center-horizontal {
    position: relative;
    left: 50%;
    transform: translateX(-50%); 
    -webkit-transform:translateX(-50%);/*解决Android手机浏览器兼容问题*/
}

```
最简单方式:
**关键对body设置text-align:center，同时对要居中DIV设置 margin:0 auto即可实现DIV水平居中，当然不能同时再设置float浮动样式。**

水平居中的` text-align:center 和 margin:0 auto `这两种方法都是用来水平居中的，**前者是针对父元素进行设置而后者则是对子元素**。他们起作用的首要条件是**子元素必须没有被float影响**，否则一切都是无用功。margin:0 auto也可以被写成margin:0 auto 0 auto。
` 注：margin:0 auto;这个必须是要给标签设定宽度，而且不能加浮动， `这个样式居中的原理就是左右边距都自适应，**因为div默认是宽度最大化的，所以不设定宽度的话无法实现居中（可以设定%）**，而加浮动后div会变为宽度最小化并且只有左或者右浮动，所以也无法实现居中
**当然table如果不设定宽度的话，默认宽度是最小化的，加margin:0 auto;也是可以实现居中的**
###  div自适应水平居中二解 
当父元素和子元素都没有定义宽度的情况下实现水平居中： 
####  方法一 
display:inline-block 
可以使用text-align:center和display:inline-block相结合，这个技巧需要一个父元素。 
HTML代码: 
```

<div class="navbar"> 
<ul> 
<li><a href="/">Home</a></li> 
… 
</ul> 
</div>

``` 
CSS代码: 
```

.navbar { 
text-align:center; 
} 
.navbar ul { 
display:inline-block; 
} 
.navbar li { 
float:left; 
} 
.navbar li + li { 
margin-left:20px; 
}

``` 
###  方法二 
position:relative 
使用position:relative与float相结合的技巧及其浮动和定位参照物的关系，这个技巧需要两个父元素，一个用来定位而另外一个用来避免出现滚动条。 
HTML代码: 

```

<div class="navbar"> 
<div> 
<ul> 
<li><a href="/">Home</a></li> 
… 
</ul> 
</div> 
</div>

``` 

CSS代码: 

```

.navbar { 
overflow:hidden; 
} 
.navbar > div { 
position:relative; 
left:50%; 
float:left; 
} 
.navbar ul { 
position:relative; 
left:-50%; 
float:left; 
} 
.navbar li { 
float:left; 
} 
.navbar li + li { 
margin-left:20px; 
}

``` 
###  方法三 
display:table 
如果向使用极少的标签实现，这个方法是个不错的选择。 
HTML代码: 

```

<ul class="navbar"> 
<li><a href="/">Home</a></li> 
… 
</ul>

``` 

CSS代码: 
```

.navbar { 
display:table; 
margin:0 auto; 
} 
.navbar li { 
display:table-cell; 
} 
.navbar li + li { 
padding-left:20px; 
}

``` 

##  垂直居中 
**垂直居中显示某个DIV**，我们知道CSS中天然有水平居中的样式text-align:center。唯独这个垂直居中无解。
###  方式一translate方式、推荐。 
下面这个样式利用了**translate**来巧妙实现了垂直居中样式，需IE9+。
` 可能存在手机端浏览器兼容问题 `
` html,body{height:100%;margin:0;padding:0;}必须加上，否则失效 `
```

html,body{height:100%;margin:0;padding:0;}
.center-vertical {
    position: relative;
    top: 50%;
    transform: translateY(-50%);
    -webkit-transform:translateY(-50%);/*解决Android浏览器兼容问题*/
}

```
###  div垂直居中二解 
参考：http://www.divcss5.com/jiqiao/j645.shtml
http://fyting.iteye.com/blog/92437

方式一：
```

<style> 
#main {
  position: absolute;
  width:400px;
  height:200px;
  left:50%;
  top:50%; 
   margin-left:-200px;
  margin-top:-100px;
  border:1px solid #00F
 } 
</style> 
<body> 
<div id="main">DIV水平居中和上下垂直居中</div> 
</body> 

``` 
水平垂直居中原理介绍
这里使用了绝对定位position:absolute，使用left和top设置对象距离上和左为50%，但如果设置50%，实际上盒子是没有实现居中效果，所以又设置margin-left:-200px;margin-top:-100px;，这里有个技巧是，margin-left的值是宽度一半，margin-top的值也是对象高度一半，同时设置为负，这样就实现了水平和垂直居中。

**方式二：推荐：**
```

.centerDiv{
position: absolute;
top: 50%;
left: 50%;
height: 30%;
width: 30%;
margin: -15% 0 0 -15%;
}

```
http://blog.csdn.net/wolinxuebin/article/details/7615098

             
**以下补充五种方式**
####  方式二、display:table方式,不推荐。但支持自适应 
当然你可以将容器设置为` display:table `，然后将**子元素**也就是要垂直居中显示的元素设置为` display:table-cell，然后加上vertical-align:middle `来实现，**但此种实现往往会因为display：table而破坏整体布局**，那还不如直接用table标签了呢。

这个方法把一些 div 的显示方式设置为表格，因此我们可以使用表格的 vertical-align property 属性。
```

wrapper {display:table;} #middle {display:table-cell; vertical-align:middle;}

```
优点：
content 可以动态改变高度(不需在 CSS 中定义)。当 wrapper 里没有足够空间时， content 不会被截断
缺点：
Internet Explorer(甚至 IE8 beta)中无效
` **注意以下几项一个都不能少，否则无效** `
```

	html,body{height:100%;margin:0;padding:0;}
	.column{height:100%;position:relative;}
	#wrapper{display:table;height:100%;padding:3.7em 10px 10px 10px;}
	#middle{display:table-cell;vertical-align:middle;}
	.content{}

```
```

<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title>aaa</title>
	<link rel="stylesheet" href="">
	<style type="text/css">
	html,body{height:100%;margin:0;padding:0;}
	#wrapper{display:table;height:100%;padding:3.7em 10px 10px 10px;}
	#middle{display:table-cell;vertical-align:middle;}
	.column{height:100%;position:relative;}
	.content{}
	</style>
</head>
<body>
	
<div class="column">
<div id="wrapper">
<div id="middle">
<div class="content">
<ul>
<li class="good">Everything in this table cell div is centred</li>
<li class="good">It can dynamically change to any height based on content (try by changing your browser font size)</li>
<li class="good">Doesn&rsquo;t get cut off when there isn't enough room</li>
<li class="bad">Doesn&rsquo;t work in Internet Explorer</li>
<li class="bad">Uses 2 extra s</li>
</ul>
</div>
</div>
</div>
</div>
</body>

</html>

```
####  方法三 不支持自适应。不推荐
这个方法使用绝对定位的 div，把它的 top 设置为 50％，top margin 设置为负的 content 高度。这意味着对象必须在 CSS 中指定固定的高度。
因为有固定高度，或许你想给 content 指定 overflow:auto，这样如果 content 太多的话，就会出现滚动条，以免content 溢出。
```

content { position:absolute; top:50%; height:240px; margin-top:-120px; /* negative half of the height */ }

```
优点：
适用于所有浏览器
不需要嵌套标签
缺点：
没有足够空间时，content 会消失(类似div 在 body 内，当用户缩小浏览器窗口，滚动条不出现的情况)
####  方法四、支持有限的自适应，但必须显式限定高度 
这种方法，在 content 元素外插入一个 div。设置此 div height:50%; margin-bottom:-contentheight;。
content 清除浮动，并显示在中间。
```

html,body{height:100%;margin:0;padding:0;}
floater {float:left; height:50%; margin-bottom:-120px;} #content {clear:both; height:240px; position:relative;}

```
实例
` **注意：floater标记的必须是个空的div元素。content标记不能有父元素** `
```

<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<title>aaa</title>
	<link rel="stylesheet" href="">
	<style type="text/css">
html,body{height:100%;margin:0;padding:0;}
#float {float:left; height:50%; margin-bottom:-120px;} .content {clear:both; height:240px; position:relative;}
	</style>
</head>
<body>
	

<div id="float"></div> <!--注意这里是个元素-->
<div class="content"><!--content标记不能有父元素-->
<ul>
<li class="good">Everything in this table cell div is centred</li>
<li class="good">It can dynamically change to any height based on content (try by changing your browser font size)</li>
<li class="good">Doesn&rsquo;t get cut off when there isn't enough room</li>
<li class="bad">Doesn&rsquo;t work in Internet Explorer</li>
<li class="bad">Uses 2 extra</li>
</ul>
</div>

</body>

</html>

```
优点：
适用于所有浏览器
没有足够空间时(例如：窗口缩小) content 不会被截断，滚动条出现

缺点：
唯一我能想到的就是需要额外的空元素了
####  方法五 
这个方法使用了一个 position:absolute，有固定宽度和高度的 div。这个 div 被设置为 top:0; bottom:0;。但是因为它有固定高度，其实并不能和上下都间距为 0，因此 margin:auto; 会使它居中。使用 margin:auto;使块级元素垂直居中是很简单的。
```

content { position:absolute; top:0; bottom:0; left:0; right:0; margin:auto; height:240px; width:70%; }

```
优点：
简单

缺点：
IE(IE8 beta)中无效
无足够空间时，content 被截断，但是不会有滚动条出现
[测试页面](http://douglasheriot.com/tutorials/css_vertical_centre/demo4.html)


##  示列：已知div高度情况下水平垂直居中 
参考：http://web.jobbole.com/85285/
在这之前，**我们先要设置div元素的祖先元素html和body的高度为100%（因为他们默认是为0的），并且清除默认样式，即把margin和padding设置为0**（如果不清除默认样式的话，浏览器就会出现滚动条）。
```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index</title>
    <style>
        html,body {
            width: 100%;/*注意*/
            height: 100%;/*注意*/
            margin: 0;/*注意*/
            padding: 0;/*注意*/
        }
        .content {
            width: 300px;
            height: 300px;
            background: orange;
            margin: 0 auto; /*水平居中*/
            position: relative; /*脱离文档流*/
            top: 50%; /*偏移*/
            margin-top: -150px;  /* transform: translateY(-50%);*/
        }
    </style>
</head>
<body>
    <div class="content"></div>
</body>
</html>

```

参考：
http://www.cnblogs.com/Wayou/p/things_you_dont_know_about_frontend.html
http://web.jobbole.com/83839/
