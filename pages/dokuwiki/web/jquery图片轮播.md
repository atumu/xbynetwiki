title: jquery图片轮播 

#  jquery图片轮播 
##  第一种： 
http://s-232923.gotocdn.com/
![](/data/dokuwiki/web/pasted/20151218-080353.png)
按钮图
![](/data/dokuwiki/web/pasted/20151218-145306.png)
CSS：
```

/*焦点图*/ 
/*yx_rotaion*/
.yx-rotaion-btn,.yx-rotaion-title,.yx-rotation-focus,.yx-rotation-t,.yx-rotaion-btn{position:absolute}
.yx-rotation-title{position:absolute;width:100%;height:40px;line-height:40px;background:#000;filter:alpha(opacity=40);-moz-opacity:0.4;-khtml-opacity:0.4;opacity:0.4;left:0;bottom:0;_bottom:-1px;z-index:1}
.yx-rotation-t{color:#fff !important;font-size:16px;font-family:microsoft yahei;z-index:2;bottom:0;left:10px;line-height:40px}
.yx-rotation-focus span,.yx-rotaion-btn span{background:url(images/ico.png) no-repeat;display:block;}
.yx-rotation-focus{height:40px;line-height:40px;right:20px;bottom:0;z-index:2}
.yx-rotation-focus span{width:12px;height:12px;line-height:12px;float:left;margin-left:5px;position:relative;top:14px;cursor:pointer;background-position:-24px -126px;text-indent:-9999px}
.yx-rotaion-btn{width:100%;height:41px;top:50%;margin-top:-20px;}
.yx-rotaion-btn span{width:41px;height:41px;cursor:pointer;filter:alpha(opacity=30);-moz-opacity:0.3;-khtml-opacity:0.3;opacity:0.3;position:relative}
.yx-rotaion-btn .left_btn{background-position:-2px -2px;float:left;left:10px}
.yx-rotaion-btn .right_btn{background-position:-2px -49px;float:right;right:10px}
.yx-rotaion-btn span.hover{filter:alpha(opacity=80);-moz-opacity:0.8;-khtml-opacity:0.8;opacity:0.8}
.yx-rotation-focus span.hover{background-position:-10px -126px}
.rotaion_list{width:0;height:0;overflow:hidden;}
.rotaion_list li{width: 100%;}

```
JS
```

(function($){   
    $.fn.extend({     
         yx_rotaion: function(options) {   
          //默认参数
          var defaults = {
          /**轮换间隔时间，单位毫秒*/
          during:4000,
          /**是否显示左右按钮*/
          btn:true,
          /**是否显示焦点按钮*/
          focus:true,
          /**是否显示标题*/
          title:true,
          /**是否自动播放*/
          auto:true        
        }        
        var options = $.extend(defaults, options);   
        return this.each(function(){
        var o = options;   
        var curr_index = 0;
        var $this = $(this);        
        var $li = $this.find("li");
        var li_count = $li.length;
        $this.css({position:'relative',overflow:'hidden',width:$li.find("img").width(),height:$li.find("img").height()});
        $this.find("li").css({position:'absolute',left:0,top:0}).hide();
        $li.first().show();
        $this.append('<div class="yx-rotaion-btn"><span class="left_btn"><\/span><span class="right_btn"></span><\/div>');
        if(!o.btn) $(".yx-rotaion-btn").css({visibility:'hidden'});
        if(o.title) $this.append(' <div class="yx-rotation-title"><\/div><a href="" class="yx-rotation-t"><\/a>');
        if(o.focus) $this.append('<div class="yx-rotation-focus"><\/div>');
        var $btn = $(".yx-rotaion-btn span"),$title = $(".yx-rotation-t"),$title_bg = $(".yx-rotation-title"),$focus = $(".yx-rotation-focus");
        //如果自动播放，设置定时器
        if(o.auto) var t = setInterval(function(){$btn.last().click()},o.during);
        $title.text($li.first().find("img").attr("alt")); 
        $title.attr("href",$li.first().find("a").attr("href"));       
           // 输出焦点按钮
           for(i=1;i<=li_count;i++){
             $focus.append('<span>'+i+'</span>');
           }
           // 兼容IE6透明图片   
           if($.browser.msie && $.browser.version == "6.0" ){
              $btn.add($focus.children("span")).css({backgroundImage:'url(images/ico.gif)'});
           }    
           var $f = $focus.children("span");
           $f.first().addClass("hover");
           // 鼠标覆盖左右按钮设置透明度
           $btn.hover(function(){
            $(this).addClass("hover");
           },function(){
            $(this).removeClass("hover");
           });
            //鼠标覆盖元素，清除计时器
           $btn.add($li).add($f).hover(function(){
            if(t) clearInterval(t);
           },function(){
            if(o.auto) t = setInterval(function(){$btn.last().click()},o.during);
           });
            //鼠标覆盖焦点按钮效果
           $f.bind("mouseover",function(){
           var i = $(this).index();
           $(this).addClass("hover");
           $focus.children("span").not($(this)).removeClass("hover");
           $li.eq(i).fadeIn(300);
             $li.not($li.eq(i)).fadeOut(300); 
           $title.text($li.eq(i).find("img").attr("alt"));
           curr_index = i;
           });
           //鼠标点击左右按钮效果
           $btn.bind("click",function(){
             $(this).index() == 1?curr_index++:curr_index--;
           if(curr_index >= li_count) curr_index = 0;
           if(curr_index < 0) curr_index = li_count-1;
             $li.eq(curr_index).fadeIn(300);
           $li.not($li.eq(curr_index)).fadeOut(300);  
           $f.eq(curr_index).addClass("hover");
           $f.not($f.eq(curr_index)).removeClass("hover");
           $title.text($li.eq(curr_index).find("img").attr("alt"));
           $title.attr("href",$li.eq(curr_index).find("a").attr("href")); 
           });
          });   
        }   
    });   
})(jQuery);

```
```

  <!--轮播图片-->
  <div id="banner"  class="yx-rotaion">
  		<ul  class="rotaion_list">
  			 
  				<li>
  					<a target="" href="http://www.7yex.com"><img src="http://i1.tietuku.com/a194ed457f297381.png" alt="既不回头，何必不忘。既然无缘，何须誓言。" width="881" height="300" /></a>
  				</li>
			 
  				<li>
  					<a target="" href="http://www.7yex.com"><img src="http://i3.tietuku.com/e449e3d45d50d74c.jpg" alt="恍惚中，时光停滞，岁月静好。宛如十年前。" width="881" height="300" /></a>
  				</li>
			  		</ul>
  </div>
  <script type="text/javascript" src="http://content/templates/black_memory v1.1//js/jquery.yx_rotaion.js"></script> 
  <script type="text/javascript">
	$(".yx-rotaion").yx_rotaion({auto:true});
  </script> 

```

##  第二种 
![](/data/dokuwiki/web/pasted/20151225-171410.png)
```

.navpic-css {
	position: relative;
	font-family: '微软雅黑';
	font-size: 14px;
}
.navpic-css ul{
	margin: 0;
	padding: 0;
	list-style: none;
}
.navpic-css .navpic-nav {
	position: absolute;
	z-index: 2;
	bottom: 5px;
	right: 10px;
	transition: all .4s;
	-moz-transition: all .4s;	/* Firefox 4 */
	-webkit-transition: all .4s;	/* Safari 和 Chrome */
	-o-transition: all .4s;	/* Opera */
}
.navpic-css .navpic-nav-item{
	display: inline-block;
	width: 15px;
	height: 15px;
	border-radius: 15px;
	border: 2px solid #fff;
	cursor: pointer;
}
.navpic-css .navpic-nav-item.active{
	background: #fff;
}
@media (max-width: 768px){
	.navpic-css .navpic-nav-item{
		width: 10px;
		height: 10px;
	}
}
.navpic-css .navpic-btn{
	position: absolute;
	z-index: 2;
	top: 50%;
	height: 50px;
	width: 50px;
	line-height: 50px;
	margin-top: -25px;
	font-size: 43px;
	text-align: center;
	border-radius: 25px;
	color: #fff;
	opacity: .5;
	filter: alpha(opacity=50);
	transition: all .4s;
	-moz-transition: all .4s;	/* Firefox 4 */
	-webkit-transition: all .4s;	/* Safari 和 Chrome */
	-o-transition: all .4s;	/* Opera */
}
.navpic-css .navpic-btn:hover{
	opacity: 1;
	filter: alpha(opacity=100);
}
.navpic-css:hover .navpic-btn.prev{
	margin-left: -10px;
}
.navpic-css:hover .navpic-btn.next{
	margin-right: -10px;
}
.navpic-css .navpic-btn.prev{
	left: 10px;
}
.navpic-css .navpic-btn.next{
	right: 10px;
}
@media (max-width: 768px){
	.navpic-css .navpic-btn{
		font-size: 30px;
	}
}
.navpic-css .navpic-caption{
	position: absolute;
	z-index: 2;
	left: 0;
	right: 0;
	bottom: 0;
	height: 36px;
	line-height: 36px;
	padding: 0 10px;
}
.navpic-css .navpic-caption a{
	color: #fff;
}
.navpic-css .navpic-caption-bg{
	z-index: 1;
	position: absolute;
	left: 0;
	right: 0;
	bottom: 0;
	height: 36px;
	background: #000;
	opacity: .4;
	filter:alpha(opacity=40);
}
.navpic-css .navpic-list{
	position: relative;
	overflow: hidden;
	z-index: 1
}
.navpic-css .navpic-list-item > a{
	display: block;
	height: auto;
}
.navpic-css .navpic-list-item img{
	width: 100%;
	height: auto;
}
/* 处理slide的样式相关 */
.navpic-css .navpic-list-item{
	display: none;
	position: relative;
	transition: left 1s ease-in-out;
	-moz-transition: left 1s ease-in-out;
	-webkit-transition: left 1s ease-in-out;
	-o-transition: left 1s ease-in-out;
}
.navpic-css .navpic-list-item.active{
	display: block;
	left: 0;
}
.navpic-css .navpic-list-item.next, .navpic-css .navpic-list-item.prev{
	display: block;
	position: absolute;
	width: 100%;
	top: 0;
}
.navpic-css .navpic-list-item.next{
	left: 100%;
}
.navpic-css .navpic-list-item.prev{
	left: -100%;
}
.navpic-css .navpic-list-item.next.left, .navpic-css .navpic-list-item.prev.right{
	left: 0;
}
.navpic-css .navpic-list-item.active.left{
	left: -100%;
}
.navpic-css .navpic-list-item.active.right{
	left: 100%;
}


```
```

define(function () {
	function Navpic(selector) {
		var selector = selector ? (selector + " .navpic-css") : ".navpic-css";
		var defaultOpts = {
			auto: true 		//自动播放
		}

		$(selector).each(function(){
			var _this = this,
				$this = $(_this);
			
			var options = $({}, defaultOpts);
			_this.options = $.extend(options, $this.data());//将dom上的data放入options中

			_this.$items = $this.find(".navpic-list-item");
			_this.$navs = $this.find(".navpic-nav-item");
			_this.$btns = $this.find(".navpic-btn");
			_this.current = $this.find(".navpic-list-item.active").index();
			_this.animate = false;

			Navpic.onload.call(_this, function(){
				//初始化事件
				Navpic.bindEvt.call(_this);
				//初始化自动播放
				_this.options.auto && Navpic.startAutoPlay.call(_this);
			})
		});
	}

	Navpic.onload = function(callback) {
		var _this = this;
		var $items = _this.$items;

		var i = 0;
		var length = $items.length;
		$items.each(function(){
			var image = new Image();
			image.src = $(this).find("img").attr("src");
			image.onload = function() {
				i++;
				if (i==length) {
					callback && callback();
				}
			}
		})
	}

	Navpic.startAutoPlay = function() {
		var _this = this;

		_this.autoPlay = setInterval(function(){
			Navpic.next.call(_this);
		}, 4000);
	}

	Navpic.endAutoPlay = function() {
		var _this = this;

		clearInterval(_this.autoPlay);
		_this.autoPlay = null;
	}

	Navpic.bindEvt = function() {
		var _this = this;
		var $this = $(_this);
		var $navs = _this.$navs;
		var $btns = _this.$btns;

		$navs.on("click", function(){
			var index = $(this).index();
			Navpic.goTo.call(_this, index);
		});

		$this.on("mouseenter", function(){
			Navpic.endAutoPlay.call(_this);
		}).on("mouseleave", function(){
			Navpic.startAutoPlay.call(_this);
		});

		$btns.on("click", function() {
			var type = $(this).data("type");
			Navpic[type].call(_this);
		})
	}

	Navpic.next = function() {
		var _this = this;
		var $items = _this.$items;
		var current = _this.current;

		var next = current == ($items.length-1) ? 0 : (current + 1);
		Navpic.play.call(_this, next, "next");
	}

	Navpic.prev = function() {
		var _this = this;
		var $items = _this.$items;
		var current = _this.current;

		var prev = current == 0 ? ($items.length - 1) : (current - 1);
		Navpic.play.call(_this, prev, "prev");
	}

	Navpic.goTo = function(index) {
		var _this = this;
		var current = _this.current;

		var type = index > current ? "next" : "prev";
		Navpic.play.call(_this, index, type);
	}

	Navpic.play = function(index, type) {
		var _this = this;
		var $items = _this.$items;
		var $navs = _this.$navs;
		var current = _this.current;

		if (index == current || _this.animate) return;

		_this.animate = true;

		var $active = $($items[current]);
		var $index = $($items[index]);

		type = type || "next";
		var direction = type == "next" ? "left" : "right";

		var timeout = $active.css("transition-duration");
		timeout = timeout.substr(0, timeout.length - 1);

		//初始化下一个
		$index.addClass(type);
		$index[0].offsetWidth;//强制浏览器重绘（这样css动画才能执行）
		//移动
		$active.addClass(direction);
		$index.addClass(direction);
		//设置nav-item
		$navs.removeClass("active");
		$($navs[index]).addClass("active");

		setTimeout(function(){
			$active.removeClass("active" + " " + direction);
			$index.toggleClass("active" + " " + type + " " + direction);
			_this.current = index;
			_this.animate = false;
		}, timeout * 1000);
	}
	
	return Navpic;
});

```
```

<div class="navpic-css" data-auto="true">
	<ul class="navpic-list">
		<li class="navpic-list-item">
			<a href="#">
				<img src="![](/data/dokuwikiurl)001.jpg">
			</a>
			<div class="navpic-caption">
				<a href="#">这是图片说明</a>
			</div>
			<div class="navpic-caption-bg"></div>
		</li>
		<li class="navpic-list-item active">
			<a href="#">
				<img src="![](/data/dokuwikiurl)banner.jpg">
			</a>
			<div class="navpic-caption">
				<a href="#">这是图片说明</a>
			</div>
			<div class="navpic-caption-bg"></div>
		</li>
		<li class="navpic-list-item">
			<a href="#">
				<img src="![](/data/dokuwikiurl)001.jpg">
			</a>
			<div class="navpic-caption">
				<a href="#">这是图片说明</a>
			</div>
			<div class="navpic-caption-bg"></div>
		</li>
		<li class="navpic-list-item">
			<a href="#">
				<img src="![](/data/dokuwikiurl)banner.jpg">
			</a>
			<div class="navpic-caption">
				<a href="#">这是图片说明</a>
			</div>
			<div class="navpic-caption-bg"></div>
		</li>
	</ul>
	<ul class="navpic-nav">
		<li class="navpic-nav-item"></li>
		<li class="navpic-nav-item active"></li>
		<li class="navpic-nav-item"></li>
		<li class="navpic-nav-item"></li>
	</ul>
	<a href="javascript:;" class="navpic-btn prev" data-type="prev"><i class="fa fa-chevron-left"></i></a>
	<a href="javascript:;" class="navpic-btn next" data-type="next"><i class="fa fa-chevron-right"></i></a>
</div>

```