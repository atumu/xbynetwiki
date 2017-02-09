title: java_session超时自动跳转实现 

#  java Session超时自动跳转实现 
解决思路：
1. 后台添加拦截器，判断session是否过期，过期返回一个标识。
2. 前台捕获到这个请求返回的值，最好在一个统一的地方捕获。一般选择jQuery.ajax的$.ajaxSetup或Ext.Ajax的requestcomplete或者requestexception事件。
web.xml配置session超时：30分钟
```

	<session-config>
  		<session-timeout>30</session-timeout>
 	</session-config>

```
##  方式一、使用Spring-Security进行超时跳转 
后来发现spring security原来有sessiontimeout配置的地方，当session超时时，会自动触发。

2. spring-config-security.xml文件中配置session超时的触发函数
```

<sec:session-management invalid-session-url="/sessiontimeout.do" ></sec:session-management> 

``` 
 这里可以直接配置invalid-session-url="/login.jsp",这里我配置了一个处理函数，因为要解决异步处理的情况
3.编写超时处理函数
```

@RequestMapping("/sessiontimeout.do")  
    public void sessionTimeout(HttpServletRequest request,HttpServletResponse response) throws IOException {  
        String requestUrl = request.getRequestURI();   
        if (request.getHeader("x-requested-with") != null  
                && request.getHeader("x-requested-with").equalsIgnoreCase(  
                        "XMLHttpRequest")) { // ajax 超时处理  
            response.setHeader("sessionstatus", "timeout");  
            PrintWriter out = response.getWriter();  
            out.print("{timeout:true}");  
            out.flush();  
            out.close();  
        }else { // http 超时处理  
            response.sendRedirect(request.getContextPath() + "/login.do");  
        }  
  
    } 

``` 
** 4. 前台统一监听处理函数**
jquery 可以用$.ajaxSetup 方法，ext也有类似的方法
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
```

Ext.Ajax.on('requestcomplete',function(conn, response, options, eOpts){  
            if(response.getResponseHeader("sessionstatus")=='timeout'){  
                alert("登入超时,系统将自动跳转到登陆页面,请重新登入!");  
                window.location = __ctxPath +  "/login.do"; //如重定向到登陆页面   
            }  
        });

```

##  方式二、使用SpringMVC拦截器进行超时跳转。 

```

public class SessionTimeoutInterceptor implements HandlerInterceptorAdapter {  
   private static final Logger logger = LoggerFactory.getLogger(SessionTimeoutInterceptor.class);
    @Override  
    public boolean preHandle(HttpServletRequest request,  
            HttpServletResponse response, Object handler) throws Exception {  
            logger.debug("访问路径--->>>" + request.getRequestURI());
                Object obj = request.getSession().getServletContext().getAttribute("user");  
                if(obj != null) {    
                    return true;    
                }   
                response.setHeader("sessionstatus", "timeout");  //给前台传一个超时标志
                return false;  //返回false。 
  
    }  
} 

``` 
 2.拦截器配置
```

<mvc:interceptors>    
 		<mvc:interceptor>
			<mvc:mapping path="/**" />
			<mvc:exclude-mapping path="/static/**" />
			<mvc:exclude-mapping path="/signin.jsp" />
			<mvc:exclude-mapping path="/captcha/*" />
			<mvc:exclude-mapping path="/login" />
			<mvc:exclude-mapping path="/logout" />
			<mvc:exclude-mapping path="/visit/record" />
			<mvc:exclude-mapping path="/file/download" />
			<mvc:exclude-mapping path="/file/downloadPic" />
			<mvc:exclude-mapping path="/news/top" />
			<mvc:exclude-mapping path="/search/*" />
			<mvc:exclude-mapping path="/" />
			<bean class="com.my.SessionTimeoutInterceptor" />
		</mvc:interceptor>

``` 
3. 前台统一监听处理函数
jquery 可以用$.ajaxSetup 方法，ext也有类似的方法
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
```

Ext.Ajax.on('requestcomplete',function(conn, response, options, eOpts){  
            if(response.getResponseHeader("sessionstatus")=='timeout'){  
                alert("登入超时,系统将自动跳转到登陆页面,请重新登入!");  
                window.location = __ctxPath +  "/login.do"; //如重定向到登陆页面   
            }  
        });

```
##  方式三、使用HttpSessionListener进行超时跳转 
```

public class OnlineUserListener implements HttpSessionListener {
    public void sessionCreated(HttpSessionEvent event) {
    }
    public void sessionDestroyed(HttpSessionEvent event) {
        HttpSession session = event.getSession();
        ServletContext application = session.getServletContext();
        // 取得登录的用户名
        String username = (String) session.getAttribute("username");
        // 从在线列表中删除用户名
        List onlineUserList = (List) application.getAttribute("onlineUserList");
        onlineUserList.remove(username);
        System.out.println(username + "超时退出。");
    }
}

```
OnlineUserListener 实现了HttpSessionListener定义的两个方法：sessionCreated()和sessionDestroyed()。这两个方法可以监听到当前应用中session的创建和销毁情况。**我们这里只用到sessionDestroyed()在session销毁时进行操作就可以。**
从HttpSessionEvent中获得即将销毁的session，得到session中的用户名，并从在线列表中删除。最后一句向console打印一条信息，提示操作成功，这只是为了调试用，正常运行时删除即可。
为了让监听器发挥作用，我们将它添加到web.xml中：
```

<listener>
<listener-class>anni.OnlineUserListener</listener-class>
</listener>

```
##  方式四、使用Shiro进行超时跳转 

参考：http://zhengjunxiang.iteye.com/blog/1990689
http://blog.csdn.net/mywordandyourword/article/details/18937775
http://zhuchengzzcc.iteye.com/blog/1830567