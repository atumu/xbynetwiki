title: ajax与302响应 

#  ajax与302响应 
在ajax请求中，如果服务器端的响应是302 Found，在ajax的回调函数中能够获取这个状态码吗？能够从Response Headers中得到Location的值进行重定向吗？让我们来一起看看实际情况。
使用jquery的$.ajax()发起ajax请求的javascript代码如下：
```

$.ajax({
    url: '/oauth/respond',
    type: 'post',
    data: data,
    complete: function(jqXHR){
        console.log(jqXHR.status);
    },
    error: function (xhr) {
        console.log(xhr.status);
    }
});

```        
当服务器端返回302 Found的响应时，浏览器中的运行结果为200.
在ajax的complete()与error()回调函数中得到的状态码都是200，而不是302。

为什么呢？
在stackoverflow上找到了答案：
You can't handle redirects with XHR callbacks because the browser takes care of them automatically. You will only get back what at the redirected location.
` 原来，当服务器将302响应发给浏览器时，浏览器并不是直接进行ajax回调处理 `，而是先执行302重定向——从Response Headers中读取Location信息，然后向Location中的Url发出请求，在收到这个请求的响应后才会进行ajax回调处理。**大致流程如下：
ajax -> browser -> server -> 302 -> browser(redirect) -> server -> browser -> ajax callback**
而在我们的测试程序中，由于302返回的重定向URL在服务器上没有相应的处理程序，所以在ajax回调函数中得到的是404状态码；如果存在对应的URL，得到的状态码就是200。
**所以，如果你想在ajax请求中根据302响应通过location.href进行重定向` 是不可行的 `。**
如何解决？
继续用ajax，**修改服务器端代码**，增加判断
```

		String requestType = request.getHeader("X-Requested-With");
			//如果是ajax请求
		if(StringUtils.checkEquals(requestType, "XMLHttpRequest")){
                  	// 给前台传一个超时标志
			response.setHeader("sessionstatus", "timeout"); 
                  	//返回任意一个json串，防止前台报错
			response.getOutputStream().print("{\"status\":\"timeout\"}");
		}else{
			//如果是普通的浏览器HTTP请求。
			ControllerUtil.redirectURL(response, "/login");
		}

```
```

	/*
	 * 未登录或session过期时ajax处理
	 */
    $(document).ajaxComplete(function (event,request,settings) {
    	var status=request.status;
    	//var data = request.responseJSON;
        if(request.getResponseHeader('sessionstatus') === 'timeout'){
        	
        		uikit.modal.confirm("登录超时了，返回到登录页面?", function(){
            	    // 点击OK确认后开始执行
        			window.location = getServer()+"/login";
            	});
        
        }else  if(status==500){
        	uikit.modal.alert("服务器异常500！");
        }else if(status==401){
        	uikit.modal.alert("权限不足 禁止访问");
        }
    });

```
参考：http://www.cnblogs.com/dudu/p/ajax_302_found.html