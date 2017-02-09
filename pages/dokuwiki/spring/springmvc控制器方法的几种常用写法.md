title: springmvc控制器方法的几种常用写法 

#  springmvc控制器方法的几种常用写法 
参考：http://blog.csdn.net/z69183787/article/details/41653875
http://blog.csdn.net/z69183787/article/details/41653843
##  使用HttpServletRequest获取 
```

@RequestMapping("/login.do")  
public String login(HttpServletRequest request){  
    String name = request.getParameter("name")  
    String pass = request.getParameter("pass")  
} 

``` 
##  自动注入单个属性值 
```

Spring会自动将表单参数注入到方法参数，和表单的name属性保持一致。和Struts2一样
@RequestMapping("/login.do")  
public String login(HttpServletRequest request,  
                                String name,  
 @RequestParam("pass")String password) // 表单属性是pass,用变量password接收  
{  
   syso(name);  
   syso(password)  
}  

```
```

	@RequestMapping(value={"/","/home"},method=RequestMethod.GET)
	public ModelAndView showHomePage(@RequestParam(value="page",defaultValue=homePageNo+"" required=true) int pageNo,HttpServletRequest request,
            HttpServletResponse response){
		 ModelAndView mav=new ModelAndView("home");
		//表示每页显示2条记录，page表示当前网页
        PersonBean pageBean = personService.getPageBean(pageSize, pageNo);
        mav.addObject("pageBean", pageBean);
		return mav;
	}

```
```

RESTFul风格
 @RequestMapping(value = "/{username}", method = RequestMethod.PUT, 
                  headers = "Content-Type=application/json")  //服务端响应内容类型为json
  @ResponseStatus(HttpStatus.NO_CONTENT)    //http响应状态吗204
  public void updateSpitter(@PathVariable String username, 
                        @RequestBody Spitter spitter) {
    spitterService.saveSpitter(spitter);
  }

```

##  自动注入Bean属性 
```

<form action="login.do">  
用户名：<input name="name"/>  
密码：<input name="pass"/>  
<input type="submit" value="登陆">  
</form>  
//封装的User类  
public class User{  
  private String name;  
  private String pass;  
}  
@RequestMapping("/login.do")  
public String login(User user)  
{  
   syso(user.getName());  
   syso(user.getPass());  
} 

``` 

##  向页面传值： 
当Controller组件处理后，向jsp页面传值,传值实际上是传到了HttpServletRequest的属性上**，可以通过EL表达式${attrName}或者${requestScope.attrName}获取**
0，使用Map对象.
1，使用HttpServletRequest 和 Session  然后setAttribute()，就和Servlet中一样
2，使用ModelAndView对象
3，使用ModelMap对象
4，使用@ModelAttribute注解
```

  @RequestMapping(value={"/","/home"}, method=RequestMethod.GET)  //定义处理方法和url
  public String showHomePage(Map<String, Object> model) {   //参数中传入map类型的model，也可以是其他类型。
    model.put("spittles", 
              spitterService.getRecentSpittles(spittlesPerPage));
    return "home"; //返回视图逻辑名称
  }

```
```

	@RequestMapping(value={"/","/home"},method=RequestMethod.GET)
	public ModelAndView showHomePage(@RequestParam(value="page",defaultValue=homePageNo+"") int pageNo,HttpServletRequest request,
            HttpServletResponse response){
		 ModelAndView mav=new ModelAndView("home");
		//表示每页显示2条记录，page表示当前网页
        PersonBean pageBean = personService.getPageBean(pageSize, pageNo);
        mav.addObject("pageBean", pageBean);
		return mav;
	}

```
```

  @RequestMapping(method=RequestMethod.GET)  //没有指定路径，就为处理类的路径映射
  public String listSpitters(
          @RequestParam("page") int page, //前面一个page指定是表单的name，用于自动表单填充。
          @RequestParam(value="perPage", defaultValue="10") int perPage,
         Model model) {  //使用org.springframework.ui.Model作为模型
    model.put("spitters", spitterService.getAllSpitters()); //填充模型
    return "spitters/list";
  }

```
```

Spring Web MVC能够自动将请求参数绑定到功能处理方法的命令/表单对象上。
@RequestMapping(value = "/commandObject", method = RequestMethod.GET)  
public String toCreateUser(HttpServletRequest request, UserModel user) {  
    return "customer/create";  
}

``` 

```

@RequestMapping("/login.do")  
public　String login(String name,String pass ,ModelMap model){  
    User user  = userService.login(name,pwd);  
    model.addAttribute("user",user);  
    model.put("name",name);  
    return "success";  
} 

```

```

 使用@ModelAttribute示例
在Controller方法的参数部分或Bean属性方法上使用
@ModelAttribute数据会利用HttpServletRequest的Attribute传值到success.jsp中
Java代码  收藏代码
@RequestMapping("/login.do")  
public String login(@ModelAttribute("user") User user){  
    //TODO  
   return "success";  
}  
  
@ModelAttribute("name")  
public String getName(){  
    return name;  
} 

``` 
##  Spring MVC使用重定向 

```

Spring MVC 默认采用的是转发forward:/app/spitters/form来定位视图，如果要使用重定向，可以如下操作
1，使用RedirectView
2，使用redirect:前缀

public ModelAndView login(){  
   RedirectView view = new RedirectView("regirst.do");  
   return new ModelAndView(view);  
}  
 或者用如下方法，工作中常用的方法：

public String login(){  
    //TODO  
    return "redirect:regirst.do";  
} 

``` 
##  Session存储： 
可以利用HttpServletReequest的getSession()方法
```

@RequestMapping("/login.do")  
public String login(String name,String pwd  
                            ModelMap model,HttpServletRequest request){  
     User user = serService.login(name,pwd);  
     HttpSession session = request.getSession();  
     session.setAttribute("user",user);  
     model.addAttribute("user",user);  
     return "success";  
} 

``` 