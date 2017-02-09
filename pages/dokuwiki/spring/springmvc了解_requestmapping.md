title: springmvc了解_requestmapping 

#  SpringMVC了解@RequestMapping与方法入参与返回类型 
使用@RequestMapping注解将请求、请求的Content-Type或者Accpet头、HTTP请求头、指定请求参数或头是否存在、或者这些的任意组合映射到控制器。

##  1使用@RequestMapping特性缩小请求匹配范围 
可以缩小到方法级别。这点与Struts2的Action是不同的。
注意：只有控制器类中指定的@RequestMapping才会被考虑，指定在它的父类上无效。因此不能在抽象类上添加该注解。
###  1.1 URL限制 
使用value特性可以指定任何` Ant风格的URL模式 `。这与Servlet URL映射只可以使用通配符作为开头或结尾(中间不能使用通配符)要灵活得多。
注意：@RequestMapping的value特性指定的URL是相对于DispatcherServlet映射的上下文的。
URL拼写规则.http://mydomain.com/应用上下文名称/对应的DispatcherServlet上下文映射路径/对应的@RequestMapping的@Controller类/对应的@RequestMapping的控制器类中的方法value特性
**注意：使用通配符时，更为具体的URL映射模式会覆盖非具体的**
一个@RequestMapping注解可以接受一个数组形式的value特性，用于映射一组URL
```

@Controller
@RequestMapping("/demo")
public class DemoController {
	@Autowired
	private MenuService menuService;
	
	@RequestMapping("/menu")
	@ResponseBody
	public List<Menu> getMenus(){
		List<Menu> list=menuService.getAllMenuForRole("ADMIN");
		return list;
	}
  	@RequestMapping("/menu/*")
	@ResponseBody
	public List<Menu> getMenuByNumber(){
		...
	}
	
}

```
##  1.2 HTTP请求方法限制 
import org.springframework.web.bind.annotation.RequestMethod;枚举常量
```

 @RequestMapping(value = "/dashboard", method = RequestMethod.GET)
    public String dashboard(Map<String, Object> model)
    {
        model.put("text", "This is a model attribute.");
        model.put("date", Instant.now());

        return "home/dashboard";
    }

```

###  1.3 请求参数限制 
 @RequestMapping的params特性。参数表达式可以有myParam=value, myParam!=value, 必须存在, ！myParam必须不存在。
 @RequestMapping(value = "/dashboard",params={"employee", "confirm=true"}

###  1.4 请求头限制 
 @RequestMapping的headers特性。
###  1.5 内容类型限制 
 @RequestMapping的` consumes特性和produces特性 `
  * consumes特性用于请求。对应于请求的Content-Type头
  * produces特性用于响应。对应于请求的Accept头

##  2 指定控制器方法参数 
控制器方法可以有任意数量的不同类型的参数。这些参数类型非常灵活。
###  2.1 标准Servlet类型 
  * HttpServletRequest
  * HttpServletResponse
  * HttpSession
  * InputStream或者Reader用于读取响应正文，但是不同同时使用二者。处理之后不应该关闭该对象
  * OutputStream或者Writer用于编写响应正文，但是不能同时使用二者。处理之后不应该关闭该对象
  * java.util.Locale,用于本地化
  * WebRequest. Spring类。用于请求属性和HTTP会话对象的操作。不需要直接使用ServletAPI。但是不能与HttpServletRequest、HttpServletResponse、HttpSession混用。
###  2.2 注解请求属性 
参数值类型转换一般有两种策略：java.beans.PropertyEditor、org.springframework.core.convert.converter.Converter.(Spring 3.2之后添加的接口)
参数类型声明方式：
  * @RequestParam注解指定特定命名的参数。默认情况下参数是必须的，可以将required特性设置为false，或者设定一个defaultValue特性。例如@RequestParam(value="name",required=false)String myName如果不指定该注解，默认请求中的名字与入参名字相同时匹配。
  * @RequestHeader
  * @PathVariable路径变量。@RequestHeader与@PathVariable都可以用于注解一个Map<String,String>,用于将所有参数注入map中。
```

 @RequestMapping(value = "/user/{userId}", method = RequestMethod.GET)
    @ResponseBody
    public User getUser(@PathVariable("userId") long userId)
    {
        User user = new User();
        user.setUserId(userId);
        user.setUsername("john");
        user.setName("John Smith");
        return user;
    }

```

###  2.3输入绑定表单对象 
使用一个普通的POJO JavaBean作为表单命令对象接收请求参数，直接将该类对象作为控制器方法参数，springmvc会自动注入需要的值。
```

 @RequestMapping(value = "create", method = RequestMethod.POST)
    public View create(HttpSession session, Form form) throws IOException
    {
    }


```
Spring也可以自动验证表单对象的细节，这意味着可以避免在控制器方法中内嵌验证逻辑。如果启用了bean验证，并且使用` @javax.validation.Valid `标记了表单对象参数，那么紧接着在表单对象参数之后参数的类型可以是org.springframework.validation.` Errors或者BingdinResult `.**(注意是紧接其后必须有两者之一，否则报错）**
```

 @RequestMapping(value = "create", method = RequestMethod.POST)
    public View create(HttpSession session, @Valid Form form，BindingResult validation) throws IOException
    {
    }


```
###  2.4请求正文转换和实体 
@RequestBody
HttpEntity<T>

###  2.5 Multipart文件上传请求数据 
文件上传的Content-Type为multipart/form-data. 
Content-Disposition为 file; filename="123.png"
@RequestPart支持文件上传的注解。（普通的转换器PropertyEditor与Converter只能转换简单类型。）
` @RequestPart `将使用HTTP消息转换器对其进行转换。转换的结果对象可以是javax.servlet.http**.Part或者spring内建的MultipartFile.**
@RequestPart("fileUpload") Part upload
如果是多文件上传可以直接使用**数组或者集合**形式
@RequestPart("fileUploads") List<MultipartFile> uploads
###  2.6模型类型 
SpringMVC架构中使用模型和视图。不过，这里应该注意的是，控制器方法可以有单个类型为Map<String,Object>、ModelMap、Model、ModelAndView的非标注参数。这些类型之一会被SpringMVC自动创建，代表了Spring传入到视图中用于渲染的模型，并且我们可以在方法执行时向其中添加任意的特性和内容。

##  3 为控制器方法选择有效的返回类型 
返回类型void告诉spring该方法将手动处理响应的写入，所以spring在该方法返回之后不再需要进一步对请求进行处理。
###  3.1模型类型 
控制器方法可以返回一个Map<String,Object>、ModelMap或者Model、ModelAndView。它们之一告诉Spring将返回类型识别为模型，并使用以配置的RequestToViewNameTranslator自动确定视图。
###  3.2视图类型 
返回View、ModelAndView、字符串。代表视图类型(如RedirectView或者自定义View)或逻辑视图名。如果返回逻辑视图名然后Spring通过ViewResolver进行解析。
###  3.3响应正文实体 
返回ResponseEntity<?>
使用@ResponseBody、@ResponseStatus（结合HttpStatus枚举常量）
注意：在指定了@ResponseBody注解后，返回类型的其他处理器将被忽略。因而不会进行视图解析。@ResponseBody总是优于所有其他控制器方法返回值处理器进行处理。

###  3.4任意返回类型 
如果没有使用@ResponseBody标注，则返回的值变成视图模型特性，名称为类名的驼峰形式（除非用@ModelAttribute注解）。
使用@ResponseBody标注，则返回的值被转换为响应内容。

###  3.5异步类型 
返回Callable<?>、DeferredResult<?>



Http消息转换器与内容协商：略。






