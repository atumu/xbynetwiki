title: jquery站内页面刷新pjax 

#  jQuery站内页面刷新pjax 
pushState + ajax = pjax
jQuery的Pjax插件，Pjax即pushState + Ajax，是实现无刷新ajax加载并解决浏览器前进和后退问题的一个开源实现。
github:https://github.com/defunkt/jquery-pjax
demo:http://pjax.herokuapp.com/

注意：**pjax需要服务端配合客户端使用**。
pjax 示例代码：
```

$.pjax({
  url: '/authors',
  container: '#main'
})

```
而 ajax中应用pushState的做法：
```

$.ajax({
  url: '/authors',
  dataType: 'html',
  beforeSend: function(xhr){
    xhr.setRequestHeader('X-PJAX', 'true')
  },
  success: function(data){
    $('#main').html(data)
    history.pushState(null, $(data).filter('title').text(), '/authors')
  })
})

```

##  什么是pjax? 
现在很多网站( facebook,  twitter)都支持这样的一种浏览方式， 当你点击一个站内的链接的时候， 不是做页面跳转， 而是只是站内页面刷新。 这样的用户体验， 比起整个页面都闪一下来说， 好很多。 其中有一个很重要的组成部分， 这些网站的ajax刷新是支持浏览器历史的， 刷新页面的同时， 浏览器地址栏位上面的地址也是会更改， 用浏览器的回退功能也能够回退到上一个页面。 
那么如果我们想要实现这样的功能， 我们如何做呢？ 我发现pjax提供了一个脚本支持这样的功能。 pjax项目地址在  https://github.com/defunkt/jquery-pjax 。 实际的效果见：  http://pjax.heroku.com/ 没有勾选pjax的时候， 点击链接是跳转的。 勾选了之后， 链接都是变成了ajax刷新。

###  浏览器支持 
提供了` history.pushState `接口的浏览器才支持这个功能
判断浏览器是否支持
```

$(document).ready(function(){

    // does current browser support PJAX
    if ($.support.pjax) {
        $.pjax.defaults.timeout = 1000; // time in milliseconds
    }

});

```
###  服务端支持 
类似于ajax, 异步请求的时候后端不能将公用的内容也返回。
所以需要一个判断是否pjax请求的接口。如：rails可以借鉴下面的实现
```

def index
  if request.headers['X-PJAX']
    render :layout => false
  end
end

```
` An X-PJAX request header is set to differentiate a pjax request from normal XHR requests. In this case, if the request is pjax, we skip the layout html and just render the inner contents of the container. `
##  使用 
$(document).pjax(selector, [container], options)
  * selector is a string to be used for click event delegation.
  * container is a string selector that uniquely identifies the pjax container.
  * options is an object with keys described below.

选项列表请参考https://github.com/defunkt/jquery-pjax
##  事件 
事件列表，请参考https://github.com/defunkt/jquery-pjax，示例如下：
```

$(document).on('pjax:send', function() {
  $('#loading').show()
})
$(document).on('pjax:complete', function() {
  $('#loading').hide()
})

```
##  示例 

```

<!DOCTYPE html>
<html>
<head>
  <!-- styles, scripts, etc -->
</head>
<body>
  <h1>My Site</h1>
  <div class="container" id="pjax-container">
    Go to <a href="/page/2">next page</a>.
  </div>
</body>
</html>

```
```

$(document).pjax('a', '#pjax-container')

```
##  服务端第三方框架支持 
支持列表见：https://gist.github.com/defunkt/4283721
Laravel5支持有两个:1、https://github.com/JacobBennett/pjax ；2、Laravel 5 middleware: https://github.com/spatie/laravel-pjax
**Laravel5第一种：**
Installation
composer require jacobbennett/pjax

Add 'JacobBennett\Pjax\PjaxMiddleware', to ` $middleware ` in app/Http/Kernel.php

How to use
This middleware will check, before outputting the http response, for the ` X-PJAX `'s header in the request. If found, it will crawl the response to return the requested element defined by ` X-PJAX-Container `'s header.

**Laravel5第二种:**
```

$ composer require spatie/laravel-pjax

```
Next you must add the \Spatie\Pjax\Middleware\FilterIfPjax-middleware to the kernel.
```

// app/Http/Kernel.php
...
protected $middleware = [
    ...
    \Spatie\Pjax\Middleware\FilterIfPjax::class,
];

```

参考：
http://my.oschina.net/sub/blog/123447