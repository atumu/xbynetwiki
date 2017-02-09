title: jquery_raty 

#  jQuery 星级评分插件 jQuery Raty 
官网：http://www.wbotelhos.com/raty/
jQuery Raty这是一个能够自动生成可定制的星级评分jQuery插件。可以自定义图标，创建各种评级组合，星星数量，每一颗星星的注释，可以在当一个星星被点击时的加回调函数。
```

<div></div>
$('div').raty();
$('div').raty({ score: 3 });//默认为3颗星
//改变星星数量。
$('#star').raty({ number: 10 });

//您可以阻止用户投票。
$('#star').raty({ readOnly: true, score: 3 });

```
##  回调 
```

<div data-score="1"></div>
$('div').raty({
  score: function() {
    return $(this).attr('data-score');
  }
});

```
Click
Callback to handle the score and the click event on click action.
You can mension the Raty element (DOM) itself using this.
```

$('div').raty({
  click: function(score, evt) {
    alert('ID: ' + this.id + "\nscore: " + score + "\nevent: " + evt);
  }
});

```
Click Prevent
If you return false into callback, the click action will be prevented.
```

$('div').raty({
  click: function(score, evt) {
    alert('Score will not change.')
    return false;
  }
});

```

##  数字回调 
您可以收到多少分动态使用回调来设置它
```

<div id="star" data-number="3"></div>

$('#star').raty({
  number: function() {
    return $(this).attr('data-number');
  }
});

```
##  得分回调 
如果你需要启动你的分数取决于一个动态值，你可以使用回调。
你可以传递任何值，因为他们，不一定是数据值。你可以使用一个字段的值，例如。
```

<div id="star" data-score="1"></div>

$('#star').raty({
  score: function() {
    return $(this).attr('data-score');
  }
});

```

##  全局改变设置： 
你可以全局更改上述提到的所有设置 $.fn.raty.defaults.OPTION = VALUE;. 该语句必须添加在插件绑定之前。
$.fn.raty.defaults.path = assets;
$.fn.raty.defaults.cancel = true;
