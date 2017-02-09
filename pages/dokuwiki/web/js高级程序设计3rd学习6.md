title: js高级程序设计3rd学习6 

#  JS高级程序设计3rd学习6BOM和客户端检测 
ES是JavaScript的核心，但如果要在web中使用javascript，那么BOM(浏览器对象模型)则无疑才是真正的核心。BOM提供了很多对象，用于访问浏览器的功能。目前已经将BOM的主要方面纳入了HTML5的规范中。
##  window对象 
BOM的核心对象是window，它表示浏览器的一个实例。在浏览器中，window对象有双重角色，它既是通过javascript访问浏览器窗口的一个接口，又是ES规则的Global对象。因此有权访问parseInt()等方法。
###  窗口关系及frame,iframe 
如果页面中包含frame，则每个frame都拥有自己的window对象，并且保存在top.frames集合中。每个window对象都有一个name属性，其中包含框架的名称。可以通过name属性或索引访问具体的frame的window对象。
```

<frameset rows='160,*'>
 <frame src="frame1.html" name="topFrame1">
 <frame src="frame2.html" name="topFrame2">
</frameset>

```
引用topFrame1可以通过window.frames[0]或者window.frames['topFrame1'];
**top对象始终指向最外层的浏览器窗口。**这样就可以确保在一个frame中正确地访问另一个frame。
与top相对的另一个window对象是parent,**parent对象始终指向当前框架的直接上层框架。**另一个对象是self

###  窗口位置与大小 
screenLeft，screenTop表示浏览器窗口相对屏幕的位置。
innerWidth、innerHeight、outerWidth、outerHeight页面视图区的大小。

导航和打开窗口：window.open(要加载的URL,窗口目标(如_blank),特性字符串,布尔值表示是否需要在新窗口打开，默认是)
###  弹窗屏蔽提示 
```

        var blocked = false;
        try {
            var wroxWin = window.open("http://www.wrox.com", "_blank");
            if (wroxWin == null){
                blocked = true;
            }
        } catch (ex){
            blocked = true;
        }
        if (blocked){
            alert("The popup was blocked!");
        }

```
###  setInterval和setTimeout 
**JavaScript是` 单线程 `语言**，但它允许通过设置超时值和间歇时间值来调度代码在特定的时刻执行。
**超时调用需要使用window对象的setTimeout()方法，它接受两个参数:要仔细的函数，以毫秒表示的时间。但是经过该时间后指定的代码不一定会执行。**因为JS是一个**单线程**的解释器，JS在内部维护了一个**任务队列**，这些任务会按照队列中的顺序执行。setTimeout的第二个时间参数只是表示再过多长时间把当前任务添加到队列中。只有当队列为空的时候才会立即执行。
```

var timeoutId=setTimeout(function(){
	alert("Hello ,World!");
},1000);
clearTimeout(timeoutId);

```
setTimeout调用会返回一个数值ID，可以通过这个ID来取消超时调用。

setInterval类似。不过不建议使用它。可以使用setTimeout来模拟它。
```

       var num = 0;
        var max = 100;
        function incrementNumber() {
            num++;
            //if the max has not been reached, set another timeout
            if (num < max) {
                setTimeout(incrementNumber, 500);
            } else {
                alert("Done");
            }
        }
        setTimeout(incrementNumber, 500);

```

###  系统对话框 
alert()、confirm()、prompt()

##  location对象 
location提供了与当前窗口中加载的文档有关的信息，同时还提供了一些导航功能。它既是window的属性又是document的属性。
location常用的属性:
  * host "www.wrox.com"
  * hostname "www.wrox.com"
  * href  "http://www.wrox.com/index.html"
  * pathname "/index.html"
  * port "8080"
  * protocol "http:"
  * search  "?q=javascript"
###  获取查询字符串数组 
```

        function getQueryStringArgs(){
        
            //get query string without the initial ?
            var qs = (location.search.length > 0 ? location.search.substring(1) : ""),
            
                //object to hold data
                args = {},
            
                //get individual items
                items = qs.length ? qs.split("&") : [],
                item = null,
                name = null,
                value = null,
                
                //used in for loop
                i = 0,
                len = items.length;
            
            //assign each item onto the args object
            for (i=0; i < len; i++){
                item = items[i].split("=");
                name = decodeURIComponent(item[0]);
                value = decodeURIComponent(item[1]);
                
                if (name.length){
                    args[name] = value;
                }
            }
            
            return args;
        }

        //assume query string of ?q=javascript&num=10
        
        var args = getQueryStringArgs();
        
        alert(args["q"]);     //"javascript"
        alert(args["num"]);   //"10"

```
###  位置操作 
使用location对象可以通过很多方式来改变浏览器位置.
location.assign("http://wiki.xby1993.net");
window.location="http://wiki.xby1993.net";
location.href="http://wiki.xby1993.net";
reload()重新加载
location.replace("http://wiki.xby1993.net")不会记录到历史记录
##  navigator对象 
navigator常用属性:
属性	描述
appCodeName	返回浏览器的代码名。
appMinorVersion	返回浏览器的次级版本。
appName	返回浏览器的名称。
appVersion	返回浏览器的平台和版本信息。
browserLanguage	返回当前浏览器的语言。
cookieEnabled	返回指明浏览器中是否启用 cookie 的布尔值。
cpuClass	返回浏览器系统的 CPU 等级。
onLine	返回指明系统是否处于脱机模式的布尔值。
platform	返回运行浏览器的操作系统平台。
systemLanguage	返回 OS 使用的默认语言。
userAgent	返回由客户机发送服务器的 user-agent 头部的值。如"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36"
userLanguage	返回 OS 的自然语言设置。

##  history对象 
history保存着用户上网的历史记录。可以通过它进行历史跳转go(1),go(-1),go(2),back(),forward()

##  用户代理检测 
由客户机发送服务器的 user-agent 头部的值。如"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.101 Safari/537.36"
navigator.userAgent获取
```

/*
* 智能机浏览器版本信息:
*
*/
  var browser={
    versions:function(){
           var u = navigator.userAgent, app = navigator.appVersion;
           return {//移动终端浏览器版本信息
                trident: u.indexOf('Trident') > -1, //IE内核
                presto: u.indexOf('Presto') > -1, //opera内核
                webKit: u.indexOf('AppleWebKit') > -1, //苹果、谷歌内核
                gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1, //火狐内核
                mobile: !!u.match(/AppleWebKit.*Mobile.*/)||!!u.match(/AppleWebKit/), //是否为移动终端
                ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端
                android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1, //android终端或者uc浏览器
                iPhone: u.indexOf('iPhone') > -1 || u.indexOf('Mac') > -1, //是否为iPhone或者QQHD浏览器
                iPad: u.indexOf('iPad') > -1, //是否iPad
                webApp: u.indexOf('Safari') == -1 //是否web应该程序，没有头部与底部
            };
         }(),
         language:(navigator.browserLanguage || navigator.language).toLowerCase()
}
document.writeln("语言版本: "+browser.language);
document.writeln(" 是否为移动终端: "+browser.versions.mobile);
document.writeln(" ios终端: "+browser.versions.ios);
document.writeln(" android终端: "+browser.versions.android);
document.writeln(" 是否为iPhone: "+browser.versions.iPhone);
document.writeln(" 是否iPad: "+browser.versions.iPad);
document.writeln(navigator.userAgent);

```