title: springmvc学习 

#  springMVC学习 
还可以参考:[[springmvc:springmvc入门]]
主要内容：
  * 映射请求到Spring控制器
  * 透明的绑定表单参数
  * 校验表单提交
  * 上传文件

基于web应用的几个关注点：状态管理、工作流、验证等。
##  跟踪SpringMVC请求 
Model2应用流程：
![](/data/dokuwiki/spring/pasted/20150806-050600.png)
SpringMVC流程：
![](/data/dokuwiki/spring/pasted/20150806-050400.png)
![](/data/dokuwiki/spring/pasted/20150806-050738.png)

**核心架构的具体流程步骤如下：**
1、首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行
处理，作为统一访问点，进行全局的流程控制；
2、DispatcherServlet——>HandlerMapping， HandlerMapping 将会把请求映射为HandlerExecutionChain 对象（包含一
个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新
的映射策略；
3、DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，
即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4、HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处
理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）；
5、ModelAndView的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的View，通过这种策
略模式，很容易更换其他视图技术；
6、View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此
很容易支持其他视图技术；
7、返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。
此处我们只是讲了核心流程，没有考虑拦截器、本地解析、文件上传解析等，后边再细述。

**在此我们可以看出具体的核心开发步骤：**
1、DispatcherServlet在web.xml 中的部署描述，从而拦截请求到Spring Web MVC
2、HandlerMapping的配置，从而将请求映射到处理器
3、HandlerAdapter 的配置，从而支持多种类型的处理器
4、ViewResolver 的配置，从而将逻辑视图名解析为具体视图技术
5、处理器（页面控制器）的配置，从而进行功能处理

#  SpringMVC开发 
` DispatcherServlet `主要用作职责调度工作，本身主要用于控制流程，主要职责如下：
1、文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；
2、通过HandlerMapping，将请求映射到处理器（返回一个HandlerExecutionChain，它包括一个处理器、多个HandlerInterceptor拦截器）；
3、通过HandlerAdapter支持多种类型的处理器(HandlerExecutionChain中的处理器)；
4、通过ViewResolver解析逻辑视图名到具体视图实现；
5、本地化解析；
6、渲染具体的视图等；
7、如果执行过程中遇到异常将交给HandlerExceptionResolver来解析。

DispatcherServlet也可以配置自己的初始化参数，覆盖默认配置：
  * contextClass实现WebApplicationContext接口的类，当前的servlet用它来创建上下文。如果这个参数没有指定， 默认使用XmlWebApplicationContext。
  * contextConfigLocation传给上下文实例（由contextClass指定）的字符串，用来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。
  * namespace :WebApplicationContext命名空间。默认值是[server-name]-servlet。

因此我们可以通过添加初始化参数
```

    <servlet>
        <servlet-name>chapter2</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet-config.xml</param-value>
        </init-param>
    </servlet>

```
如果使用如上配置，Spring Web MVC框架将加载“classpath:spring-servlet-config.xml”来进行初始化上下文而不是“/WEB-INF/[servlet名字]-servlet.xml”。

` Controller `控制器，是MVC中的部分C，为什么是部分呢？因为此处的控制器主要负责功能处理部分：
  * 1、收集、验证请求参数并绑定到命令对象；
  * 2、将命令对象交给业务对象，由业务对象处理并返回模型数据；
  * 3、返回` ModelAndView `（Model部分是业务对象返回的模型数据，视图部分为逻辑视图名）。
 
还记得DispatcherServlet吗？主要负责整体的控制流程的调度部分：
1、负责将请求委托给控制器进行处理；
2、根据控制器返回的逻辑视图名选择具体的视图进行渲染（并把模型数据传入）。
 
因此MVC中完整的C（包含控制逻辑+功能处理）由（DispatcherServlet + Controller）组成。


##  搭建SpringMVC 

**第一步：web.xml配置DispatcherServlet**
```

 <servlet>
        <servlet-name>spitter</servlet-name>
   注意，这个Servlet名很重要，它决定了Spring应用上下文的加载文件名为spitter-servlet.xml
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>spitter</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

```
`  注意，这个Servlet名很重要，它决定了Spring应用上下文的加载文件名为spitter-servlet.xml `如果不想受这一限制，可以采取如下方式配置文件地址：
```

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
	    <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/config/springmvc-config.xml</param-value>
		</init-param>
        <load-on-startup>1</load-on-startup>    
	</servlet>

```
###  spitter-servlet.xml配置文件示例 
```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:mvc="http://www.springframework.org/schema/mvc"  
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd     
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="app04a.controller"/>  
    <mvc:annotation-driven/>
    <mvc:resources mapping="/css/**" location="/css/"/>
    <mvc:resources mapping="/*.html" location="/"/>
    
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>

```
##  POST中文乱码解决方案 

spring Web MVC框架提供了` org.springframework.web.filter.CharacterEncodingFilter `用于解决POST方式造成的中文乱码问题，具体配置如下：
以后我们项目及所有页面的编码均为UTF-8。
```

<filter>
<filter-name>CharacterEncodingFilter</filter-name>
<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
<init-param>
<param-name>encoding</param-name>
<param-value>utf-8</param-value>
</init-param>
</filter>
<filter-mapping>
<filter-name>CharacterEncodingFilter</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>

```
**1.2对静态资源的处理**
针对spring上下文配置文件spitter-servlet.xml中配置:
 xmlns:mvc="http://www.springframework.org/schema/mvc"
```

 <mvc:resources mapping="/resources/**" location="/resources/" /> 针对静态资源的处理，制定静态资源的位置

```
  ** 采用ant语法，指定任意子路径。
  
##  Spring应用上下文加载多个配置文件 
```

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            /WEB-INF/spitter-security.xml
            classpath:service-context.xml
            classpath:persistence-context.xml
            classpath:dataSource-context.xml
        </param-value>
    </context-param>
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

```
##  编写基本的控制器 
开发**面向资源**的控制器：每一种资源提供一个控制器，而不是每个用例编写一个控制器。
![](/data/dokuwiki/spring/pasted/20150806-052144.png)
**配置注解驱动的SpringMVC:**
配置` <mvc:annotation-driven/> `
###  Spring的处理器映射器： 

![](/data/dokuwiki/spring/pasted/20150806-052753.png)
![](/data/dokuwiki/spring/pasted/20150806-052851.png)
###  定义首页控制器 
```

@Controller  //声明为控制器
public class HomeController {

  private SpitterService spitterService;

  @Inject  //注入依赖
  public HomeController(SpitterService spitterService) {
    this.spitterService = spitterService;
  }
  
  @RequestMapping(value={"/","/home"}, method=RequestMethod.GET)  //定义处理方法和url
  public String showHomePage(Map<String, Object> model) {   //参数中传入map类型的model，也可以是其他类型。
    model.put("spittles", 
              spitterService.getRecentSpittles(spittlesPerPage));
    return "home"; //返回视图逻辑名称
  }

```
`  <context:component-scan base-package="com.habuma.spitter.mvc" /> `
###  springmvc命令/表单对象-数据绑定： 
更多参考：http://jinnianshilongnian.iteye.com/blog/1698916 http://jinnianshilongnian.iteye.com/blog/1705701
Spring Web MV**C能够自动将请求参数绑定到功能处理方法的命令/表单对象上。**
```

@RequestMapping(value = "/commandObject", method = RequestMethod.GET)  
public String toCreateUser(HttpServletRequest request, UserModel user) {  
    return "customer/create";  
}  
@RequestMapping(value = "/commandObject", method = RequestMethod.POST)  
public String createUser(HttpServletRequest request, UserModel user) {  
    System.out.println(user);  
    return "success";  
}

```  
 如果提交的表单（包含username和password文本域），将自动将请求参数绑定到命令对象user中去。

##  解析视图 
![](/data/dokuwiki/spring/pasted/20150806-053944.png)
先介绍` InternalResourceViewResolver `
![](/data/dokuwiki/spring/pasted/20150806-054044.png)
在spitter-servlet.xml中配置它：
![](/data/dokuwiki/spring/pasted/20150806-054126.png)
![](/data/dokuwiki/spring/pasted/20150806-054157.png)
![](/data/dokuwiki/spring/pasted/20150806-054212.png)
###  解析Tiles视图 
Apache Tiles是一个模板框架。它将页面分成片段并在运行时组装成完整的页面。
```

   <bean class="org.springframework.web.servlet.view.tiles2.TilesConfigurer">
     <property name="definitions">
       <list>
         <value>/WEB-INF/views/**/views.xml</value>
       </list>
     </property>
   </bean>
      
   <bean class="org.springframework.web.servlet.view.tiles2.TilesViewResolver"/>

```
![](/data/dokuwiki/spring/pasted/20150806-054431.png)
tiles views.xml中内容
```

<tiles-definitions>
   <definition name="template" 
               template="/WEB-INF/views/main_template.jsp">
     <put-attribute name="top" 
                    value="/WEB-INF/views/tiles/spittleForm.jsp" />
     <put-attribute name="side" 
                    value="/WEB-INF/views/tiles/signinsignup.jsp" />
   </definition>

   <definition name="home" extends="template">
     <put-attribute name="content" value="/WEB-INF/views/home.jsp" />
   </definition>  
   
   <definition name="login" extends="template">
     <put-attribute name="content" value="/WEB-INF/views/login.jsp" />
     <put-attribute name="side" value="/WEB-INF/views/tiles/alreadyamember.jsp" />
   </definition>   
   
   <definition name="admin" extends="template">
     <put-attribute name="content" value="/WEB-INF/views/admin.jsp" />
   </definition>     
</tiles-definitions>

```
###  定义首页视图 
```

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="s" uri="http://www.springframework.org/tags"%>  springmvc标签
<%@ taglib prefix="t" uri="http://tiles.apache.org/tags-tiles"%>  
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
<c:forEach var="spittle" items="${spittles}"> <!--<co id="cp_foreach_spittles"/>-->
    
      <s:url value="/spitters/{spitterName}"  s:url相对于servlet上下文路径，支持参数化路径， {spitterName}为占位符，由<s:param>给定
                  var="spitter_url" >    <!--<co id="cp_spitter_url"/>-->
        <s:param name="spitterName" 
                      value="${spittle.spitter.username}" />
      </s:url>

```
![](/data/dokuwiki/spring/pasted/20150806-055131.png)
##  处理控制器的输入 
```

@Controller
@RequestMapping("/spitters")  //类的路径映射
public class SpitterController {
  private final SpitterService spitterService;
  
  @Value("#{s3Properties['s3.accessKey']}")
  private String s3AccessKey;
  @Value("#{s3Properties['s3.secretKey']}")
  private String s3SecretKey;
  
  @Inject
  public SpitterController(SpitterService spitterService) {
    this.spitterService = spitterService;
  }

  @RequestMapping(method=RequestMethod.GET)  //没有指定路径，就为处理类的路径映射
  public String listSpitters(
          @RequestParam("page") int page, //前面一个page指定是表单的name，用于自动表单填充。
          @RequestParam(value="perPage", defaultValue="10") int perPage,
         Model model) {  //使用org.springframework.ui.Model作为模型
    model.put("spitters", spitterService.getAllSpitters()); //填充模型
    return "spitters/list";
  }

```
![](/data/dokuwiki/spring/pasted/20150806-073623.png)
![](/data/dokuwiki/spring/pasted/20150806-073639.png)
![](/data/dokuwiki/spring/pasted/20150806-073651.png)
###  展现注册表单 
![](/data/dokuwiki/spring/pasted/20150806-073934.png)
![](/data/dokuwiki/spring/pasted/20150806-073948.png)
使用SpringMVC的表单标签：
<%@ taglib prefix="sf" uri="http://www.springframework.org/tags/form"%>
```

<sf:form method="POST" modelAttribute="spitter"   //sf:form 的modelAttribute将表单绑定到模型属性
         enctype="multipart/form-data">         //指定表单可以用于上传
   <fieldset> 
   <table cellspacing="0">
      <tr>
         <th><sf:label path="username">Username:</sf:label></th>
         <td><sf:input path="username" size="15" maxlength="15" />  //sf:input的path用于对应模型的属性名。
              <small id="username_msg">No spaces, please.</small><br/>  
             <sf:errors path="username" cssClass="error" />  //sf:errors 用于表单校验失败时显示错误信息，path为对应的模型属性名。

            </td>
      </tr>
      <!--<start id="image_field"/>--> 
      <tr>
        <th><label for="image">Profile image:</label></th>
        <td><input name="image" type="file"/>
      </tr>
      <!--<end id="image_field"/>--> 
      <tr>
         <th></th>
         <td>

```
![](/data/dokuwiki/spring/pasted/20150806-074743.png)
![](/data/dokuwiki/spring/pasted/20150806-074809.png)
###  处理带有路径变量的请求 
![](/data/dokuwiki/spring/pasted/20150806-074931.png)
###  校验输入 
![](/data/dokuwiki/spring/pasted/20150806-075043.png)
**定义校验规则：**
![](/data/dokuwiki/spring/pasted/20150806-075110.png)
![](/data/dokuwiki/spring/pasted/20150806-075312.png)
**展现校验错误：**
BindingResult
<sf:errors>
###  处理文件上传 
**配置Spring支持文件上传：**
![](/data/dokuwiki/spring/pasted/20150806-080100.png)
```

   <bean id="multipartResolver" class=
	   "org.springframework.web.multipart.commons.CommonsMultipartResolver"
     p:maxUploadSize="500000" />

```
![](/data/dokuwiki/spring/pasted/20150806-075550.png)
```

@RequestMapping(method=RequestMethod.POST)
  public String addSpitterFromForm(@Valid Spitter spitter, 
      BindingResult bindingResult,
      @RequestParam(value="image", required=false) MultipartFile image) {  多了个参数MultipartFile,required=false表明不是必须的。
    
    if(bindingResult.hasErrors()) {
      return "spitters/edit";
    } 
    
    spitterService.saveSpitter(spitter);
    
    try {
      if(!image.isEmpty()) {
        validateImage(image);
        saveImage(spitter.getId() + ".jpg", image);
      }
    } catch (ImageUploadException e) {
      bindingResult.reject(e.getMessage());
      return "spitters/edit";
    }
 private void validateImage(MultipartFile image) {
    if(!image.getContentType().equals("image/jpeg")) {
      throw new ImageUploadException("Only JPG images accepted");
    }
  }                                                                         

```
![](/data/dokuwiki/spring/pasted/20150806-075901.png)
