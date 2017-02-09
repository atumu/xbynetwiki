createAt:2017-01-06 13:41:03
author:xbynet
modifyAt:2017-01-06 13:41:03
location:dokuwiki/spring/spring_aop_aspectj切入点语法
title:Spring AOP AspectJ切入点表达式语法

#  Spring AOP AspectJ切入点表达式语法 
另一篇关于AOP的文章在[[spring:spring3核心]]
##  概念 
**切面（Aspect）**： 一个关注点的模块化，这个关注点可能会横切多个对象。事务管理是J2EE应用中一个关于横切关注点的很好的例子。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @Aspect 注解（@AspectJ风格）来实现。

**连接点（Joinpoint）**： 在程序执行过程中某个特定的点，比如某方法调用的时候或者处理异常的时候。 在Spring AOP中，一个连接点 总是 代表一个方法的执行。 通过声明一个 org.aspectj.lang.JoinPoint 类型的参数可以使通知（Advice）的主体部分获得连接点信息。

**通知（Advice）**： 在切面的某个特定的连接点（Joinpoint）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。 通知的类型将在后面部分进行讨论。许多AOP框架，包括Spring，都是以拦截器做通知模型，并维护一个以连接点为中心的拦截器链。

**切入点（Pointcut）**： 匹配连接点（Joinpoint）的断言。通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行（例如，当执行某个特定名称的方法时）。 切入点表达式如何和连接点匹配是AOP的核心：Spring缺省使用AspectJ切入点语法。

**引入（Introduction）**： （也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。 **Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。** 例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（advised） 对象。 既然**Spring AOP是通过运行时代理实现的**，这个对象永远是一个 被代理（proxied） 对象。
AOP代理（AOP Proxy）： AOP框架创建的对象，用来实现切面契约（aspect contract）（包括通知方法执行等功能）。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。 注意：Spring 2.0最新引入的基于模式（schema-based）风格和@AspectJ注解风格的切面声明，对于使用这些风格的用户来说，代理的创建是透明的。
**织入（Weaving）**： 把切面（aspect）连接到其它的应用程序类型或者对象上，**并创建一个被通知（advised）的对象。** 这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。 Spring和其他纯Java AOP框架一样，**在运行时完成织入。**

**通知的类型：**
前置通知（Before advice）： 在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。
返回后通知（After returning advice）： 在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
抛出异常后通知（After throwing advice）： 在方法抛出异常退出时执行的通知。
后通知（After (finally) advice）： 当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
**环绕通知（Around Advice）**： 包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。
环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。
##  切入点介绍 
` Spring目前仅支持使用方法调用作为连接点（join point） `
Pointcut可以有下列方式来定义或者**通过` && || 和! `的方式进行组合.** 
  * args() 用于匹配当前执行的方法传入的**参数**为指定类型的执行方法；
  * @args() 用于匹配当前执行的方法传入的参数持有指定注解的执行； 
  * execution() 用于**匹配方法**执行的连接点；
  * this() 用于匹配**当前AOP代理对象类型**的执行方法；注意是AOP代理对象的类型匹配，这样就可能**包括引入接口也类型匹配；**
  * target() 用于匹配**当前目标对象类型**的执行方法；注意是 目标对象的类型匹配，这样就**不包括引入接口也类型匹配**；
  * @target()  用于匹配当前目标对象类型的执行方法，其中**目标对象持有指定的注解**；
  * within() 用于匹配**指定类型**内的方法执行；
  * @within() 用于匹配所以持有**指定注解类型类**内的方法；
  * @annotation 用于匹配当前执行方法**持有指定注解的方法**
  * bean Spring AOP扩展的，AspectJ没有对于指示符，用于匹配**特定名称的Bean对象**的执行方法；  
  * reference pointcut  表示**引用其他命名切入点**，**只有@ApectJ风格支持，Schema风格不支持**。
 execution()比较常用，其格式为:
```

execution(annotation? modifiers-pattern? return-type-pattern declaring-type-pattern? name-pattern(param-pattern)throws-pattern?)
  即注解？ 访问修饰？ 返回类型 类名？ 方法名(方法参数) 异常列表？

```
  * returning type pattern,name pattern, and parameters pattern是必须的.` 即返回值类型，方法名，方法参数是必须指定的。其余可选 `
  * ret-type-pattern:可以为` *表示任何返回值 `,全路径的类名等.
  * name-pattern:指定方法名,` *代表所以,set*,代表以set开头的所有方法. `
  * parameters pattern:指定方法参数(声明的类型),` (..)代表所有参数,(*)代表一个参数,(*,String)代表第一个参数为任何值,第二个为String类型. `

##  AspectJ类型匹配的通配符 
  * *：匹配任何数量字符；          
  * ..：匹配任何数量字符的重复，如在类型模式中匹配任何数量**子包**；而在方法参数模式中匹配任何数量参数。           
  * +：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。
##  组合切入点表达式 
AspectJ使用 且` （&&）、或（||）、非（！） `来组合切入点表达式。        
在Schema风格下，由于在XML中使用“&&”需要使用转义字符“&amp;&amp;”来代替之，所以很不方便，因此Spring AOP 提供了` and、or、not `来代替&&、||、！。
**举例说明:（注意这里的区别和理解，重要）**
**1、execution**
  * 任意公共**方法**的执行：
  * execution(public * *(..))
  * 任何一个以“set”开始的方法的执行：
  * execution(* set*(..))
  * AccountService 接口的任意方法的执行：
  * execution(* com.xyz.service.AccountService.*(..))
  * 定义在service包里的任意方法的执行：
  * execution(* com.xyz.service.*.*(..))
  * 定义在service包和所有子包里的任意类的任意方法的执行：
  * execution(* com.xyz.service..*.*(..))
  * 定义在pointcutexp包和所有子包里的JoinPointObjP2类的任意方法的执行：
  * execution(* com.test.spring.aop.pointcutexp..JoinPointObjP2.*(..))")

**2、within**
  * pointcutexp包里的任意**类**.
  * within(com.test.spring.aop.pointcutexp.*)
  * pointcutexp包和所有子包里的任意类.
  * within(com.test.spring.aop.pointcutexp..*)
  * within(@cn.javass..Secure *) **持有cn.javass..Secure注解**的任何**类型**的任何方法  ` 必须是在目标对象上声明这个注解，在接口上声明的对它不起作用 `否则用this来替代within

**3、this** ` 注意this和target中使用的表达式必须是类型全限定名，不支持通配符； `
  * 实现了Intf接口的所有**类**,如果Intf不是接口,限定Intf单个类.
  * this(com.test.spring.aop.pointcutexp.Intf)

**4、@within、@target** `  注解类型也必须是全限定类型名； `
  * 带有@Transactional标注的**所有类**的任意方法.
  * @within(org.springframework.transaction.annotation.Transactional)
  * @target(org.springframework.transaction.annotation.Transactional)

**5、@annotation** ` 注解类型也必须是全限定类型名 `
  * 带有@Transactional标注的任意**方法**.
  * @annotation(org.springframework.transaction.annotation.Transactional)
**@within和@target` 针对类 `的注解,@annotation是` 针对方法 `的注解**

**6、@args、args**  ` 参数类型列表中的参数必须是类型全限定名，通配符不支持 `；args属于动态切入点，这种切入点开销非常大，**非特殊情况最好不要使用；**
  * **参数**带有@Transactional标注的方法.
  * @args(org.springframework.transaction.annotation.Transactional)
  * 参数为String类型(运行是决定)的方法.
  * args(String)

##  配置方式 
Pointcut 可以通过Java注解和XML两种方式配置,如下所示:
```

方式一、XML
<aop:config>  
    <aop:aspect ref="aspectDef">  
        <aop:pointcutid="pointcut1"expression="execution(* com.test.spring.aop.pointcutexp..JoinPointObjP2.*(..))"/>  
        <aop:before pointcut-ref="pointcut1" method="beforeAdvice" />  
        <aop:after-returning pointcut-ref="pointcut1" method="applaud" /> 
    </aop:aspect>  
     <!--aop:aspect可以配置多个  -->
    <aop:advisor advice-ref="serviceAdvice" pointcut-ref="servicePointcut" />
</aop:config>  
  
方式二、JAVA类
@Component
@Aspect
public class DataSourceAspect {

	// 配置切入点,该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点
	@Pointcut("execution(* com.css..service..*(..))")
	public void aspect() {
	}
	//@Before("aspect() && args(name)")  
    	//public void doAccessCheck(String name){  
        //	System.out.println(name);  
        //	System.out.println("前置通知");  
    	//}  
	/**
	 * 如果已经设置为写库，则保持写库；如果还未设置，则按照注解进行设置；如果没有注解，则设置为写库。
	 * @param point
	 */
	@Before("aspect()")
	public void doBefore(JoinPoint point) {
		String type = DynamicDataSourceHolder.getDataSourceType();
		if(MultiDataSource.WRITE.equalsIgnoreCase(type)){
			return;
		}else{

			Object target = point.getTarget();
			String method = point.getSignature().getName();

			Class<?> classz = target.getClass();
			try {
				Method m = ((MethodSignature) point.getSignature())
					.getMethod()
				if (m != null && m.isAnnotationPresent(DataSource.class)) {
					DataSource data = m.getAnnotation(DataSource.class);
					DynamicDataSourceHolder.setDataSourceType(data.value());
					System.out.println(data.value());
				}else{
					DynamicDataSourceHolder.setDataSourceType(MultiDataSource.WRITE);
				}

			} catch (Exception e) {
				// TODO: handle exception
			}
		}
	}
}


```

**` 注意 `：**编写了注解形式的切面之后，我们需要把切面类声明为Spring应用上下文中的一个Bean.如果注解方式` 使用@Aspect外加一个@Component注解 `。
**声明spring自动可以识别@Aspect注解及相关注解然后自动代理:**
```

<aop:aspectj-autoproxy proxy-target-class="true" /> <!--  支持注解AOP，基于CGLIB代理 -->

```
##  通知参数JoinPoint          
如果想获取被被通知方法信息，该如何实现呢？接下来我们将介绍两种获取通知参数的方式。    
###  使用JoinPoint获取： 
Spring AOP提供使用org.aspectj.lang.JoinPoint类型获取连接点数据，**任何通知方法的第一个参数都可以是JoinPoint**(` 环绕通知是ProceedingJoinPoint，JoinPoint子类 `)，当然第一个参数位置也可以是JoinPoint.StaticPart类型，这个只返回连接点的静态部分。  
**1) JoinPoint**：提供访问当前被通知方法的目标对象、代理对象、方法参数等数据：     
```

package org.aspectj.lang;    
import org.aspectj.lang.reflect.SourceLocation;   
public interface JoinPoint {    
     String toString();         //连接点所在位置的相关信息   
     String toShortString();     //连接点所在位置的简短相关信息   
     String toLongString();     //连接点所在位置的全部相关信息   
     Object getThis();         //返回AOP代理对象   
     Object getTarget();       //返回目标对象,也就是被拦截的对象。    
     Object[] getArgs();       //返回被通知方法参数列表   
     Signature getSignature();  //返回当前连接点签名，它可以被强制转换为MethodSignature    
     SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置   
     String getKind();        //连接点类型    
     StaticPart getStaticPart(); //返回连接点静态部分   
 }

```
一般会这样用
```

@Component
@Aspect
public class RoleControlAspect {
	@Pointcut("execution(* net.xby1993.springmvc.controller..*.*(..)) && @within(net.xby1993.springmvc.annotation.RoleControl)")
	public void aspect(){
		
	}
	@Pointcut("@within(net.xby1993.springmvc.annotation.RoleControl)")
	public void aspect2(){
		
	}
	@Before("aspect2()")
	public void doBefore(JoinPoint point) {
		Object target = point.getTarget();
		String method = point.getSignature().getName();
		Class<?> classz = target.getClass();
		Method m = ((MethodSignature) point.getSignature()).getMethod();
		try {
			if (m != null && m.isAnnotationPresent(RoleControl.class)) {
				RoleControl rc=m.getAnnotation(RoleControl.class);
				String value=rc.value();
				
		    }
		}catch(Exception e){
			
		}

	}
}

```
###  ProceedingJoinPoint：用于环绕通知 
**使用 ` proceed() `方法来执行目标方 法：**  
```

 public interface ProceedingJoinPoint extends JoinPoint {   
     public Object proceed() throws Throwable;  
     public Object proceed(Object[] args) throws Throwable;   
 }

```  
使用如下方式在通知方法上声明，**必须是在第一个参数**，然后使用jp.getArgs()就能获取到被通知方法参数：
```

@Before(value="execution(* sayBefore(*))")   
 public void before(JoinPoint jp) {}   
@Before(value="execution(* sayBefore(*))")  
 public void before(JoinPoint.StaticPart jp) {}

```

###  参数自动获取 
通过切入点表达式**可以将相应的参数自动传递给通知方法**， 在Spring AOP中，除了execution和bean指示符不能传递参数给通知方法，其他指示符都可以将匹配的相应参数或对象自动传递给通知方法。    
```

 @Before(value="execution(* test(*)) && args(param)", argNames="param") 
 public void before1(String param) {    
      System.out.println("===param:" + param);    
  }

``` 
args(param)将首先查找通知方法上同名的参数，并在方法执行时（运行时）匹配传入的参数是使用该同名参数类型，即java.lang.String；如果匹配将把该被通知参数传递给通知方法上同名参数。
其他指示符（**除了execution和bean指示符**）都可以使用这种方式进行参数绑定。

##  Spring XML配置AOP 
在Spring配置文件中，所以**AOP相关定义必须放在<aop:config>标签下**，该标签下可以有` <aop:pointcut>、<aop:advisor>、<aop:aspect> `标签，配置顺序不可变。   
<aop:pointcut>：用来定义切入点，该切入点可以重用； 
<aop:advisor>：用来定义**只有一个通知和一个切入点**的切面；    
<aop:aspect>：用来定义切面，该切面可以**包含多个切入点和通知**，而且标签内部的通知和切入点定义是无序的；**和advisor的区别就在此，advisor只包含一个通知和一个切入点**。
![](/data/dokuwiki/spring/pasted/20151116-160744.png)
参考：
http://wenku.baidu.com/link?url=CvR3rZSqrjIglohZTsNrPBr6KMALTHT3VGLAT_F1pJzeVFxbNAOaIpvqdtJoDiKeZSQz6Iltktl9qXx7JX7jHIFsCnSRvZsZmILhc682gHu
http://blog.csdn.net/kkdelta/article/details/7441829
http://lavasoft.blog.51cto.com/62575/172292/