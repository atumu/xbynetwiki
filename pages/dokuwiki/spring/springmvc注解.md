title: springmvc注解 

#  springmvc注解介绍 
更多可以参考：http://jinnianshilongnian.iteye.com/blog/1705701
spring的注解有很多，今天主要对如下几个spring mvc常用的注解进行一个介绍。 
二、Spring2.5引入注解式处理器支持，通过` @Controller 和 @RequestMapping `注解定义我们的处理器类。并且提供了一组强大的注解：
需要通过处理器映射DefaultAnnotationHandlerMapping和处理器适配器AnnotationMethodHandlerAdapter来开启支持@Controller 和 @RequestMapping注解的处理器。
(Spring 3.1中只需要在配置文件中添加` <mvc:annotation-driven/> `即可开启)
  * @Controller：用于标识是处理器类；
  * @RequestMapping：请求到处理器功能方法的映射规则；
  * @RequestParam：请求参数到处理器功能处理方法的方法参数上的绑定；
  * @ModelAttribute：请求参数到命令对象的绑定；
  * @SessionAttributes：用于声明session级别存储的属性，放置在处理器类上，通常列出模型属性（如@ModelAttribute）对应的名称，则这些属性会透明的保存到session中；
  * @InitBinder：自定义数据绑定注册支持，用于将请求参数转换到命令对象属性的对应类型；
 
三、Spring3.0引入**RESTful架构风格**支持(通过` @PathVariable `注解和一些其他特性支持),且又引入了更多的注解支持：
  * @CookieValue：cookie数据到处理器功能处理方法的方法参数上的绑定；
  * @RequestHeader：请求头（header）数据到处理器功能处理方法的方法参数上的绑定；
  * @RequestBody：**请求的body体的绑定**（通过HttpMessageConverter进行类型转换）；
  * @ResponseBody：处理器功能**处理方法的返回值作为响应体**（通过HttpMessageConverter进行类型转换）；
  * @ResponseStatus：定义处理器功能处理方法/异常处理器返回的状态码和原因；
  * @ExceptionHandler：注解式声明异常处理器；
  * @PathVariable：请求URI中的模板变量部分到处理器功能处理方法的方法参数上的绑定，从而支持RESTful架构风格的URI；

四、还有比如：
**JSR-303验证框架的无缝支持**（通过` @Valid `注解定义验证元数据）；
使用Spring 3开始的ConversionService进行类型转换（PropertyEditor依然有效），支持使用` @NumberFormat 和 @DateTimeFormat `来进行数字和日期的格式化；
HttpMessageConverter（Http输入/输出转换器，比如JSON、XML等的数据输出转换器）；
ContentNegotiatingViewResolver，内容协商视图解析器，它还是视图解析器，只是它支持根据请求信息将同一模型数据以不同的视图方式展示（如json、xml、html等），RESTful架构风格中很重要的概念（同一资源，多种表现形式）；
**Spring 3 引入 一个  mvc XML的命名空间用于支持mvc配置**，包括如：
  *  ` <mvc:annotation-driven> `：
自动注册基于注解风格的处理器需要的DefaultAnnotationHandlerMapping、AnnotationMethodHandlerAdapter
支持Spring3的ConversionService自动注册
支持JSR-303验证框架的自动探测并注册（只需把JSR-303实现放置到classpath）
自动注册相应的HttpMessageConverter（用于支持@RequestBody  和 @ResponseBody）（如XML输入输出转换器（只需将JAXP实现放置到classpath）、JSON输入输出转换器（只需将Jackson实现放置到classpath））等。
  * <mvc:interceptors>：注册自定义的处理器拦截器；
  * <mvc:view-controller>：和ParameterizableViewController类似，收到相应请求后直接选择相应的视图；
  * <mvc:resources>：逻辑**静态资源**路径到物理静态资源路径的支持；
  * <mvc:default-servlet-handler>：当在web.xml 中DispatcherServlet使用<url-pattern>/</url-pattern> 映射时，能映射静态资源（当Spring Web MVC框架没有处理请求对应的控制器时（如一些静态资源），转交给默认的Servlet来响应静态文件，否则报404找不到资源错误，）。
……等等。
五、Spring3.1新特性：
对Servlet 3.0的全面支持。
@EnableWebMvc：用于在基于Java类定义Bean配置中开启MVC支持，和XML中的<mvc:annotation-driven>功能一样；
新的@Contoller和@RequestMapping注解支持类：处理器映射RequestMappingHandlerMapping 和 处理器适配器RequestMappingHandlerAdapter组合来代替Spring2.5开始的处理器映射DefaultAnnotationHandlerMapping和处理器适配器AnnotationMethodHandlerAdapter，提供更多的扩展点，它们之间的区别我们在处理器映射一章介绍。
新的@ExceptionHandler 注解支持类：ExceptionHandlerExceptionResolver来代替Spring3.0的AnnotationMethodHandlerExceptionResolver，在异常处理器一章我们再详细讲解它们的区别。
 
@RequestMapping的"consumes" 和 "produces" 条件支持：用于支持@RequestBody 和 @ResponseBody，
1consumes指定请求的内容是什么类型的内容，即本处理方法消费什么类型的数据，如consumes="application/json"表示JSON类型的内容，Spring会根据相应的HttpMessageConverter进行请求内容区数据到@RequestBody注解的命令对象的转换；

2produces指定生产什么类型的内容，如produces="application/json"表示JSON类型的内容，Spring的根据相应的HttpMessageConverter进行请求内容区数据到@RequestBody注解的命令对象的转换，Spring会根据相应的HttpMessageConverter进行模型数据（返回值）到JSON响应内容的转换
 
URI模板变量增强：URI模板变量可以直接绑定到@ModelAttribute指定的命令对象、@PathVariable方法参数在视图渲染之前被合并到模型数据中（除JSON序列化、XML混搭场景下）。
@Validated：JSR-303的javax.validation.Valid一种变体（非JSR-303规范定义的，而是Spring自定义的），用于提供对Spring的验证器（org.springframework.validation.Validator）支持，需要Hibernate Validator 4.2及更高版本支持；
 
` @RequestPart `：提供对“` multipart/form-data `”请求的全面支持，支持Servlet 3.0文件上传（javax.servlet.http.Part）、支持内容的HttpMessageConverter（即根据请求头的Content-Type，来判断内容区数据是什么类型，如JSON、XML，能自动转换为命令对象），比@RequestParam更强大（只能对请求参数数据绑定，key-alue格式），而@RequestPart支持如JSON、XML内容区数据的绑定；

` Flash 属性 和 RedirectAttribute `：通过FlashMap存储一个请求的输出，当进入另一个请求时作为该请求的输入，典型场景如重定向（` POST-REDIRECT-GET `模式，1、POST时将下一次需要的数据放在FlashMap；2、重定向；3、通过GET访问重定向的地址，此时FlashMap会把1放到FlashMap的数据取出放到请求中，并从FlashMap中删除；**从而支持在两次请求之间保存数据并防止了重复表单提交**）。
Spring Web MVC提供FlashMapManager用于管理FlashMap，默认使用SessionFlashMapManager，**即数据默认存储在session中。**

**@Controller** 
@Controller 负责注册一个bean 到spring 上下文中，**bean 的ID 默认为类名称开头字母小写,你也可以自己指定**，如下 
方法一： 
@Controller 
public class TestController {} 
方法二：            
@Controller("tmpController") 
public class TestController {} 

##  @RequestMapping 详解 
建议参考：http://blog.csdn.net/kobejayandy/article/details/12690041
@RequestMapping
RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。
**RequestMapping注解有六个属性**，下面我们把她分成三类进行说明。
1、 value， method；
  * ` value `：     指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；
  * ` method `：  指定请求的method类型， GET、POST、PUT、DELETE等；

2、 consumes，produces；
  * ` consumes `： **指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;**
  * ` produces `:    **指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回**；

3、 params，headers；
  * ` params `： 指定request中必须包含某些参数值是，才让该方法处理。
  * ` headers `： 指定request中必须包含某些指定的header值，才能让该方法处理请求。


@RequestMapping 
1.**@RequestMapping**用来定义访问的URL，你可以为整个类定义一个@RequestMapping，或者为每个方法指定一个。 
把@RequestMapping放在类级别上，这可令它与方法级别上的@RequestMapping注解协同工作，取得缩小选择范围的效果。 
例如： 
@RequestMapping("/test") 
public class TestController {} 
则，该类下的所有访问路径都在/test之下。 

2.将@RequestMapping用于整个类不是必须的，如果没有配置，所有的方法的访问路径配置将是完全独立的，没有任何关联。 

3.完整的参数项为：` @RequestMapping(value="",method ={"",""},headers={},params={"",""})， `各参数说明如下：
value :String[] 设置访问地址 
method: RequestMethod[]设置访问方式，字符数组，查看RequestMethod类，包括GET, HEAD, POST, PUT, DELETE, OPTIONS, TRACE,常用` RequestMethod.GET `，RequestMethod.POST 
headers:String[] headers一般` 结合method = RequestMethod.POST `使用 
params: String[] **访问参数设置，字符数组 例如：userId=id** 

4.value的配置还可以采用模版变量的形式 ，例如：@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)，这点将在介绍` @PathVariable `中详细说明。 

5.@RequestMapping params的补充说明，你可以通过设置参数条件来限制访问地址，例如params="myParam=myValue"表达式，访问地址中参数只有包含了该规定的值"myParam=myValue"才能匹配得上，类似"myParam"之类的表达式也是支持的，表示当前请求的地址必须有该参数(参数的值可以是任意)，"!myParam"之类的表达式表明当前请求的地址不能包含具体指定的参数"myParam"。 

6.有一点需要注意的，如果为类定义了访问地址为*.do,*.html之类的，则在方法级的@RequestMapping，不能再定义value值，否则会报错，例如 
@RequestMapping("/bbs.do")  
public class BbsController {  
    @RequestMapping(params = "method=getList")  
    public String getList() {  
     return "list";  
    }  
@RequestMapping(value= "/spList")  
public String getSpecialList() {  
     return "splist";  
    }  
}  

如上例：/bbs.do?method=getList 可以访问到方法getList() ；而访问/bbs.do/spList则会报错. 

**@PathVariable** 
1.@PathVariable用于方法中的参数，表示方法参数绑定到地址URL的模板变量。 
例如： 
```

@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)  
public String findOwner(@PathVariable String ownerId, Model model) {  
  Owner owner = ownerService.findOwner(ownerId);    
  model.addAttribute("owner", owner);    
  return "displayOwner";  
} 

``` 

2.@PathVariable用于地址栏使用{xxx}模版变量时使用。 
如果@RequestMapping没有定义类似"/{ownerId}" ，这种变量，则使用在方法中@PathVariable会报错。 


**@ModelAttribute绑定请求参数到命令对象**
@ModelAttribute一个具有如下三个作用：
①**绑定请求参数到命令对象**：放在功能处理方法的入参上时，用于**将多个请求参数绑定到一个命令对象**，从而简化绑定流程，而且**自动暴露为模型数据用于视图页面展示时使用；**
②**暴露表单引用对象为模型数据**：放在处理器的一般方法（非功能处理方法）上时，是为表单准备要展示的表单引用对象，如注册时需要选择的所在城市等，而且在执行功能处理方法（@RequestMapping注解的方法）之前，自动添加到模型对象中，用于视图页面展示时使用；
③**暴露@RequestMapping方法返回值为模型数据**：放在功能处理方法的返回值上时，是暴露功能处理方法的返回值为模型数据，用于视图页面展示时使用。
```

一、绑定请求参数到命令对象
如用户登录，我们需要捕获用户登录的请求参数（用户名、密码）并封装为用户对象，此时我们可以使用@ModelAttribute绑定多个请求参数到我们的命令对象。
public String test1(@ModelAttribute("user") UserModel user)  
和6.6.1一节中的五、命令/表单对象功能一样。只是此处多了一个注解@ModelAttribute("user")，它的作用是将该绑定的命令对象以“user”为名称添加到模型对象中供视图页面展示使用。我们此时可以在视图页面使用${user.username}来获取绑定的命令对象的属性。
 
绑定请求参数到命令对象支持对象图导航式的绑定，如请求参数包含“?username=zhang&password=123&workInfo.city=bj”自动绑定到user中的workInfo属性的city属性中。
@RequestMapping(value="/model2/{username}")  
public String test2(@ModelAttribute("model") DataBinderTestModel model) {   
DataBinderTestModel相关模型请从第三章拷贝过来，请求参数到命令对象的绑定规则详见【4.16.1、数据绑定】一节，URI模板变量也能自动绑定到命令对象中，当你请求的URL中包含“bool=yes&schooInfo.specialty=computer&hobbyList[0]=program&hobbyList[1]=music&map[key1]=value1&map[key2]=value2&state=blocked”会自动绑定到命令对象上。
当URI模板变量和请求参数同名时，URI模板变量具有高优先权。
 
二、暴露表单引用对象为模型数据
@ModelAttribute("cityList")  
public List<String> cityList() {  
    return Arrays.asList("北京", "山东");  
}   
如上代码会在执行功能处理方法之前执行，并将其自动添加到模型对象中，在功能处理方法中调用Model 入参的containsAttribute("cityList")将会返回true。

@ModelAttribute("user")  //①  
public UserModel getUser(@RequestParam(value="username", defaultValue="") String username) {  
//TODO 去数据库根据用户名查找用户对象  
UserModel user = new UserModel();  
user.setRealname("zhang");  
     return user;  
}   
如你要修改用户资料时一般需要根据用户的编号/用户名查找用户来进行编辑，此时可以通过如上代码查找要编辑的用户。
也可以进行一些默认值的处理。

@RequestMapping(value="/model1") //②  
public String test1(@ModelAttribute("user") UserModel user, Model model)   
此处我们看到①和②有同名的命令对象，那Spring Web MVC内部如何处理的呢：
(1、首先执行@ModelAttribute注解的方法，准备视图展示时所需要的模型数据；@ModelAttribute注解方法形式参数规则和@RequestMapping规则一样，如可以有@RequestParam等；
（2、执行@RequestMapping注解方法，进行模型绑定时首先查找模型数据中是否含有同名对象，如果有直接使用，如果没有通过反射创建一个，因此②处的user将使用①处返回的命令对象。即②处的user等于①处的user。
 
三、暴露@RequestMapping方法返回值为模型数据
public @ModelAttribute("user2") UserModel test3(@ModelAttribute("user2") UserModel user)  
大家可以看到返回值类型是命令对象类型，而且通过@ModelAttribute("user2")注解，此时会暴露返回值到模型数据（名字为user2）中供视图展示使用。那哪个视图应该展示呢？此时Spring Web MVC会根据RequestToViewNameTranslator进行逻辑视图名的翻译，详见【4.15.5、RequestToViewNameTranslator】一节。
 
此时又有问题了，@RequestMapping注解方法的入参user暴露到模型数据中的名字也是user2，其实我们能猜到：
（3、@ModelAttribute注解的返回值会覆盖@RequestMapping注解方法中的@ModelAttribute注解的同名命令对象。
 
四、匿名绑定命令参数
public String test4(@ModelAttribute UserModel user, Model model)  
或  
public String test5(UserModel user, Model model)   
此时我们没有为命令对象提供暴露到模型数据中的名字，此时的名字是什么呢？Spring Web MVC自动将简单类名（首字母小写）作为名字暴露，如“cn.javass.chapter6.model.UserModel”暴露的名字为“userModel”。
public @ModelAttribute List<String> test6()  
或  
public @ModelAttribute List<UserModel> test7()   
对于集合类型（Collection接口的实现者们，包括数组），生成的模型对象属性名为“简单类名（首字母小写）”+“List”，如List<String>生成的模型对象属性名为“stringList”，List<UserModel>生成的模型对象属性名为“userModelList”。
 
其他情况一律都是使用简单类名（首字母小写）作为模型对象属性名，如Map<String, UserModel>类型的模型对象属性名为“map”。

```

**@ResponseBody** 
这个注解可以直接放在方法上，**表示返回类型将会直接作为HTTP响应字节流输出(不被放置在Model，也不被拦截为视图页面名称)。可以用于ajax。** 

**@RequestParam** 
@RequestParam是一个可选参数，例如：@RequestParam("id") 注解，所以它将和URL所带参数 id进行绑定 
如果入参是基本数据类型（如 int、long、float 等），URL 请求参数中一定要有对应的参数，否则将抛出 org.springframework.web.util.NestedServletException 异常，提示无法将 null 转换为基本数据类型. 

` @RequestParam包含3个配置 @RequestParam(required = ,value="", defaultValue = "") ` 
required :参数是否必须，boolean类型,可选项，默认为true 
value: 传递的参数名称，String类型,可选项，如果有值，对应到设置方法的参数 
defaultValue:String类型,参数没有传递时为参数默认指定的值 

**@SessionAttributes session管理** 
Spring 允许我们有选择地指定 ModelMap 中的哪些属性需要转存到 session 中，以便下一个请求属对应的 ModelMap 的属性列表中还能访问到这些属性。这一功能是通过类定义处标注 @SessionAttributes 注解来实现的。@SessionAttributes 只能声明在类上，而不能声明在方法上。 

例如 

@SessionAttributes("currUser") / / 将ModelMap 中属性名为currUser 的属性 
@SessionAttributes({"attr1","attr2"}) 
@SessionAttributes(types = User.class) 
@SessionAttributes(types = {User.class,Dept.class}) 
@SessionAttributes(types = {User.class,Dept.class},value={"attr1","attr2"}) 


**@CookieValue** 获取cookie信息 
```

@RequestMapping(value = "/testOne", method = RequestMethod.GET)
public String testOne(@CookieValue(value = "JSESSIONID", defaultValue = "123") String cookieStr,HttpServletRequest request)
{
   System.out.println("cookieStr = " + cookieStr);
   return null;
}

```
**@RequestHeader** 获取请求的头部信息
` @RequestHeader ` 注解，可以把Request请求header部分的值绑定到方法的参数上。 
```

@RequestMapping(value = "/example", method = RequestMethod.GET)  
   public String  getHello(@RequestHeader ("host") String hostName,  
        @RequestHeader ("Accept") String acceptType,  
        @RequestHeader ("Accept-Language") String acceptLang,  
        @RequestHeader ("Accept-Encoding") String acceptEnc,  
        @RequestHeader ("Cache-Control") String cacheCon,  
        @RequestHeader ("Cookie") String cookie,  
        @RequestHeader ("User-Agent") String userAgent)  
   {  
    System.out.println("Host : " + hostName);  
    System.out.println("Accept : " + acceptType);  
    System.out.println("Accept Language : " + acceptLang);  
    System.out.println("Accept Encoding : " + acceptEnc);  
    System.out.println("Cache-Control : " + cacheCon);  
    System.out.println("Cookie : " + cookie);  
    System.out.println("User-Agent : " + userAgent);  
       return "example";  
   }  

```