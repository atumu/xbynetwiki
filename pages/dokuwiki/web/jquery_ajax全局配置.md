title: jquery_ajax全局配置 

#  jQuery Ajax全局配置 
##  $.ajaxSetup所有的全局设置 
```

//全局的ajax访问，处理ajax清求时sesion超时  
$.ajaxSetup({  
    complete:function(XMLHttpRequest,textStatus){  
        //通过XMLHttpRequest取得响应头，sessionstatus，  
        var sessionstatus=XMLHttpRequest.getResponseHeader("sessionstatus");   
        if(sessionstatus=="timeout"){  
        //如果超时就处理 ，指定要跳转的页面  
            window.location = "<c:url value="/" />";  
        }  
    }  
});

```
##  $(document).ajaxSuccess()/ $(document).ajaxComplete()单个document全局配置 
只是针对单个document的全局配置，不同document配置不同。
但是从 jQuery 1.8 开始, .ajaxSuccess()和.ajaxComplete() 等全局方法只能绑定到 document元素.
```

 /*
     * 未登录或session过期时ajax处理
     */
    $(document).ajaxSuccess(function (event,request,settings) {
    	var data = request.responseJSON;
        if(request.getResponseHeader('LOGIN-AUTH') === 'login'){
            require(["UtilDir/util"],function(util){
                util.confirm("您没有登录或会话已过期请重新登录，是否立即跳转到登录页？",function(){
                    window.location = getServer();
                })
            });
        }else  if(data.status=="500"){
        	alert(data.entity.msg+"\n"+data.entity.cause);
        	console.log(data.entity.stackTrace);
        }else if(data.status=="403"){
        	alert("权限不足 禁止访问");
        }else if("800" == data.status){
        	 require(["UtilDir/util"],function(util){
                util.confirm("您没有登录或会话已过期请重新登录，是否立即跳转到登录页？",function(){
                    window.location = getServer()+"/";
                })
             });
        }
    }).ajaxSend(function () {
        require(["UtilDir/util"],function(util){
            //util.progress.start();
        });
    }).ajaxError(function (event, jqxhr, settings, thrownError) {
        require(["UtilDir/util"],function(util){
            //处理没有被服务器捕获的异常, 可能服务器崩溃
        });
    })

```