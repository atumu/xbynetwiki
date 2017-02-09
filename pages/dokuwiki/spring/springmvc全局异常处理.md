title: springmvc全局异常处理 

#  SpringMVC全局异常处理 
参考：http://cgs1999.iteye.com/blog/1547197
http://blog.csdn.net/mr__fang/article/details/9092463
##  方式0：web.xml中配置 
对于Unchecked Exception而言，由于代码不强制捕获，往往被忽略，如果运行期产生了Unchecked Exception，而代码中又没有进行相应的捕获和处理，则我们可能不得不面对尴尬的404、500……等服务器内部错误提示页面。 
我们需要一个全面而有效的异常处理机制。目前大多数服务器也都支持**在Web.xml中通过<error-page>(Websphere/Weblogic)或者<error-code>(Tomcat)节点配置特定异常情况的显示页面。修改web.xml文件**，增加以下内容： 
```

<error-page>  
    <exception-type>java.lang.Throwable</exception-type>  
    <location>/500.jsp</location>  
</error-page>  
<error-page>  
    <error-code>500</error-code>  
    <location>/500.jsp</location>  
</error-page>  
<error-page>  
    <error-code>404</error-code>  
    <location>/404.jsp</location>  
</error-page> 

``` 


  
##  方式一、实现自己的HandlerExceptionResolver 
实现自己的` HandlerExceptionResolver `，HandlerExceptionResolver是一个接口，springMVC本身已经对其有了一个自身的实现——DefaultExceptionResolver,该解析器只是对其中的一些比较典型的异常进行了拦截处理。
```

public class ExceptionHandler implements HandlerExceptionResolver {   
  
    @Override  
    public ModelAndView resolveException(HttpServletRequest request,   
            HttpServletResponse response, Object handler, Exception ex) {   
        // TODO Auto-generated method stub   
        return new ModelAndView("exception");   
    }   
}

```
上述的resolveException的第4个参数表示对哪种类型的异常进行处理，如果想同时对多种异常进行处理，可以把它换成一个异常数组。
定义了这样一个异常处理器之后就要在applicationContext中定义这样一个bean对象，如：
```

<bean id="exceptionResolver" class="com.tiantian.xxx.web.handler.ExceptionHandler"/>

```  
##  方式二、使用注解@ExceptionHandler 
使用` @ExceptionHandler `进行处理有一个不好的地方是**进行异常处理的方法必须与出错的方法在同一个Controller里面**
现在spring3.0注解很方便强大，所以更多的开发者都倾向于用注解来代替原来繁琐的配置
写一个**公共的controller**，用` @ExceptionHandler `来拦截异常，**然后此controller被其他controller继承，这样就用很少的代码解决异常拦截的问题**，公共controller代码如下：
```

@Controller  
public class ExceptionHandlerController {  
    @ExceptionHandler(RuntimeException.class)  
    public String operateExp(RuntimeException ex,HttpServletRequest request){  
        System.out.println("this is for test");  
        //mod.addAttribute("err", ex.getMessage()); //ExceptionHandler处理异常时，Model，是不能用的，否则会不起作用，这里用了HttpServletRequest  
        request.setAttribute("err", ex.getMessage());  
        return "public/error";  
    }  
} 

``` 
##  方式三、使用SimpleMappingExceptionResolver实现异常处理 
在Spring的配置文件applicationContext.xml中增加以下内容： 
```

<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">  
    <!-- 定义默认的异常处理页面，当该异常类型的注册时使用 -->  
    <property name="defaultErrorView" value="error"></property>  
    <!-- 定义异常处理页面用来获取异常信息的变量名，默认名为exception -->  
    <property name="exceptionAttribute" value="ex"></property>  
    <!-- 定义需要特殊处理的异常，用类名或完全路径名作为key，异常也页名作为值 -->  
    <property name="exceptionMappings">  
        <props>  
            <prop key="cn.basttg.core.exception.BusinessException">error-business</prop>  
            <prop key="cn.basttg.core.exception.ParameterException">error-parameter</prop>  
  
            <!-- 这里还可以继续扩展对不同异常类型的处理 -->  
        </props>  
    </property>  
</bean>

```


##  总结  
综合上述可知，Spring MVC集成异常处理3种方式都可以达到统一异常处理的目标。
从3种方式的优缺点比较，
  * 若只需要简单的集成异常处理，推荐使用SimpleMappingExceptionResolver即可；
  * 若需要集成的异常处理能够更具个性化，提供给用户更详细的异常信息，推荐自定义实现HandlerExceptionResolver接口的方式；
  * 若不喜欢Spring配置文件或要实现“零配置”，且能接受对原有代码的适当入侵，则建议使用@ExceptionHandler注解方式。 

但是总之，无论如何对web.xml中404进行配置