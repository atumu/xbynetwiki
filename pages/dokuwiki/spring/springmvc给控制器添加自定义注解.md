title: springmvc给控制器添加自定义注解 

#  SpringMVC给控制器添加自定义注解 
场景描述：现在需要对部分Controller或者Controller里面的服务方法进行权限拦截。如果存在我们自定义的注解，通过自定义注解提取所需的权限值，然后对比session中的权限判断当前用户是否具有对该控制器或控制器方法的访问权限。如果没有相关权限则终止控制器方法执行直接返回。有两种方式对这种情况进行处理。
  * 方式一：使用SpringAOP中的环绕Around
  * 方式二：使用Spring web拦截器

1 定义注解
```

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
//最高优先级
@Order(Ordered.HIGHEST_PRECEDENCE)
public @interface RoleControl {
    /**
     * 
     * 角色类型，以便决定是否具有相关权限
     */
    String value() default "user";
}

```
在Controller中使用
```

@RoleControl("ADMIN")
@RequestMapping("/test")
public String test(HttpServletRequest request, ModelMap modelMap) {
    //TODO
 }

```
##  方式一：使用SpringAOP中的环绕Around 
实现注解,定义一个切面进行拦截。关键点说明：
**@Before是在方法执行前的无法终止原方法执行,你用@Around这个是环绕通知.**
```

@Around("拦截表达式")
public Object around(ProceedingJoinPoint pjp){
   if(validation()){//你的校验成功执行方法,失败方法就不用执行了
     return pjp.proceed();
   }else{
     //可以返回你失败的信息也可以直接抛出校验失败的异常
   }
}

```
```

@Component
@Aspect
public class RoleControlAspect {
	@Autowired
	HttpSession session;
	/**类上注解情形 */
//	@Pointcut("@within(net.xby1993.springmvc.annotation.RoleControl)")
	@Pointcut("execution(* net.xby1993.springmvc.controller..*.*(..)) && @within(net.xby1993.springmvc.annotation.RoleControl)")
	public void aspect(){
		
	}
	/**方法上注解情形 */
	@Pointcut("execution(* net.xby1993.springmvc.controller..*.*(..)) && @annotation(net.xby1993.springmvc.annotation.RoleControl)")
	public void aspect2(){
		
	}
	/**aop实际拦截两种情形*/
	@Around("aspect() || aspect2()")
	public Object doBefore(ProceedingJoinPoint point) {
		Object target = point.getTarget();
		String method = point.getSignature().getName();
		Class<?> classz = target.getClass();
		Method m = ((MethodSignature) point.getSignature()).getMethod();
		try {
			if (classz!=null && m != null ) {
				boolean isClzAnnotation= classz.isAnnotationPresent(RoleControl.class);
				boolean isMethondAnnotation=m.isAnnotationPresent(RoleControl.class);
				RoleControl rc=null;
				//如果方法和类声明中同时存在这个注解，那么方法中的会覆盖类中的设定。
				if(isMethondAnnotation){
					rc=m.getAnnotation(RoleControl.class);
				}else if(isClzAnnotation){
					rc=classz.getAnnotation(RoleControl.class);
				}
				String value=rc.value();
				Object obj=session.getAttribute(GeneUtil.SESSION_USERTYPE_KEY);
				String curUserType=obj==null?"":obj.toString();
				//进行角色访问的权限控制，只有当前用户是需要的角色才予以访问。
				boolean isEquals=StringUtils.checkEquals(value, curUserType);
				if(isEquals){
					try {
						return point.proceed();
					} catch (Throwable e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
				
		    }
		}catch(Exception e){
			
		}
		return null;
	}
}

```

##  方式二：使用拦截器，推荐 
```

import java.lang.reflect.Method;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import net.xby1993.springmvc.annotation.RoleControl;
import net.xby1993.springmvc.util.GeneUtil;
import net.xby1993.springmvc.util.PathUtil;
import net.xby1993.springmvc.util.StringUtils;

public class GlobalInterceptor extends HandlerInterceptorAdapter{
	private static Logger log=LoggerFactory.getLogger(LoginInterceptor.class);
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		HttpSession s=request.getSession();
		s.setAttribute("host", PathUtil.getHost());
		s.setAttribute("siteName", GeneUtil.SITE_NAME);
		//角色权限控制访问
		roleControl(request,response,handler);
		return true;
	}
	/**角色权限控制访问*/
	private boolean roleControl(HttpServletRequest request,HttpServletResponse response, Object handler){
		HttpSession session=request.getSession();
		System.out.println(handler.getClass().getName());
		if(handler instanceof HandlerMethod){
			HandlerMethod hm=(HandlerMethod)handler;
			Object target=hm.getBean();
			Class<?> clazz=hm.getBeanType();
			Method m=hm.getMethod();
			try {
				if (clazz!=null && m != null ) {
					boolean isClzAnnotation= clazz.isAnnotationPresent(RoleControl.class);
					boolean isMethondAnnotation=m.isAnnotationPresent(RoleControl.class);
					RoleControl rc=null;
					//如果方法和类声明中同时存在这个注解，那么方法中的会覆盖类中的设定。
					if(isMethondAnnotation){
						rc=m.getAnnotation(RoleControl.class);
					}else if(isClzAnnotation){
						rc=clazz.getAnnotation(RoleControl.class);
					}
					String value=rc.value();
					Object obj=session.getAttribute(GeneUtil.SESSION_USERTYPE_KEY);
					String curUserType=obj==null?"":obj.toString();
					//进行角色访问的权限控制，只有当前用户是需要的角色才予以访问。
					boolean isEquals=StringUtils.checkEquals(value, curUserType);
					if(!isEquals){
						//401未授权访问
						response.setStatus(401);
						return false;
					}
			    }
			}catch(Exception e){
				
			}
		}
		
		return true;
	}

}


```