title: springmvc拦截器_资源和权限管理 

#  SpringMVC拦截器（资源和权限管理） 
参考：http://blog.csdn.net/tonytfjing/article/details/39207551 http://haohaoxuexi.iteye.com/blog/1750680
3.自定义拦截器
SpringMVC的拦截器HandlerInterceptor对应提供了三个` preHandle，postHandle，afterCompletion `方法。并提供了一个便利的适配器类：` HandlerInterceptorAdapter `。 
  * preHandle在业务处理器处理请求之前被调用，
  * postHandle在业务处理器处理请求执行完成后,生成视图之前执行，
  * afterCompletion在DispatcherServlet完全处理完请求后被调用,可用于清理资源等 。
所以要想实现自己的权限管理逻辑，需要继承HandlerInterceptorAdapter并重写其方法。

首先在配置文件中加入自己定义的拦截器我的实现逻辑
```

<!--配置拦截器, 多个拦截器,顺序执行 -->  
<mvc:interceptors>    
   <!-- 使用bean定义一个Interceptor，直接定义在mvc:interceptors根下面的Interceptor将拦截所有的请求 -->  
    <bean class="com.host.app.web.interceptor.AllInterceptor"/>  
    <mvc:interceptor>    
        <!-- 匹配的是url路径， 如果不配置或/**,将拦截所有的Controller -->  
        <mvc:mapping path="/" />  
        <mvc:mapping path="/user/**" />  
        <mvc:mapping path="/test/**" />  
      <!-- 定义在mvc:interceptor下面的表示是对特定的请求才进行拦截的 -->  
        <bean class="com.alibaba.interceptor.CommonInterceptor"></bean>    
    </mvc:interceptor>  
    <!-- 当设置多个拦截器时，先按顺序调用preHandle方法，然后逆序调用每个拦截器的postHandle和afterCompletion方法 -->  
</mvc:interceptors>  

```

我的拦截逻辑是“在未登录前，任何访问url都跳转到login页面；登录成功后跳转至先前的url”，具体代码如下：
```

public class CommonInterceptor extends HandlerInterceptorAdapter{  
    private final Logger log = LoggerFactory.getLogger(CommonInterceptor.class);  
    public static final String LAST_PAGE = "com.alibaba.lastPage";  

    /**  
     * 在业务处理器处理请求之前被调用  
     * 如果返回false  
     *     从当前的拦截器往回执行所有拦截器的afterCompletion(),再退出拦截器链 
     * 如果返回true  
     *    执行下一个拦截器,直到所有的拦截器都执行完毕  
     *    再执行被拦截的Controller  
     *    然后进入拦截器链,  
     *    从最后一个拦截器往回执行所有的postHandle()  
     *    接着再从最后一个拦截器往回执行所有的afterCompletion()  
     */    
    @Override    
    public boolean preHandle(HttpServletRequest request,    
            HttpServletResponse response, Object handler) throws Exception {    
        if ("GET".equalsIgnoreCase(request.getMethod())) {  
            RequestUtil.saveRequest();  
        }  
        log.info("# ==执行顺序: 1、preHandle# ====");    
        String requestUri = request.getRequestURI();  
        String contextPath = request.getContextPath();  
        String url = requestUri.substring(contextPath.length());  
        
        log.info("requestUri:"+requestUri);    
        log.info("contextPath:"+contextPath);    
        log.info("url:"+url);    
          
        String username =  (String)request.getSession().getAttribute("user");   
        if(username == null){  
            log.info("Interceptor：跳转到login页面！");  
            request.getRequestDispatcher("/WEB-INF/jsp/login.jsp").forward(request, response);  
            return false;  
        }else  
            return true;     
    }    
    
    /** 
     * 在业务处理器处理请求执行完成后,生成视图之前执行的动作    
     * 可在modelAndView中加入数据，比如当前时间 
     */  
    @Override    
    public void postHandle(HttpServletRequest request,    
            HttpServletResponse response, Object handler,    
            ModelAndView modelAndView) throws Exception {     
        log.info("# ==执行顺序: 2、postHandle# ====");    
        if(modelAndView != null){  //加入当前时间    
            modelAndView.addObject("var", "测试postHandle");    
        }    
    }    
    
    /**  
     * 在DispatcherServlet完全处理完请求后被调用,可用于清理资源等   
     *   
     * 当有拦截器抛出异常时,会从当前拦截器往回执行所有的拦截器的afterCompletion()  
     */    
    @Override    
    public void afterCompletion(HttpServletRequest request,    
            HttpServletResponse response, Object handler, Exception ex)    
            throws Exception {    
        log.info("# ==执行顺序: 3、afterCompletion# ====");    
    }    
  
}    

```
至此，拦截器已经实现了。
