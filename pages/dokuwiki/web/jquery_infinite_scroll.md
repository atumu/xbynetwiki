title: jquery_infinite_scroll 

#  jQuery Infinite Scroll无刷新无分页滚动追加内容插件 
是基于Infinite Scroll也是基于Jquery插件，用于当滚动条滚动时追加页面内容，有网友称这种效果为”无刷新无分页完美瀑布流”展现方式。
官网地址：http://infinite-scroll.com/
github:https://github.com/infinite-scroll/infinite-scroll
一.无限滚动概念
首先，它是基于Jquery的，另外还要明白无限滚动的概念：无限滚动的实现原理就是当你在网页上的**滚动条滚动到离网页底部一定长度的时候**，**触发某ajax函数（infinite-scroll内已经封装好）,往后台加载文件或者数据，又或者从外部引入静态html形式文件。**

常用选项
  * itemSelector    class选择器,默认'div.post'，这个表示上面介绍的瀑布流的块的选择器 
  * nextSelector    表示分页中下一页的选择器，默认为div.navigation a:first 
  * navSelector     表示分页导航的选择器，分页导航会被隐藏 
  * extraScrollPx   滚动条距离底部多少像素的时候开始加载，默认150 
  * dataType        表示动态加载返回数据的格式，默认html 
  * template         表示返回json时，用来生成瀑布流块html代码的模板方法，如果返回是json，那么必须指定这个参数，否则会报错 

```

一般的代码如下：

$('#waterfall').infinitescroll({
                navSelector: "#navigation", //导航的选择器，会被隐藏
                nextSelector: "#navigation a", //包含下一页链接的选择器
                itemSelector: ".wfc", //你将要取回的选项(内容块)
                debug: true, //启用调试信息
                animate: true, //当有新数据加载进来的时候，页面是否有动画效果，默认没有
                extraScrollPx: 150, //滚动条距离底部多少像素的时候开始加载，默认150
                bufferPx: 40, //载入信息的显示时间，时间越大，载入信息显示时间越短
                errorCallback: function() {
                    alert('error');
                }, //当出错的时候，比如404页面的时候执行的函数
                localMode: true, //是否允许载入具有相同函数的页面，默认为false
                dataType: 'json',//可以是html
	                template: function(jsondata) {
	                    //data表示服务端返回的json格式数据，这里需要把data转换成瀑布流块的html格式，然后返回给回到函数
	                   return '';
	                },
                loading: {
                    msgText: "加载中...",
                    finishedMsg: '没有新数据了...',
                    selector: '.loading' // 显示loading信息的div
                }
            }, function(newElems) {
                //程序执行完的回调函数
                var $newElems = $(newElems);
                $('.wrapper:eq(1)').masonry('appended', $newElems);
            });

```

demo:
```

$(document).ready(function (){  
  $("#container").infinitescroll({  
        navSelector: "#navigation",      //页面分页元素--本页的导航，意思就是一个可以触发ajax函数的模块  
        nextSelector: "#navigation a",  //下一页的导航  
        itemSelector: ".scroll " ,             //此处我用了类选择器，选择的是你要加载的那一个块（每次载入的数据放的地方）        
        animate: true,                          //加载时候时候需要动画，默认是false  
        maxPage: 3,                            //最大的页数，也就是滚动多少次停。这个页码必须得要你从数据库里面拿         
    });  
}); 

```
```

<%@ page language="java" contentType="text/html; charset=utf-8"  
    pageEncoding="utf-8"%>  
<!DOCTYPE html>  
<html>  
<head>  
<meta charset="utf-8">  
<title>无限翻页效果</title>  
<script src="css/infinite-scroll/jquery-1.6.4.js"></script>  
<script src="css/infinite-scroll/jquery.infinitescroll.js"></script>  
<script src="css/infinite-scroll/test/debug.js"></script>  
 <script>  
 $(document).ready(function (){               //别忘了加这句，除非你没学Jquery  
      $("#container").infinitescroll({  
            navSelector: "#navigation",     //页面分页元素--成功后自动隐藏  
            nextSelector: "#navigation a",  
            itemSelector: ".scroll " ,             
            animate: true,  
            maxPage: 3,                                                  
        });  
 });   
    </script>  
</head>  
<body>  
    <div id="container">            <!-- 容器 -->  
        <div class="scroll">         <!-- 每次要加载数据的数据块-->  
        第一页内容第一页内容<br>  
        第一页内容<br>第一页内容<br>第一页内容<br>  
        第一页内容<br>第一页内容<br>第一页内容<br>  
        第一页内容<br>第一页内容<br>第一页内容  
        </div>  
    </div>  
        <div id="navigation" align="center">         <!-- 页面导航-->  
        <a href="user/list?page=1"></a>        <!-- 此处可以是url，可以是action，要注意不是每种html都可以加，是跟当前网页有相同布局的才可以。另外一个重要的地方是page参数，这个一定要加在这里，它的作用是指出当前页面页码，没加载一次数据，page自动+1,我们可以从服务器用request拿到他然后进行后面的分页处理。-->  
    </div>  
</body>  
</html>  

```
  
1、分页导航html格式问题，并不是任意的html都可以的，必须有一定的格式，具体可以看插件的源码，格式如：page.aspx?page=1或者page/2/，还有其他格式请看源码；每次加载前会数字会递增1，后台可以用Request["` page `"]获取参数；
**2、如果要json数据**，那么` 必须指定dataType参数为json `，并设置` template `模板方法，该方法接收一个json数据，然后把json替换成瀑布流的html就可以了；
3、数据返回后，需要重新调用瀑布流插件重新布局，query.infinitescroll在数据返回后又一个回掉，格式为：```

$('#content').infinitescroll({},function(newElems){//newElems表示返回的数据，如果是json的话就是template的返回值})；

```
3.数据分页方式
本例用hibernate进行数据分页。
##  选项 
```

$('.selector').infinitescroll({
  loading: {
    finished: undefined,
    finishedMsg: "<em>Congratulations, you've reached the end of the internet.</em>",
                img: null,
    msg: null,
    msgText: "<em>Loading the next set of posts...</em>",
    selector: null,
    speed: 'fast',
    start: undefined
  },
  state: {
    isDuringAjax: false,
    isInvalidPage: false,
    isDestroyed: false,
    isDone: false, // For when it goes all the way through the archive.
    isPaused: false,
    currPage: 1
  },
  behavior: undefined,
  binder: $(window), // used to cache the selector for the element that will be scrolling
  nextSelector: "div.navigation a:first",
  navSelector: "div.navigation",
  contentSelector: null, // rename to pageFragment
  extraScrollPx: 150,
  itemSelector: "div.post",
  animate: false,
  pathParse: undefined,
  dataType: 'html',
  appendCallback: true,
  bufferPx: 40,
  errorCallback: function () { },
  infid: 0, //Instance ID
  pixelsFromNavToBottom: undefined,
  path: undefined, // Can either be an array of URL parts (e.g. ["/page/", "/"]) or a function that accepts the page number and returns a URL
  maxPage:undefined // to manually control maximum page (when maxPage is undefined, maximum page limitation is not work)
});

```

##  Methods 
A method is a command you can use to control Infinite Scroll once the plugin has been initialized. You can call on any Infinite Scroll method by using $('.selector').infinitescroll('method-name');.

Bind
$('.selector').infinitescroll('bind');
Binds selector to check on scroll to see if the plugin needs to load more content.

Unbind
$('.selector').infinitescroll('unbind');
Unbinds selector to check on scroll to see if the plugin needs to load more content.

Destroy
$('.selector').infinitescroll('destroy');
Destroys the instance of infinite scroll. This is create a flag to not load anymore content and will unbind all events.

Pause
$('.selector').infinitescroll('pause');
Pausing the plugin will temporarily create a flag to not retrieve content on scroll. To unpause, use the method resume.

Resume
$('.selector').infinitescroll('resume');
Destroys the instance of infinite scroll. This is create a flag to not load anymore content and will unbind all events.

Toggle
$('.selector').infinitescroll('toggle');
Toggling will switch the pause value of the plugin, either pausing or resuming the plugin.

Retrieve
$('.selector').infinitescroll('retrieve');
Retrieve will load the next page of content if available.

Scroll
$('.selector').infinitescroll('scroll');
Scroll will check to see if the next page is to be loaded, the same thing as if a user scrolled.

Update
$('.selector').infinitescroll('update', {debug: true});
The update method is used to update options in the instance of Infinite Scroll after initialization. The second argument is the object of options that you want to update.

参考：http://www.tuicool.com/articles/F3iURz