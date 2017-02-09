title: jquery效果与动画 

#  jQuery效果与动画 
##  显示与隐藏 
```

$("#hide").click(function(){
  $("p").hide();
});

$("#show").click(function(){
  $("p").show();
});

```
##  淡入淡出 
```

$(document).ready(function(){
  $("button").click(function(){
    $("#div1").fadeToggle();
    $("#div2").fadeToggle("slow");
    $("#div3").fadeToggle(3000);
  });
});
$(document).ready(function(){
  $("button").click(function(){
    $("#div1").fadeIn();
    $("#div2").fadeIn("slow");
    $("#div3").fadeIn(3000);
  });
});
$(document).ready(function(){
  $("button").click(function(){
    $("#div1").fadeOut();
    $("#div2").fadeOut("slow");
    $("#div3").fadeOut(3000);
  });
});
$(document).ready(function(){
  $("button").click(function(){
    $("#div1").fadeTo("slow",0.15);
    $("#div2").fadeTo("slow",0.4);
    $("#div3").fadeTo("slow",0.7);
  });
});

```
```

<!DOCTYPE html>
<html>
<head>
<script src="http://libs.baidu.com/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
  $("button").click(function(){
    $("#div1").fadeIn();
    $("#div2").fadeIn("slow");
    $("#div3").fadeIn(3000);
  });
});
</script>
</head>
<body>
<p>Demonstrate fadeIn() with different parameters.</p>
<button>Click to fade in boxes</button>
<br><br>
<div id="div1" style="width:80px;height:80px;display:none;background-color:red;"></div><br>
<div id="div2" style="width:80px;height:80px;display:none;background-color:green;"></div><br>
<div id="div3" style="width:80px;height:80px;display:none;background-color:blue;"></div>
</body>
</html>			

```
##  滑动 
```

$(document).ready(function(){
  $("#flip").click(function(){
    $("#panel").slideUp("slow");
  });
});
$(document).ready(function(){
  $("#flip").click(function(){
    $("#panel").slideDown("slow");
  });
});
$(document).ready(function(){
  $("#flip").click(function(){
    $("#panel").slideToggle("slow");
  });
});

```
```

<!DOCTYPE html>
<html>
<head>
<script src="http://libs.baidu.com/jquery/1.10.2/jquery.min.js">
</script>
<script> 
$(document).ready(function(){
  $("#flip").click(function(){
    $("#panel").slideDown("slow");
  });
});
</script>
 
<style type="text/css"> 
#panel,#flip
{
padding:5px;
text-align:center;
background-color:#e5eecc;
border:solid 1px #c3c3c3;
}
#panel
{
padding:50px;
display:none;
}
</style>
</head>
<body>
 
<div id="flip">Click to slide down panel</div>
<div id="panel">Hello world!</div>

</body>
</html>		

```
##  动画 
jQuery ` animate() ` 方法用于创建自定义动画。
$(selector).animate({params},speed,callback);
必需的 params 参数定义形成动画的 CSS 属性。
可选的 speed 参数规定效果的时长。它可以取以下值：` "slow"、"fast" 或毫秒 `。
可选的 callback 参数是动画完成后所执行的函数名称。
**注意：jQuery.animate()不能用于处理背景颜色。**需要使用**jquery.color插件**：gitHub地址：https://github.com/jquery/jquery-color/
下面的例子演示 animate() 方法的简单应用。它把 <div> 元素往右边移动了 250 像素：
实例
```

$("button").click(function(){
  $("div").animate({left:'250px'});
});

``` 
**jQuery animate() - 使用相对值**
也可以定义相对值（该值相对于元素的当前值）。需要在值的前面加上 += 或 -=：
```

$("button").click(function(){
  $("div").animate({
    left:'250px',
    height:'+=150px',
    width:'+=150px'
  });
});

``` 
**jQuery animate() - 使用队列功能**
默认地，jQuery 提供针对动画的队列功能。
这意味着如果您在彼此之后编写多个 animate() 调用，jQuery 会创建包含这些方法调用的"内部"队列。**然后逐一运行这些 animate 调用。**
```

$("button").click(function(){
  var div=$("div");
  div.animate({height:'300px',opacity:'0.4'},"slow");
  div.animate({width:'300px',opacity:'0.8'},"slow");
  div.animate({height:'100px',opacity:'0.4'},"slow");
  div.animate({width:'100px',opacity:'0.8'},"slow");
}); 

```
```

<!DOCTYPE html>
<html>
<head>
<script src="http://libs.baidu.com/jquery/1.10.2/jquery.min.js">
</script>
<script> 
$(document).ready(function(){
  $("button").click(function(){
    var div=$("div");
    div.animate({height:'300px',opacity:'0.4'},"slow");
    div.animate({width:'300px',opacity:'0.8'},"slow");
    div.animate({height:'100px',opacity:'0.4'},"slow");
    div.animate({width:'100px',opacity:'0.8'},"slow");
  });
});
</script> 
</head>
 
<body>
<button>Start Animation</button>
<p>By default, all HTML elements have a static position, and cannot be moved. To manipulate the position, remember to first set the CSS position property of the element to relative, fixed, or absolute!</p>
<div style="background:#98bf21;height:100px;width:100px;position:absolute;">
</div>

</body>
</html>		

```

**jQuery animate() - 操作多个属性**
请注意，生成动画的过程中可同时使用多个属性：
```

$("button").click(function(){
  $("div").animate({
    left:'250px',
    opacity:'0.5',
    height:'150px',
    width:'150px'
  });
});

``` 
```

<!DOCTYPE html>
<html>
<head>
<script src="http://libs.baidu.com/jquery/1.10.2/jquery.min.js">
</script>
<script> 
$(document).ready(function(){
  $("button").click(function(){
    $("div").animate({
      left:'250px',
      opacity:'0.5',
      height:'150px',
      width:'150px'
    });
  });
});
</script> 
</head>
 
<body>
<button>Start Animation</button>
<p>By default, all HTML elements have a static position, and cannot be moved. To manipulate the position, remember to first set the CSS position property of the element to relative, fixed, or absolute!</p>
<div style="background:#98bf21;height:100px;width:100px;position:absolute;">
</div>
</body>
</html>			

```
##  停止动画 
jQuery stop() 方法
jQuery stop() 方法用于停止动画或效果，在它们完成之前。
stop() 方法适用于所有 jQuery 效果函数，包括滑动、淡入淡出和自定义动画。
语法:
$(selector).stop(stopAll,goToEnd);
```

<!DOCTYPE html>
<html>
<head>
<script src="http://libs.baidu.com/jquery/1.10.2/jquery.min.js">
</script>
<script> 

$(document).ready(function(){
  $("#start").click(function(){
    $("div").animate({left:'100px'},5000);
    $("div").animate({fontSize:'3em'},5000);
  });
  
  $("#stop").click(function(){
    $("div").stop();
  });

  $("#stop2").click(function(){
    $("div").stop(true);
  });

  $("#stop3").click(function(){
    $("div").stop(true,true);
  });
  
});
</script> 
</head>
<body>

<button id="start">Start</button>
<button id="stop">Stop</button>
<button id="stop2">Stop all</button>
<button id="stop3">Stop but finish</button>
<p>The "Start" button starts the animation.</p>
<p>The "Stop" button stops the current active animation, but allows the queued animations to be performed afterwards.</p>
<p>The "Stop all" button stops the current active animation and clears the 
animation queue; so all animations on the element is stopped.</p>
<p>The "Stop but finish" rushes through the current active animation, then it stops.</p> 

<div style="background:#98bf21;height:100px;width:200px;position:absolute;">HELLO</div>

</body>
</html>			

```
##  jQuery - Chaining 
```

$("#p1").css("color","red").slideUp(2000).slideDown(2000);

```
##  使用jQuery.color插件与jQuery.animate处理背景颜色渐变动画 
**jQuery.animate()不能用于处理背景颜色。**需要使用**jquery.color插件**：gitHub地址：https://github.com/jquery/jquery-color/
```

$(document).ready(function(){
		function changeBackground(){
			$("body").animate({backgroundColor:"#FFCCCC"},2000);
			$("body").animate({backgroundColor:"#9966CC"},2000);
			$("body").animate({backgroundColor:"#006699"},2000);
			setTimeout(function(){changeBackground();},6000);
		}
		changeBackground();
		});
});

```
**其中backgroundColor是jquery.color插件支持的属性。**