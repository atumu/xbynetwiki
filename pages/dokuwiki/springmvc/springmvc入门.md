title: springmvc入门 

#  SpringMVC入门 
本文参考：http://jinnianshilongnian.iteye.com/blog/系列文章
##  MVC概述 

![](/data/dokuwiki/springmvc/pasted/20150604-083814.png)![](/data/dokuwiki/springmvc/pasted/20150604-083825.png)
![](/data/dokuwiki/springmvc/pasted/20150604-083911.png)![](/data/dokuwiki/springmvc/pasted/20150604-084043.png)

##  Spring Web MVC是什么 
Spring Web MVC也是服务到工作者模式的实现，但进行可优化。前端控制器是**DispatcherServlet**；应用控制器其实拆为处理器映射器(Handler Mapping)进行处理器管理和视图解析器(View Resolver)进行视图管理；页面控制器/动作/处理器为Controller接口（仅包含ModelAndView handleRequest(request, response) 方法）的实现（也可以是任何的POJO类）；支持本地化（Locale）解析、主题（Theme）解析及文件上传等；提供了非常灵活的数据验证、格式化和数据绑定机制；提供了强大的约定大于配置（惯例优先原则）的契约式编程支持。
##  Spring Web MVC能帮我们做什么 

√让我们能非常简单的设计出干净的Web层和薄薄的Web层；
√进行更简洁的Web层的开发；
√天生与Spring框架集成（如IoC容器、AOP等）；
√提供强大的约定大于配置的契约式编程支持；
√能简单的进行Web层的单元测试；
√支持灵活的URL到页面控制器的映射；
√非常容易与其他视图技术集成，如Velocity、FreeMarker等等，因为模型数据不放在特定的API里，而是放在一个Model里（Map数据结构实现，因此很容易被其他框架使用）；
√非常灵活的数据验证、格式化和数据绑定机制，能使用任何对象进行数据绑定，不必实现特定框架的API；
` √提供一套强大的JSP标签库，简化JSP开发； `
√支持灵活的本地化、主题等解析；
√更加简单的异常处理；
` √对静态资源的支持； `
` √支持Restful风格。 `
##  Spring Web MVC架构 
![](/data/dokuwiki/springmvc/pasted/20150604-084601.png)![](/data/dokuwiki/springmvc/pasted/20150604-084622.png)
![](/data/dokuwiki/springmvc/pasted/20150604-124228.png)
![](/data/dokuwiki/springmvc/pasted/20150604-124210.png)
###  Spring工作流程描述 

1. 用户向服务器发送请求，请求被Spring 前端控制Servelt DispatcherServlet捕获；
2. DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；
3. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）
4.  提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：
 HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中
5.  Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；
6.  根据返回的ModelAndView，选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet ；
7. ViewResolver 结合Model和View，来渲染视图
8. 将渲染结果返回给客户端。

**Spring工作流程描述**
  * 为什么Spring只使用一个Servlet(DispatcherServlet)来处理所有请求？
详细见J2EE设计模式-前端控制模式
  * Spring为什么要结合使用HandlerMapping以及HandlerAdapter来处理Handler?
 符合面向对象中的单一职责原则，代码架构清晰，便于维护，最重要的是代码可复用性高。如HandlerAdapter可能会被用于处理多种Handler。
在此我们可以看出具体的**核心开发步骤**：
1、  DispatcherServlet在web.xml中的部署描述，从而拦截请求到Spring Web MVC
2、  HandlerMapping的配置，从而将请求映射到处理器
3、  HandlerAdapter的配置，从而支持多种类型的处理器
4、  ViewResolver的配置，从而将逻辑视图名解析为具体视图技术
5、处理器（页面控制器）的配置，从而进行功能处理
##  Hello World入门 
###  前端控制器的配置 

在我们的web.xml中添加如下配置：
```

<servlet>  
    <servlet-name>chapter2</servlet-name>  
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
    <load-on-startup>1</load-on-startup>  
</servlet>  
<servlet-mapping>  
    <servlet-name>chapter2</servlet-name>  
    <url-pattern>/</url-pattern>  
</servlet-mapping>  

```
自此请求已交给Spring Web MVC框架处理，因此我们需要配置Spring的配置文件，**默认DispatcherServlet会加载WEB-INF/[DispatcherServlet的Servlet名字]-servlet.xml配置文件。本示例为WEB-INF/ chapter2-servlet.xml**。
###  配置HandlerMapping、HandlerAdapter 
具体配置在WEB-INF/ chapter2-servlet.xml文件中：
```

<!-- HandlerMapping -->  
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>  
   
<!-- HandlerAdapter -->  
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>  

```
BeanNameUrlHandlerMapping：表示将请求的URL和Bean名字映射，如URL为 “上下文/hello”，则Spring配置文件必须有一个名字为“/hello”的Bean，上下文默认忽略。
SimpleControllerHandlerAdapter：表示所有实现了org.springframework.web.servlet.mvc.Controller接口的Bean可以作为Spring Web MVC中的处理器。如果需要其他类型的处理器可以通过实现HadlerAdapter来解决。
###  配置ViewResolver 
具体配置在WEB-INF/ chapter2-servlet.xml文件中：
```

<!-- ViewResolver -->  
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>  
    <property name="prefix" value="/WEB-INF/jsp/"/>  
    <property name="suffix" value=".jsp"/>  
</bean>  

```
InternalResourceViewResolver：用于支持Servlet、JSP视图解析；
  * viewClass：JstlView表示JSP模板页面需要使用JSTL标签库，classpath中必须包含jstl的相关jar包；
  * prefix和suffix：查找视图页面的前缀和后缀（**前缀[逻辑视图名]后缀**），比如传进来的逻辑视图名为hello，则该该jsp视图页面应该存放在“WEB-INF/jsp/hello.jsp”；

###  开发处理器/页面控制器 
```

import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import org.springframework.web.servlet.ModelAndView;  
import org.springframework.web.servlet.mvc.Controller;  
public class HelloWorldController implements Controller {  
    @Override  
    public ModelAndView handleRequest(HttpServletRequest req, HttpServletResponse resp) throws Exception {  
       //1、收集参数、验证参数  
       //2、绑定参数到命令对象  
       //3、将命令对象传入业务对象进行业务处理  
       //4、选择下一个页面  
       ModelAndView mv = new ModelAndView();  
       //添加模型数据 可以是任意的POJO对象  
       mv.addObject("message", "Hello World!");  
       //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面  
       mv.setViewName("hello");  
       return mv;  
    }  
}  

```
我们需要将其添加到Spring配置文件(WEB-INF/chapter2-servlet.xml)，让其接受Spring IoC容器管理:
```

<!-- 处理器 -->  
<bean name="/hello" class="cn.javass.chapter2.web.controller.HelloWorldController"/>  

```
###  开发视图页面 
创建 /WEB-INF/jsp/hello.jsp视图页面：
```

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>  
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">  
<title>Hello World</title>  
</head>  
<body>  
${message}  
</body>  
</html>  

```
${message}：表示显示由HelloWorldController处理器传过来的模型数据。
通过请求：http://localhost:9080/springmvc-chapter2/hello，如果页面输出“Hello World! ”就表明我们成功了！

**回忆下我们主要进行了如下配置：**
1、  前端控制器DispatcherServlet；
2、  HandlerMapping
3、  HandlerAdapter
4、  ViewResolver
5、  处理器/页面控制器
6、  视图
##  POST中文乱码解决方案 

spring Web MVC框架提供了**org.springframework.web.filter.CharacterEncodingFilter**用于解决POST方式造成的中文乱码问题，具体配置如下：
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
以后我们项目及所有页面的编码均为UTF-8。
##  Spring3.1基于注解配置新特性 
一、Spring2.5之前，我们都是通过实现Controller接口或其实现来定义我们的处理器类。
二、Spring2.5引入注解式处理器支持，通过**@Controller 和 @RequestMapping**注解定义我们的**处理器类**。并且提供了一组强大的注解：
需要通过处理器映射**DefaultAnnotationHandlerMapping**和处理器适配器**AnnotationMethodHandlerAdapter**来开启支持@Controller 和 @RequestMapping注解的处理器。
  * @Controller：用于标识是处理器类；
  * @RequestMapping：请求到处理器功能方法的映射规则；
  * @RequestParam：请求参数到处理器功能处理方法的方法参数上的绑定；
  * @ModelAttribute：请求参数到命令对象的绑定；
  * @SessionAttributes：用于声明session级别存储的属性，放置在处理器类上，通常列出模型属性（如@ModelAttribute）对应的名称，则这些属性会透明的保存到session中；
  * @InitBinder：自定义数据绑定注册支持，用于将请求参数转换到命令对象属性的对应类型；

##  Spring3.0引入RESTful架构风格支持 
(通过**@PathVariable**注解和一些其他特性支持),且又引入了更多的注解支持：
  * @CookieValue：cookie数据到处理器功能处理方法的方法参数上的绑定；
  * @RequestHeader：请求头（header）数据到处理器功能处理方法的方法参数上的绑定；
  * @RequestBody：请求的body体的绑定（通过HttpMessageConverter进行类型转换）；
  * @ResponseBody：处理器功能处理方法的返回值作为响应体（通过HttpMessageConverter进行类型转换）；
  * @ResponseStatus：定义处理器功能处理方法/异常处理器返回的状态码和原因；
  * @ExceptionHandler：注解式声明异常处理器；
  * @PathVariable：请求URI中的模板变量部分到处理器功能处理方法的方法参数上的绑定，从而支持**RESTful**架构风

##  Spring3.1其它新特性 
还有比如：
JSR-303验证框架的无缝支持（通过@Valid注解定义验证元数据）；
使用Spring 3开始的ConversionService进行类型转换（PropertyEditor依然有效），支持使用@NumberFormat 和 @DateTimeFormat来进行数字和日期的格式化；
HttpMessageConverter（Http输入/输出转换器，比如JSON、XML等的数据输出转换器）；
ContentNegotiatingViewResolver，内容协商视图解析器，它还是视图解析器，只是它支持根据请求信息将同一模型数据以不同的视图方式展示（如json、xml、html等），RESTful架构风格中很重要的概念（同一资源，多种表现形式）；
Spring 3 引入 一个  mvc XML的命名空间用于支持mvc配置，包括如：
 <mvc:annotation-driven>：
  *  自动注册基于注解风格的处理器需要的DefaultAnnotationHandlerMapping、AnnotationMethodHandlerAdapter
  *    支持Spring3的ConversionService自动注册
  *    支持JSR-303验证框架的自动探测并注册（只需把JSR-303实现放置到classpath）
  *    自动注册相应的HttpMessageConverter（用于支持@RequestBody  和 @ResponseBody）（如XML输入输出转换器（只需将JAXP实现放置到classpath）、JSON输入输出转换器（只需将Jackson实现放置到classpath））等。
 <mvc:interceptors>：注册自定义的处理器拦截器；
 <mvc:view-controller>：和ParameterizableViewController类似，收到相应请求后直接选择相应的视图；
 <mvc:resources>：逻辑静态资源路径到物理静态资源路径的支持；
 <mvc:default-servlet-handler>：当在web.xml 中DispatcherServlet使用<url-pattern>/</url-pattern> 映射时，能映射静态资源（当Spring Web MVC框架没有处理请求对应的控制器时（如一些静态资源），转交给默认的Servlet来响应静态文件，否则报404找不到资源错误，）。
……等等。

**对Servlet 3.0的全面支持。**
@EnableWebMvc：用于在基于Java类定义Bean配置中开启MVC支持，和XML中的<mvc:annotation-driven>功能一样；
新的@Contoller和@RequestMapping注解支持类：处理器映射RequestMappingHandlerMapping 和 处理器适配器RequestMappingHandlerAdapter组合来代替Spring2.5开始的处理器映射DefaultAnnotationHandlerMapping和处理器适配器AnnotationMethodHandlerAdapter，提供更多的扩展点，它们之间的区别我们在处理器映射一章介绍。
新的@ExceptionHandler 注解支持类：ExceptionHandlerExceptionResolver来代替Spring3.0的AnnotationMethodHandlerExceptionResolver，在异常处理器一章我们再详细讲解它们的区别。 
@RequestMapping的"consumes" 和 "produces" 条件支持：用于支持@RequestBody 和 @ResponseBody，
1consumes指定请求的内容是什么类型的内容，即本处理方法消费什么类型的数据，如consumes="application/json"表示JSON类型的内容，Spring会根据相应的HttpMessageConverter进行请求内容区数据到@RequestBody注解的命令对象的转换；
2produces指定生产什么类型的内容，如produces="application/json"表示JSON类型的内容，Spring的根据相应的HttpMessageConverter进行请求内容区数据到@RequestBody注解的命令对象的转换，Spring会根据相应的HttpMessageConverter进行模型数据（返回值）到JSON响应内容的转换
3以上内容，本章第×××节详述。
 
URI模板变量增强：URI模板变量可以直接绑定到@ModelAttribute指定的命令对象、@PathVariable方法参数在视图渲染之前被合并到模型数据中（除JSON序列化、XML混搭场景下）。 
@Validated：JSR-303的javax.validation.Valid一种变体（非JSR-303规范定义的，而是Spring自定义的），用于提供对Spring的验证器（org.springframework.validation.Validator）支持，需要Hibernate Validator 4.2及更高版本支持；
@RequestPart：提供对“multipart/form-data”请求的全面支持，支持Servlet 3.0文件上传（javax.servlet.http.Part）、支持内容的HttpMessageConverter（即根据请求头的Content-Type，来判断内容区数据是什么类型，如JSON、XML，能自动转换为命令对象），比@RequestParam更强大（只能对请求参数数据绑定，key-alue格式），而@RequestPart支持如JSON、XML内容区数据的绑定；详见本章的第×××节；
 
Flash 属性 和 RedirectAttribute：通过FlashMap存储一个请求的输出，当进入另一个请求时作为该请求的输入，典型场景如重定向（POST-REDIRECT-GET模式，1、POST时将下一次需要的数据放在FlashMap；2、重定向；3、通过GET访问重定向的地址，此时FlashMap会把1放到FlashMap的数据取出放到请求中，并从FlashMap中删除；从而支持在两次请求之间保存数据并防止了重复表单提交）。
Spring Web MVC提供FlashMapManager用于管理FlashMap，默认使用SessionFlashMapManager，即数据默认存储在session中。

##  DispatcherServlet详解 
DispatcherServlet是前端控制器设计模式的实现，提供Spring Web MVC的集中访问点，而且负责职责的分派，而且与Spring IoC容器无缝集成，从而可以获得Spring的所有好处。
**DispatcherServlet主要用作职责调度工作**，本身主要用于**控制流程**，主要职责如下：
1、文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；
2、通过HandlerMapping，将请求映射到处理器（返回一个HandlerExecutionChain，它包括一个处理器、多个HandlerInterceptor拦截器）；
3、通过HandlerAdapter支持多种类型的处理器(HandlerExecutionChain中的处理器)；
4、通过ViewResolver解析逻辑视图名到具体视图实现；
5、本地化解析；
6、渲染具体的视图等；
7、如果执行过程中遇到异常将交给HandlerExceptionResolver来解析。
该DispatcherServlet默认使用**WebApplicationContext**作为上下文，Spring默认配置文件为“/WEB-INF/[servlet名字]-servlet.xml”。
DispatcherServlet也可以配置自己的初始化参数，覆盖默认配置：
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

**DispatcherServlet中使用的特殊的Bean**
DispatcherServlet默认使用WebApplicationContext作为上下文，因此我们来看一下该上下文中有哪些特殊的Bean：
1、Controller：处理器/页面控制器，做的是MVC中的C的事情，但控制逻辑转移到前端控制器了，用于对请求进行处理；
2、HandlerMapping：请求到处理器的映射，如果映射成功返回一个HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象；如BeanNameUrlHandlerMapping将URL与Bean名字映射，映射成功的Bean就是此处的处理器；
3、HandlerAdapter：HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；如SimpleControllerHandlerAdapter将对实现了Controller接口的Bean进行适配，并且掉处理器的handleRequest方法进行功能处理；
4、ViewResolver：ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；如InternalResourceViewResolver将逻辑视图名映射为jsp视图；
5、LocalResover：本地化解析，因为Spring支持国际化，因此LocalResover解析客户端的Locale信息从而方便进行国际化；
6、ThemeResovler：主题解析，通过它来实现一个页面多套风格，即常见的类似于软件皮肤效果；
7、MultipartResolver：文件上传解析，用于支持文件上传；
8、HandlerExceptionResolver：处理器异常解析，可以将异常映射到相应的统一错误界面，从而显示用户友好的界面（而不是给用户看到具体的错误信息）；
9、RequestToViewNameTranslator：当处理器没有返回逻辑视图名等相关信息时，自动将请求URL映射为逻辑视图名；
10、FlashMapManager：用于管理FlashMap的策略接口，FlashMap用于存储一个请求的输出，当进入另一个请求时作为该请求的输入，通常用于重定向场景，后边会细述。
参考：
http://blog.csdn.net/xtu_xiaoxin/article/details/8796499