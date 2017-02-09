title: spring使用陷阱七 

#  Spring陷阱之普通类中如何取session和request和response对象 
在使用spring时,经常需要在普通类中获取session,request等对像.**比如一些AOP拦截器类.**(在有使用struts2时,因为struts2有一个接口使用org.apache.struts2.ServletActionContext即可很方便的取到session对像.用法:ServletActionContext.getRequest().getSession();)
但在单独使用spring时如何在普通类中获取session,reuqest呢?
其实也是有办法的.
首先要在**web.xml**增加如下代码:
```

 <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
 </listener>

```

接着在普通bean类中:
```

@Autowired  
private HttpSession session;  
  
@Autowired  
private HttpServletRequest request; 

``` 
**` 注意：不能直接注入HttpServletResponse `**
即可,在类中使用session对像了,是不是很方便呢..但是不建议这么做，因为Spring的Bean都是只初始化一次，这样的话，对每个用户用的都是第一次的那个。这将是比较危险的。
之所以要写出来是因为目前网上关于这个的用法,都是用什么写个lister再把session保存起来,太麻烦了.
spring这么强大的框架,当然他们早也想到了.所以才有了我们这么方便的使用方法.
你还可以` RequestContextHolder `使用下面的方式获取：
```

HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.getRequestAttributes()).getRequest();

```

**如果需要获取` HttpServletResponse `可以通过如下方式：**
可以通过创建一个Filter过滤器，并将response对象保存到一个静态变量中即可。
参考：http://blog.csdn.net/yousite1/article/details/7108585