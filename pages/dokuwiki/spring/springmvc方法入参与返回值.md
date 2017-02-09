title: springmvc方法入参与返回值 

#  springmvc方法入参与返回值的介绍 
更多可以的参考：http://jinnianshilongnian.iteye.com/blog/1698916
##  springmvc方法入参 

 Spring MVC 框架中你可以按任意顺序定义请求处理方法的入参（**除了 Errors 和 BindingResult 必须紧跟在命令对象/表单参数后面以外**），Spring MVC 会根据反射机制自动将对应的对象通过入参传递给请求处理方法。这种机制让开发者完全可以不依赖 Servlet API 开发控制层的程序，当请求处理方法需要特定的对象时，仅仅需要在参数列表中声明入参即可，不需要考虑如何获取这些对象，下面列举下spring mvc支持的处理方法参数。 

*** Java 基本数据类型 和String** 
默认情况下将按名称匹配的方式绑定到 URL 参数上，可以通过`  @RequestParam ` 注解改变默认的绑定规则 

*** Servlet API对象.** 
Request或者response 对象 (Servlet API)，选择任意的request或者response类型，**例如：ServletRequest 、 HttpServletRequest. HttpSession** 

***.java.util.Locale** 
用于获得当前的请求区域。 

*.java.io.**InputStream** / java.io.Reader for access to the request's content. This value is the raw InputStream/Reader as exposed by the Servlet API. 
值为原始的InputStream/Reader被Servlet API **可以借此访问 request 的内容** 

*.java.io.**OutputStream** / java.io.Writer for generating the response's content. This value is the raw OutputStream/Writer as exposed by the Servlet API. 
**用于产生响应的内容,以此操作 response 的内容** 

*.java.security.**Principal** containing the currently authenticated user. 
**包含当前认证的用户。** 

**命令/表单对象**

*.**@PathVariable** annotated parameters for access to URI template variables 
用于注解参数对应到地址栏的**变量参数** 


```

@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)  
public String findOwner(@PathVariable("ownerId") String ownerId, Model model) {  
  // implementation omitted  
} 

``` 

访问地址"/owners/{ownerId}" 指定了访问变量名称为ownerId，当控制器处理这个请求的时候，ownerId的值被设置到请求的地址栏， 
例如：当请求来自/owners/fred，值fred被绑定到访问方法的参数String ownerId 

***.@RequestParam** 
用于注解参数到Servlet 请求的参数，参数值对应到控制器方法中声明的参数。 
```

@RequestMapping(method = RequestMethod.GET)  
    public String setupForm(@RequestParam("id") int petId, ModelMap model) {  
        Pet pet = this.clinic.loadPet(petId);  
        model.addAttribute("pet", pet);  
        return "petForm";  
    } 

``` 

例如：请求的参数id对应到方法中的petId. 

***.@RequestHeader** 
注解参数用于具体的Servlet请求的http头部，参数值被转换到生命的方法参数类型。 
请求头部信息例如：Connection，Accept-Language、Accept-Charset等 
Java代码  收藏代码
```

public void displayHeaderInfo(@RequestHeader("Connection") String connection,
			@RequestHeader("Accept-Encoding") String encoding) {  
        System.out.println("connection:" + connection);  
        System.out.println("Accept-Encoding:" + encoding);  
}

```  

***.@RequestBody** 
***.HttpEntity<?>** 
*.java.util.**Map** / org.springframework.ui.**Model** / org.springframework.ui.ModelMap 
它绑定 Spring MVC 框架中每个请求所创建的潜在的**模型对象**，它们**可以被 Web 视图对象访问（如 JSP）.用于增强传递到web视图页面的model**，也就是说他们作为方法的参数后，页面视图就可以直接根据key来取得相应的值。 

*.org.springframework.validation.**Errors** / org.springframework.validation.**BindingResult** 
BindingResult继承于Errors，为属性列表中的命令/表单对象的校验结果,返回错误信息到页面，设置为方法参数后，在视图页面上可以直接获取。 

* org.springframework.web.bind.support.SessionStatus status handle for marking form processing as complete, which triggers the cleanup of session attributes that have been indicated by the @SessionAttributes annotation at the handler type level. 
可以通过该类型 status 对象显式结束表单的处理，这相当于触发 session 清除其中的通过 @SessionAttributes 定义的属性 
例子：使用 SessionStatus 控制 Session 级别的模型属性 
##  springmvc方法返回值 
spring mvc处理方法支持如下的返回方式：` ModelAndView, Model, ModelMap, Map,View, 代表逻辑视图名的String, void。 `下面将对具体的一一进行说明： 
**ModelAndView**   
@RequestMapping("/show1")  
```

public ModelAndView show1(HttpServletRequest request,  
           HttpServletResponse response) throws Exception {  
       ModelAndView mav = new ModelAndView("/demo2/show");  
       mav.addObject("account", "account -1");  
       return mav;  
} 

``` 

通过ModelAndView构造方法可以**指定返回的页面名称**，也可以通过` setViewName() `方法跳转到指定的页面 , 
使用` addObject() `设置需要返回的值，addObject()有几个不同参数的方法，可以默认和指定返回对象的名字。 
调用addObject()方法将值设置到一个名为ModelMap的类属性，ModelMap是LinkedHashMap的子类， 
具体请看类。 

**Model 是一个接口， 其实现类为ExtendedModelMap，继承了ModelMap类。** 

**Map ** 
Java代码  收藏代码
```

@RequestMapping("/demo2/show")  
    public Map<String, String> getMap() {  
        Map<String, String> map = new HashMap<String, String>();  
        map.put("key1", "value-1");  
        map.put("key2", "value-2");  
        return map;  
    }

```  

**在jsp页面中可直通过${key1}获得到值, map.put()相当于request.setAttribute方法。** 

**View 可以返回pdf excel等，暂时没详细了解。** 

**String 指定返回的视图页面名称**，结合设置的返回地址路径加上页面名称后缀即可访问到。 
**注意：如果方法声明了注解@ResponseBody ，则会直接将返回值输出到页面。** 
例如： 
```

@RequestMapping(value = "/something", method = RequestMethod.GET)  
@ResponseBody  
public String helloWorld()  {  
return "Hello World";  
}

```  

**上面的结果会将文本"Hello World "直接写到http响应流。** 

```

@RequestMapping("/welcome")  
public String welcomeHandler() {  
  return "center";  
} 

``` 

**对应的逻辑视图名为“center”，URL= prefix前缀+视图名称 +suffix后缀组成。** 

**void  如果返回值为空，则响应的视图页面对应为访问地址** 
@RequestMapping("/welcome")  
public void welcomeHandler() {}  

**此例对应的逻辑视图名为"welcome"。** 

**小结：** 
1.**使用 String 作为请求处理方法的返回值类型是比较通用的方法**，这样返回的逻辑视图名不会和请求 URL 绑定，具有很大的灵活性，**而模型数据又可以通过 ModelMap 控制。** 
2.使用void,map,Model 时，返回对应的逻辑视图名称真实url为：prefix前缀+视图名称 +suffix后缀组成。 
3.使用String,ModelAndView返回视图名称可以不受请求的url绑定，**ModelAndView可以设置返回的视图名称。**
```

 @Override
    public ModelAndView handleRequest(HttpServletRequest request,
            HttpServletResponse response) throws Exception {
        logger.info("SaveProductController called");
        ProductForm productForm = new ProductForm();
        // populate action properties
        productForm.setName(request.getParameter("name"));
        productForm.setDescription(request.getParameter("description"));
        productForm.setPrice(request.getParameter("price"));

        // create model
        Product product = new Product();
        product.setName(productForm.getName());
        product.setDescription(productForm.getDescription());
        try {
            product.setPrice(Float.parseFloat(productForm.getPrice()));
        } catch (NumberFormatException e) {
        }

        // insert code to save Product

        return new ModelAndView("/WEB-INF/jsp/ProductDetails.jsp", "product",
                product);
    }

```