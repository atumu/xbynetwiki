title: jquery内容滚动插件_flexslider 

#  jquery内容滚动插件_flexslider 
FlexSlider是一个轻量级的jQuery内容滚动插件，压缩后只有4KB大小。它能让你轻松的创建内容滚动的效果，具有非常高的可定制性。它是将UL列表转换成内容滚动的列表，可以自动播放，或者使用导航按钮和键盘来控制。
官网：http://www.woothemes.com/flexslider/
demo:http://flexslider.woothemes.com/
使用:
导入head:
```

<script src="http://cdn.bootcss.com/jquery/1.11.3/jquery.min.js"></script>
<script defer src="index/jquery.flexslider.js"></script>
<link rel="stylesheet" href="index/flexslider.css"  />

```
JS代码加载：
```

// Can also be used with $(document).ready()
$(window).load(function() {
  $('.flexslider').flexslider({
    animation: "slide"
  });
});

```
HTML代码：
```

<!-- Place somewhere in the <body> of your page -->
<div class="flexslider">
  <ul class="slides">
    <li>
      1page
    </li>
    <li>
      2page
    </li>
    <li>
      3page
    </li>
    <li>
     4page
    </li>
  </ul>
</div>

```
##  FlexSlider插件的详细设置参数 
```

$(window).load(function() {
    $('.flexslider').flexslider({

        namespace: 'flex-',    //控件的命名空间，会影响样式前缀 
        animation: "slide", //String: Select your animation type, "fade" or "slide"图片变换方式：淡入淡出或者滑动
        slideDirection: "horizontal", //String: Select the sliding direction, "horizontal" or "vertical"图片设置为滑动式时的滑动方向：左右或者上下
        
        selector: '.thumbnails .thumbnail',
        slideshowSpeed: 5000, // 自动播放速度毫秒
        animationSpeed: 600, //滚动效果播放时长
        pausePlay: false,//是否显示播放暂停按钮
        minItems: common.globals.SCREEN.ITEM,//最少显示多少项
        itemWidth: 220,//一个滚动项目的宽度
        itemMargin: 20,//滚动项目之间的间距
        slideshow: true, //Boolean: Animate slider automatically 载入页面时，是否自动播放
        animationDuration: 600, //Integer: S动画淡入淡出效果延时
        directionNav: true, //Boolean:  (true/false)是否显示左右控制按钮
        controlNav: true, //Boolean:  usage是否显示控制菜单//什么是控制菜单？
        keyboardNav: true, //Boolean:left/right keys键盘左右方向键控制图片滑动
        mousewheel: false, //Boolean: mousewheel鼠标滚轮控制制图片滑动
        prevText: "Previous", //String: 上一项的文字
        nextText: "Next", //String: 下一项的文字
        pauseText: 'Pause', //String: 暂停文字
        playText: 'Play', //String: 播放文字
        randomize: false, //Boolean: Randomize slide order 是否随机幻灯片
        slideToStart: 0, //Integer:  (0 = first slide)初始化第一次显示图片位置
        animationLoop: true, //  "disable" classes at either end 是否循环滚动 循环播放
        pauseOnAction: true, //Boolean:  highly recommended.
        pauseOnHover: false, //Boolean: ng
        controlsContainer: "", //Selector:  be taken.
        manualControls: ".js-slidernav i", //Selector: .自定义控制导航// 小圆点活数字标示 css 选择器        
        manualControlEvent: "", //String:自定义导航控制触发事件:默认是click,可以设定hover
        start: function() {}, //Callback: function(slider) - Fires when the slider loads the first slide
        before: function() {}, //Callback: function(slider) - Fires asynchronously with each slider animation
        after: function() {}, //Callback: function(slider) - Fires after each slider animation completes
        end: function() {} //Callback: function(slider) - Fires when the slider reaches the last slide (asynchronous)
        
    });
});

```