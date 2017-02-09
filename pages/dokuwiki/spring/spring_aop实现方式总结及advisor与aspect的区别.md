title: spring_aop实现方式总结及advisor与aspect的区别 

#  spring_aop实现方式总结及advisor与aspect的区别 
[jdk动态代理实现原理](http://blog.csdn.net/moreevan/article/details/11642445)
[Spring AOP 实现原理](http://blog.csdn.net/moreevan/article/details/11977115)
业务类接口
```

package cn.test.business;  
  
public interface Work {  
  
    public void doWork(String userName);  
}  

```
业务类实现
```

package cn.test.business;  
  
public class Worker implements Work{  
  
    @Override  
    public void doWork(String userName) {  
        System.out.println(userName + " is working !");  
    }  
}  

```
##  1. 基于注解实现Spring Aop 

注解实现切面类
```

@Aspect  
public class AopAnnotationTest {    
      
    @Pointcut("execution(* cn.test.business.*.*(..))")    
    private void anyMethod(){}//定义一个切入点   
      
    @Before("anyMethod() && args(name)")    
    public void doBefore(String name){    
        System.out.println("doBefore...");    
    }    
      
    @AfterReturning("anyMethod()")  
    public void doAfterReturning(){  
        System.out.println("doAfterReturning...");  
    }  
      
    @After("anyMethod()")  
    public void doAfter(){  
        System.out.println("doAfter...");  
    }  
      
    @Around("anyMethod()")  
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable{  
        System.out.println("begin doAround...");  
        Object object = joinPoint.proceed();  
        System.out.println("after doAround...");  
        return object;  
    }  
      
    @AfterThrowing("anyMethod()")  
    public void doThrow(){  
        System.out.println("意外通知");  
    }  
}    

```
spring配置文件：spring-aop.xml
```

<beans xmlns="http://www.springframework.org/schema/beans"  
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"  
 xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"  
 xsi:schemaLocation="http://www.springframework.org/schema/beans   
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-2.5.xsd  
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">  
   
          <bean id="work" class="cn.test.business.Worker"></bean>  
            
          <!-- aop注解 实现 -->  
          <aop:aspectj-autoproxy/>  
          <bean id="anno-beforeadvice" class="cn.test.aop.advice.annoation.impl.AopAnnotationTest"/>    
            
          <!-- 实现相应的Advice方法实现aop -->  
          <!-- <bean id="logBeforeAdvice" class="cn.test.aop.advice.inteface.impl.LogBeforeAdvice"></bean>  
          <bean id="logAfterReturnAdvice" class="cn.test.aop.advice.inteface.impl.LogAfterReturnAdvice"></bean>  
          <bean id="logExceptionAdvice" class="cn.test.aop.advice.inteface.impl.LogExceptionAdvice"></bean>  
          <bean id="logAroundAdvice" class="cn.test.aop.advice.inteface.impl.LogAroundAdvice"></bean>  
   
          <aop:config>  
                <aop:pointcut id="pointcut" expression="execution(* cn.test.business.*.*(..))" />  
                <aop:advisor advice-ref="logBeforeAdvice" pointcut-ref="pointcut"/>  
                <aop:advisor advice-ref="logAfterReturnAdvice" pointcut-ref="pointcut"/>  
                <aop:advisor advice-ref="logAroundAdvice" pointcut-ref="pointcut"/>  
                <aop:advisor advice-ref="logExceptionAdvice" pointcut-ref="pointcut"/>  
          </aop:config>    -->    
            
          <!-- 定义一个切面类 -->  
          <!-- <bean id="logAspect" class="cn.test.aop.advice.defineAspectClass.impl.TestAspect"></bean>  
          <aop:config>  
                  <aop:pointcut id="pointcut" expression="execution(* cn.test.business.*.*(..))" />  
                  <aop:aspect id="aspect" ref="logAspect">  
                          <aop:before pointcut-ref="pointcut" method="doBefore"/>  
                          <aop:after-returning pointcut-ref="pointcut" method="afterReturning" returning="retValue"/>  
                          <aop:after-throwing pointcut-ref="pointcut" method="doThrowing" throwing="ex"/>  
                          <aop:after pointcut-ref="pointcut" method="doAfter"/>  
                          <aop:around pointcut-ref="pointcut" method="doAround"/>  
                 </aop:aspect>  
          </aop:config> -->  
</beans>  

```
测试类：
```

package cn.test.aop.test;  
  
import org.junit.Test;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.test.context.ContextConfiguration;  
import org.springframework.test.context.junit4.AbstractJUnit4SpringContextTests;  
  
import cn.test.business.Work;  
  
  
@ContextConfiguration(  
    locations={  
        "classpath:/spring-aop.xml"  
    }  
)  
public class SpringAopTest extends AbstractJUnit4SpringContextTests{  
  
    @Autowired  
    private Work work;  
      
    @Test  
    public void aopTest(){  
        work.doWork("张三");  
    }  
}  

```
测试结果：
```

begin doAround...  
doBefore...  
张三 is working !  
after doAround...  
doAfter...  
doAfterReturning...  

```
##  2. 实现Adivce接口的方式实现Spring Aop 
定义前置通知
```

public class LogBeforeAdvice  implements MethodBeforeAdvice {  
  
    @Override  
    public void before(Method method, Object[] args, Object target)  
            throws Throwable {  
        System.out.println(args[0] + "开始工作!");  
    }  
}  

```
定义环绕通知
```

public class LogAroundAdvice implements MethodInterceptor{  
  
    @Override  
    public Object invoke(MethodInvocation arg0) throws Throwable {  
        System.out.println(arg0.getArguments()[0] + " 工作中，请勿打扰...");  
        Object obj = arg0.proceed();  
        System.out.println(arg0.getArguments()[0] + " 工作完成...");  
        return obj;  
    }  
}  

```
定义返回后通知
```

public class LogAfterReturnAdvice implements AfterReturningAdvice{  
  
    @Override  
    public void afterReturning(Object returnValue, Method method,  
            Object[] args, Object target) throws Throwable {  
        System.out.println(args[0] + "完成工作");  
    }  
}  

```
定义抛出异常后通知
```

public class LogExceptionAdvice implements ThrowsAdvice{  
  
    public void afterThrowing(Method method, Object[] parameters, Object target, Exception ex){  
        System.out.println(parameters[0] + " 工作中出现异常... ");  
    }  
}  

```
spring配置文件：只需要把上面的配置文件中第二部分打开即可。
```

<!-- 实现相应的Advice方法实现aop -->  
          <bean id="logBeforeAdvice" class="cn.test.aop.advice.inteface.impl.LogBeforeAdvice"></bean>  
          <bean id="logAfterReturnAdvice" class="cn.test.aop.advice.inteface.impl.LogAfterReturnAdvice"></bean>  
          <bean id="logExceptionAdvice" class="cn.test.aop.advice.inteface.impl.LogExceptionAdvice"></bean>  
          <bean id="logAroundAdvice" class="cn.test.aop.advice.inteface.impl.LogAroundAdvice"></bean>  
   
          <aop:config>  
                <aop:pointcut id="pointcut" expression="execution(* cn.test.business.*.*(..))" />  
                <aop:advisor advice-ref="logBeforeAdvice" pointcut-ref="pointcut"/>  
                <aop:advisor advice-ref="logAfterReturnAdvice" pointcut-ref="pointcut"/>  
                <aop:advisor advice-ref="logAroundAdvice" pointcut-ref="pointcut"/>  
                <aop:advisor advice-ref="logExceptionAdvice" pointcut-ref="pointcut"/>  
          </aop:config>  

```
##  3. 定义切面类的方式实现Spring Aop 
切面类
```

public class TestAspect {  
  
    public void doAfter(JoinPoint jp) {  
        System.out.println(jp.getArgs()[0] + " 回家...");  
    }  
  
    public void afterReturning (JoinPoint joinPoint, Object retValue) {  
        System.out.println(joinPoint.getArgs()[0] + " 结束工作...");  
    }  
      
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {  
        long time = System.currentTimeMillis();  
        Object retVal = pjp.proceed();  
        time = System.currentTimeMillis() - time;  
        System.out.println(pjp.getArgs()[0] +" 工作时间: " + time + " ms");  
        return retVal;  
    }  
  
    public void doBefore(JoinPoint jp) {  
        System.out.println(jp.getArgs()[0] + " 开始工作...");  
    }  
  
    public void doThrowing(JoinPoint jp, Throwable ex) {  
        System.out.println(jp.getArgs()[0] + " 工作中出现异常... " + ex);  
    }  
}   

```
```

<bean id="logAspect" class="cn.test.aop.advice.defineAspectClass.impl.TestAspect"></bean>  
          <aop:config>  
                  <aop:pointcut id="pointcut" expression="execution(* cn.test.business.*.*(..))" />  
                  <aop:aspect id="aspect" ref="logAspect">  
                          <aop:before pointcut-ref="pointcut" method="doBefore"/>  
                          <aop:after-returning pointcut-ref="pointcut" method="afterReturning" returning="retValue"/>  
                          <aop:after-throwing pointcut-ref="pointcut" method="doThrowing" throwing="ex"/>  
                          <aop:after pointcut-ref="pointcut" method="doAfter"/>  
                          <aop:around pointcut-ref="pointcut" method="doAround"/>  
                 </aop:aspect>  
          </aop:config>  

```
参考：http://blog.csdn.net/xianymo/article/details/41544937
