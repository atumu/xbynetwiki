title: js返回顶部按钮代码 

#  js返回顶部按钮代码 
```

<div id="topcontrol" title="回到顶部" style="position: fixed; bottom: 55px; right: 25px; cursor: pointer;z-index: 999999;display: block; opacity: 1;">
<img src="static/images/gotop.gif" style="width:31px; height:31px;">
</div>

```
##  不带特效返回顶部 
```

			// Back to top
			$('#topcontrol').click(function (e) {
			  e.preventDefault();
			  //$(document.body).animate({scrollTop: 0}, 80);
			  $(document.body).scrollTop(0);
			  
			});

```
##  带特效返回顶部 
```

// Back to top
$('#topcontrol').click(function (e) {
 e.preventDefault();
  //$(document.body).scrollTop(0);
 $(document.body).animate({scrollTop: 0}, 80);
});

```
通过使用 jQuery 中的 animate 和 scrollTop 方法，你无需插件便可创建一个简单地回到顶部动画：将 scrollTop 的值改为你想要 scrollbar 停止的地方。然后你要做的就是，设置在 80 毫秒内回到顶部。

参考OSChina首页与http://web.jobbole.com/84028/