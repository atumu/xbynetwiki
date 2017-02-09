title: apache_shiro综合实例 

#  apache Shiro综合实例 
原文：http://jinnianshilongnian.iteye.com/blog/2037222
简单的实体关系图
![](/data/dokuwiki/shiro/pasted/20151031-235246.png)
简单数据字典
用户(sys_user)
![](/data/dokuwiki/shiro/pasted/20151031-235313.png)![](/data/dokuwiki/shiro/pasted/20151031-235326.png)
  * **资源**：表示菜单元素、页面按钮元素等；菜单元素用来显示界面菜单的，页面按钮是每个页面可进行的操作，如新增、修改、删除按钮；使用type来区分元素类型（如menu表示菜单，button代表按钮），priority是元素的排序，如菜单显示顺序；permission表示权限；如用户菜单使用user:*；也就是把菜单授权给用户后，用户就拥有了user:*权限；如用户新增按钮使用user:create，也就是把用户新增按钮授权给用户后，用户就拥有了user:create权限了；available表示资源是否可用，如菜单显示/不显示。
  * **角色：**role表示角色标识符，如admin，用于后台判断使用；description表示角色描述，如超级管理员，用于前端显示给用户使用；resource_ids表示该角色拥有的资源列表，即该角色拥有的权限列表（显示角色），即角色是权限字符串集合；available表示角色是否可用。
  * **组织机构：**name表示组织机构名称，priority是组织机构的排序，即显示顺序；available表示组织机构是否可用。
  * **用户：**username表示用户名；password表示密码；salt表示加密密码的盐；role_ids表示用户拥有的角色列表，可以通过角色再获取其权限字符串列表；locked表示用户是否锁定。

此处如资源、组织机构都是树型结构：
![](/data/dokuwiki/shiro/pasted/20151031-235414.png)
parent_id表示父编号，parent_ids表示所有祖先编号；如0/1/2/表示其祖先是2、1、0；其中根节点父编号为0。
为了简单性，如用户-角色，角色-资源关系直接在实体（用户表中的role_ids，角色表中的resource_ids）里完成的，没有建立多余的关系表，如要查询拥有admin角色的用户时，建议建立关联表，否则就没必要建立了。在存储关系时如role_ids=1,2,3,；多个之间使用逗号分隔。
用户组、组织机构组本实例没有实现，即可以把一组权限授权给这些组，组中的用户/组织机构就自动拥有这些角色/权限了；另外对于用户组可以实现一个默认用户组，如论坛，不管匿名/登录用户都有查看帖子的权限。
更复杂的权限请参考《JavaEE项目开发脚手架》：http://github.com/zhangkaitao/es。

**UserRealm实现 **    
```

public class UserRealm extends AuthorizingRealm {  
    @Autowired private UserService userService;  
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {  
        String username = (String)principals.getPrimaryPrincipal();  
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();  
        authorizationInfo.setRoles(userService.findRoles(username));  
        authorizationInfo.setStringPermissions(userService.findPermissions(username));  
        System.out.println(userService.findPermissions(username));  
        return authorizationInfo;  
    }  
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        String username = (String)token.getPrincipal();  
        User user = userService.findByUsername(username);  
        if(user == null) {  
            throw new UnknownAccountException();//没找到帐号  
        }  
        if(Boolean.TRUE.equals(user.getLocked())) {  
            throw new LockedAccountException(); //帐号锁定  
        }  
        return new SimpleAuthenticationInfo(  
                user.getUsername(), //用户名  
                user.getPassword(), //密码  
                ByteSource.Util.bytes(user.getCredentialsSalt()),//salt=username+salt  
                getName()  //realm name  
        );  
    }  
} 

``` 
 
**Web层控制器** 
```

@Controller  
public class IndexController {  
    @Autowired  
    private ResourceService resourceService;  
    @Autowired  
    private UserService userService;  
    @RequestMapping("/")  
    public String index(@CurrentUser User loginUser, Model model) {  
        Set<String> permissions = userService.findPermissions(loginUser.getUsername());  
        List<Resource> menus = resourceService.findMenus(permissions);  
        model.addAttribute("menus", menus);  
        return "index";  
    }  
}  

``` 
IndexController中查询菜单在前台界面显示，请参考相应的jsp页面；
```

@Controller  
public class LoginController {  
    @RequestMapping(value = "/login")  
    public String showLoginForm(HttpServletRequest req, Model model) {  
        String exceptionClassName = (String)req.getAttribute("shiroLoginFailure");  
        String error = null;  
        if(UnknownAccountException.class.getName().equals(exceptionClassName)) {  
            error = "用户名/密码错误";  
        } else if(IncorrectCredentialsException.class.getName().equals(exceptionClassName)) {  
            error = "用户名/密码错误";  
        } else if(exceptionClassName != null) {  
            error = "其他错误：" + exceptionClassName;  
        }  
        model.addAttribute("error", error);  
        return "login";  
    }  
}  

``` 
LoginController用于显示登录表单页面，**其中shiro authc拦截器进行登录，登录失败的话会把错误存到shiroLoginFailure属性中，**在该控制器中获取后来显示相应的错误信息。 
```

@RequiresPermissions("resource:view")  
@RequestMapping(method = RequestMethod.GET)  
public String list(Model model) {  
    model.addAttribute("resourceList", resourceService.findAll());  
    return "resource/list";  
} 

```  
在控制器方法上使用**@RequiresPermissions指定需要的权限信息**，其他的都是类似的，请参考源码。
 
**Web层标签库**
com.github.zhangkaitao.shiro.chapter16.web.taglib.Functions提供了函数标签实现，有根据编号显示资源/角色/组织机构名称，其定义放在src/main/webapp/tld/zhang-functions.tld。
**Web层异常处理器** 
```

@ControllerAdvice  
public class DefaultExceptionHandler {  
    @ExceptionHandler({UnauthorizedException.class})  
    @ResponseStatus(HttpStatus.UNAUTHORIZED)  
    public ModelAndView processUnauthenticatedException(NativeWebRequest request, UnauthorizedException e) {  
        ModelAndView mv = new ModelAndView();  
        mv.addObject("exception", e);  
        mv.setViewName("unauthorized");  
        return mv;  
    }  
}  

``` 
如果抛出UnauthorizedException，将被该异常处理器截获来显示没有权限信息。
 
Spring配置——spring-config.xml
定义了context:component-scan来扫描除web层的组件、dataSource（数据源）、事务管理器及事务切面等；具体请参考配置源码。
 
Spring配置——spring-config-cache.xml
定义了spring通用cache，使用ehcache实现；具体请参考配置源码。
 
Spring配置——spring-config-shiro.xml
定义了shiro相关组件。 
```

<bean id="userRealm" class="com.github.zhangkaitao.shiro.chapter16.realm.UserRealm">  
    <property name="credentialsMatcher" ref="credentialsMatcher"/>  
    <property name="cachingEnabled" value="false"/>  
</bean>  

``` 
userRealm组件禁用掉了cache，可以参考https://github.com/zhangkaitao/es/tree/master/web/src/main/java/com/sishuok/es/extra/aop实现自己的cache切面；**否则需要在修改如资源/角色等信息时清理掉缓存。** 

```

<bean id="sysUserFilter"   
class="com.github.zhangkaitao.shiro.chapter16.web.shiro.filter.SysUserFilter"/>

```   
sysUserFilter用于根据当前登录用户身份获取User信息放入request；然后就可以通过request获取User。 
```

<property name="filterChainDefinitions">  
  <value>  
    /login = authc  
    /logout = logout  
    /authenticated = authc  
    /** = user,sysUser  
  </value>  
</property>  

``` 
如上是shiroFilter的filterChainDefinitions定义。 
Spring MVC配置——spring-mvc.xml
定义了spring mvc相关组件。 
```

<mvc:annotation-driven>  
  <mvc:argument-resolvers>  
    <bean class="com.github.zhangkaitao.shiro.chapter16  
        .web.bind.method.CurrentUserMethodArgumentResolver"/>  
  </mvc:argument-resolvers>  
</mvc:annotation-driven>  

``` 
此处注册了一个@CurrentUser参数解析器。如之前的IndexController，从request获取shiro sysUser拦截器放入的当前登录User对象。
 
Spring MVC配置——spring-mvc-shiro.xml
定义了spring mvc相关组件。 
```

<aop:config proxy-target-class="true"></aop:config>  
<bean class="org.apache.shiro.spring.security  
    .interceptor.AuthorizationAttributeSourceAdvisor">  
  <property name="securityManager" ref="securityManager"/>  
</bean>  

``` 
定义aop切面，用于代理如@RequiresPermissions注解的控制器，进行权限控制。
 
web.xml配置文件
定义Spring ROOT上下文加载器、ShiroFilter、及SpringMVC拦截器。具体请参考源码。
 
JSP页面       
```

<shiro:hasPermission name="user:create">  
    <a href="${pageContext.request.contextPath}/user/create">用户新增</a><br/>  
</shiro:hasPermission>  

``` 
使用shiro标签进行权限控制。具体请参考源码。
系统截图
访问http://localhost:8080/chapter16/；