title: js技巧总结 

#  JS技巧总结 

##  js验证用户输入的只能是数字？ 
```

function check_validate1(value){
    //定义正则表达式部分,注意以数字开头结尾。
    var reg = /^\d+$/;
    if( value.constructor === String ){
        //var re = value.match( reg );
        //return true;
      return reg.test(value);
    }
    return false;
}

```
##  js中json与数组字符串的相互转化 
**js对象转json字符串:**
方式一:
var jsonText = JSON.stringify(obj);
方式二:
var jsonText=obj.toJSONString();
**js字符串转换为json对象**：
方式一:通过eval() 函数可以将JSON字符串转化为对象
var obj = eval(t3);
方式二:直接将字符串赋值给变量:
var data=jsonText;
方式三:
var obj=JSON.parse(jsonText);

##  Hover 上的 Class 切换 
如果用户的鼠标悬停在页面上某个可点击元素时，你想要改变这个元素的视觉表现。可以使用下面这段代码，当用户悬停时，为该元素增加一个 class；当用户鼠标离开后移除这个 class：
```

$('.btn').hover(function () {
  $(this).addClass('hover');
}, function () {
  $(this).removeClass('hover');
});

```
你仅需增加必须的 CSS。如果需要更简单的方式，还可以使用 toggleClass 方法：
```

$('.btn').hover(function () {
  $(this).toggleClass('hover');
});

```
注意：CSS 或许是这个例子更快速的解决方式，但大家仍然值得知道这一点。

##  停止链接加载 
有时你不想链接跳转到某个页面或重加载该页面，而希望可以做一些其他事情，比如触发其他脚本。下面的代码是禁止默认行为的一个小诀窍：
```

$('a.no-link').click(function (e) {
  e.preventDefault();
});

```

##  淡入淡出/滑动开关 
淡入淡出与滑动是我们经常使用 jQuery 做成的动画效果。或许你只是想在用户点击某物时展现一个元素，使用 ` fadeIn 和 slideDown ` 都很棒。但如果想让该元素在第一次点击时显现，第二次点击时消失，下面的代码可以很好地完成这个工作：
```

// Fade
$('.btn').click(function () {
  $('.element').fadeToggle('slow');
});
 
// Toggle
$('.btn').click(function () {
  $('.element').slideToggle('slow');
});

```
##  jquery同时监听点击和enter键事件 
```

this.$element.find(".inputPageNo").on("click keypress",function(e){
            	 var isReturn=true; 
            	 if(e.type=="keypress" && e.keyCode == "13")    
                 {
                    isReturn =false;
                 }
                 if(e.type=="click"){
            		 isReturn =false;
            	 }
            	 if(isReturn){
            		 return true;
            	 }

```
##  JS判断对象是否为空 
http://blog.csdn.net/yiluoak_47/article/details/7766760
```

/*
 *
 检测对象是否是空对象(不包含任何可读属性)。
 *
 方法既检测对象本身的属性，也检测从原型继承的属性(因此没有使hasOwnProperty)。
 */
function isEmpty(obj)
{
    for (var name in obj)
    {
        return false;
    }
    return true;
};

```
```

/*
 *
 检测对象是否是空对象(不包含任何可读属性)。
 *
 方法只既检测对象本身的属性，不检测从原型继承的属性。
 */
function isOwnEmpty(obj)
{
    for(var name in obj)
    {
        if(obj.hasOwnProperty(name))
        {
            return false;
        }
    }
    return true;
};

```

##  JS自定义CommonTool 
```

define([],function(){
	function CommonTool(){

	}
	CommonTool.isEmpty=function(obj){
		for(var name in obj){
			return false;
		}
		return true;
	}
	/*
		@param search：要搜索的字符串
		@param need 原始字符串	
		@param pos 第几次出现，必须为正整数，或者-1，代表最后一次出现的位置
		@return 如果指定了pos,则返回第$pos次出现的位置，否则返回第一次出现的位置。-1代表没有找到。
	 */
	CommonTool.indexOf=function(search,need,pos){
		if(!pos || pos<-1 || pos==0) pos=1;
		if(need.indexOf(search)==-1) return -1;
		var strs=need.split(search);
		if((pos+1)>strs.length) pos=-1;
		if(pos==-1){
			var base=0;
			strs.pop();
			if(strs.length==0||(strs[0]=="" && strs.length==1)) return 0;
			return strs.join(search).length;
		}
		if(strs[0]==""&&pos==1){
			return 0;
		}else{
			var tmpLen=strs.slice(0,pos).join(search).length;
			return tmpLen;
		}

	}
	return CommonTool;
});

```