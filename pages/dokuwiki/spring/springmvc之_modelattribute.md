title: springmvc之_modelattribute 

#  SpringMVC之@ModelAttribute详解 
参考：http://hbiao68.iteye.com/blog/1948380
http://blog.csdn.net/xiejx618/article/details/43638537
http://blog.csdn.net/kobejayandy/article/details/12690161
先看一个没有使用@ModelAttribute的Controller方法.
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
表单参数会被注入到这个user

##  @ModelAttribute 
该注解有两个用法，一个是用于方法上，一个是用于参数上；
  * **用于方法上时**：  通常用来在处理` @RequestMapping `之前，为请求绑定需要从后台查询的model，注意，这个方法是非处理方法，但是在每次处理方法被调用前都会被调用。
  * **用于参数上时**： 用来**通过名称对应，把相应名称的值绑定到注解的参数bean上**；**` 要绑定的值来源于 `**：
A） ` @SessionAttributes ` 启用的attribute 对象上；
B） ` @ModelAttribute ` 用于方法上时指定的model对象；
C） 上述两种情况都没有时，new一个需要绑定的bean对象，**然后把request中按名称对应的方式把值绑定到bean中。**
**凡是在SpringMVC自动创建的model中的数据均可在前端通过EL表达式使用。**

` @ModelAttribute `绑定请求参数到命令对象
@ModelAttribute一个具有如下三个作用：
①**绑定请求参数到命令对象**：放在功能` 处理方法的入参 `上时，用于将多个请求` 参数绑定 `到一个命令对象，从而简化绑定流程，而且自动` 暴露为模型数据用于视图页面展示 `时使用；
②**暴露表单引用对象为模型数据**：放在` 非功能处理方法上 `时，是` 为表单准备要展示的表单引用对象 `，而且**在执行功能处理方法（@RequestMapping注解的方法）之前，自动添加到模型对象中，用于视图页面展示时使用；**
③**暴露@RequestMapping方法返回值为模型数据**：放` 在功能处理方法的返回值上 `时，是暴露功能处理方法的` 返回值为模型数据，用于视图页面展示时使用。 `

##  一、绑定请求参数到命令对象 

使用@ModelAttribute绑定多个` 请求参数 `到我们的命令对象。
` public String test1(@ModelAttribute("user") UserModel user)  ` 
只是此处多了一个注解@ModelAttribute("user")，**它的作用是将该绑定的命令对象以“user”为名称添加到模型对象中供视图页面展示使用**。
我们**此时可以在视图页面使用${user.username}来获取绑定的命令对象的属性**。
绑定请求参数到命令对象支持` 对象图导航式 `的绑定，如请求参数包含“?username=zhang&password=123&workInfo.city=bj”自动绑定到user中的workInfo属性的city属性中。

##  二、暴露表单引用对象为模型数据 
作用在**非请求**的处理方法上，` **被@ModelAttribute注解的方法会在每次调用该控制器类的请求处理方法时被调用** `。这意味着，如果一个控制器类有两个请求处理方法，以及一个@ModelAttribute注解的非请求方法，该方法的调用次数就会比每个请求处理方法更频繁。
` SpringMVC会在调用每个请求处理方法之前都会调用带@ModelAttribute注解的方法。 `
**@ModelAttribute注解的方法返回的对象会自动被加入到Spring MVC每次为调用请求处理方法创建的Model中。同时可以直接在前端页面通过EL表达式使用这个对象**
```

/** 
 * 设置这个注解之后可以直接在前端页面使用cityList这个对象（List）集合 
 * @return 
 */  
@ModelAttribute("cityList")  
public List<String> cityList() {  
    return Arrays.asList("北京", "山东");  
} 

```  
如上代码会在执行功能处理方法之前执行，并将其自动添加到模型对象中，在功能处理方法中调用Model 入参的containsAttribute("cityList")将会返回true。
 
```

  //①  
public @ModelAttribute("user") UserModel getUser(@RequestParam(value="username", defaultValue="") String username) {  
//TODO 去数据库根据用户名查找用户对象  
UserModel user = new UserModel();  
user.setRealname("zhang");  
     return user;  
}   
如你要修改用户资料时一般需要根据用户的编号/用户名查找用户来进行编辑，此时可以通过如上代码查找要编辑的用户。
也可以进行一些默认值的处理。
 
@RequestMapping(value="/model1") //②  
public String test1(@ModelAttribute("user") UserModel user, Model model)  

``` 
此处我们看到①和②有同名的命令对象，那Spring Web MVC内部如何处理的呢：
(1、**首先执行@ModelAttribute注解的方法**，准备视图展示时所需要的模型数据；@ModelAttribute注解方法形式参数规则和@RequestMapping规则一样，**如可以有@RequestParam等；**
（2、执行@RequestMapping注解方法，进行模型绑定时首先查找模型数据中是否含有同名对象，如果有直接使用，如果没有通过反射创建一个，因此②处的user将使用①处返回的命令对象。即②处的user等于①处的user。
 

**用在参数上的@ModelAttribute示例代码：**
```

@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)  
public String processSubmit(@ModelAttribute Pet pet) {  
     
} 

``` 
**查询顺序**
  * 首先查询`  @SessionAttributes `有无绑定的Pet对象，
  * 若没有则查询` @ModelAttribute `**方法层面**上是否绑定了Pet对象，
  * 若没有则将URI template中的**值按对应的名称绑定到Pet对象的各属性**上。
 
##  三、暴露@RequestMapping方法返回值为模型数据 

` public @ModelAttribute("user2") UserModel test3(@ModelAttribute("user2") UserModel user)  ` 
大家可以看到返回值类型是命令对象类型，而且通过@ModelAttribute("user2")注解，此时会暴露返回值到模型数据（名字为user2）中供视图展示使用。那哪个视图应该展示呢？此时Spring Web MVC会根据RequestToViewNameTranslator进行逻辑视图名的翻译，详见【4.15.5、RequestToViewNameTranslator】一节。
 
此时又有问题了，@RequestMapping注解方法的入参user暴露到模型数据中的名字也是user2，其实我们能猜到：
（3、@ModelAttribute注解的返回值会覆盖@RequestMapping注解方法中的@ModelAttribute注解的同名命令对象。


###  四、匿名绑定命令参数 

public String test4(@ModelAttribute UserModel user, Model model)  
或  
public String test5(UserModel user, Model model)   
此时我们没有为命令对象提供暴露到模型数据中的名字，此时的名字是什么呢？Spring Web MVC自动将简单类名（首字母小写）作为名字暴露，如“cn.javass.chapter6.model.UserModel”暴露的名字为“userModel”。
public @ModelAttribute List<String> test6()  
或  
public @ModelAttribute List<UserModel> test7()   
对于集合类型（Collection接口的实现者们，包括数组），生成的模型对象属性名为“简单类名（首字母小写）”+“List”，如List<String>生成的模型对象属性名为“stringList”，List<UserModel>生成的模型对象属性名为“userModelList”。
 
其他情况一律都是使用简单类名（首字母小写）作为模型对象属性名，如Map<String, UserModel>类型的模型对象属性名为“map”。