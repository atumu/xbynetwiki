title: spring3核心 

#  spring3核心 
##  概述 

Spring的主要功能：依赖注入(DI)与面向切面编程(AOP),当然还有许多其他特性如模板，SpringMVC等
Spring是基于POJO的轻量级开发框架。可以采用基于XML或注解的方式进行配置。
Spring的核心思路-简化Java开发：
![](/data/dokuwiki/spring/pasted/20150805-024338.png)

###  依赖注入示例： 

```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="knight" class="com.springinaction.knights.BraveKnight">
    <constructor-arg ref="quest" /> <!--<co id="co_inject_quest_bean"/>-->       
  </bean>

  <bean id="quest"
        class="com.springinaction.knights.SlayDragonQuest" /><!--<co id="co_quest_bean"/>-->
      
</beans>

```
```

public class BraveKnight implements Knight {
  private Quest quest;
  
  public BraveKnight(Quest quest) { // 构造器注入，Quest将被注入进来。
    this.quest = quest;       //<co id="co_injectedQuest"/>
  }
  
  public void embarkOnQuest() throws QuestException {
    quest.embark();
  }
}


```
主类启动Spring容器，并获取已经被注入好的bean对象。（Spring中bean对象默认为相对Spring容器为单例，只会实例化一次）
```

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class KnightMain {
  public static void main(String[] args) {
    ApplicationContext context = 
        new ClassPathXmlApplicationContext("knights.xml"); //spring容器初始化。
    
    Knight knight = (Knight) context.getBean("knight"); //获取被注入的bean对象。
    
    knight.embarkOnQuest();//<co id="co_useKnight"/>
  }
}

```
###  应用切面示例 
![](/data/dokuwiki/spring/pasted/20150805-025513.png)
![](/data/dokuwiki/spring/pasted/20150805-025540.png)
传统方式：
![](/data/dokuwiki/spring/pasted/20150805-025606.png)
AOP方式:
![](/data/dokuwiki/spring/pasted/20150805-025638.png)
切面类:
```

public class Minstrel {
  public void singBeforeQuest() {     //<co id="co_singBefore"/>
    System.out.println("Fa la la; The knight is so brave!");
  }
  
  public void singAfterQuest() {     //<co id="co_singAfter"/>
    System.out.println(
            "Tee hee he; The brave knight did embark on a quest!");
  }
}

```
业务类：
```

public class BraveKnight implements Knight {
  private Quest quest;
  
  public BraveKnight(Quest quest) {
    this.quest = quest;       //<co id="co_injectedQuest"/>
  }
  
  public void embarkOnQuest() throws QuestException {
   /**想要实现的效果。
   *minstrel.singBeforeQuest();
   *quest.embark;
   *minstrel.singAfterQuest();
   */
    quest.embark();
  }
}


```
声明切面：
```

<bean id="knight" class="com.springinaction.knights.BraveKnight">
    <constructor-arg ref="quest" />       
  </bean>

  <bean id="quest"
        class="com.springinaction.knights.SlayDragonQuest" />
    
  <bean id="minstrel" 
     class="com.springinaction.knights.Minstrel" /> <!--<co id="co_minstrel_bean"/>-->
    
	切面定义
  <aop:config>
    <aop:aspect ref="minstrel"> 切面
	
      <aop:pointcut id="embark"  切点
          expression="execution(* *.embarkOnQuest(..))" /> 切点匹配要切入的地方，任何带有名为embarkOnQuest方法的类。

      <aop:before pointcut-ref="embark" 前置通知
                  method="singBeforeQuest"/>    <!--<co id="co_minstrel_before_advice"/>-->

      <aop:after pointcut-ref="embark" 后置通知
                 method="singAfterQuest"/>     <!--<co id="co_minstrel_after_advice"/>-->

    </aop:aspect>
  </aop:config>  

```
###  使用模板消除样式代码： 
![](/data/dokuwiki/spring/pasted/20150805-030504.png)
###  Spring容器介绍 
分为两类：
BeanFactory接口：较为低级
ApplicationContext接口：面向应用
应用上下文分类：
ClassPathXmlApplicationContext
FileSystemXmlApplicationContext
XmlWebApplicationContext
![](/data/dokuwiki/spring/pasted/20150805-030758.png)![](/data/dokuwiki/spring/pasted/20150805-030805.png)
###  Bean的生命周期 
![](/data/dokuwiki/spring/pasted/20150805-030854.png)
###  Spring模块 
![](/data/dokuwiki/spring/pasted/20150805-030934.png)
##  装配Bean 
主要内容：
  * 声明Bean
  * 构造器注入和Setter方法注入
  * 装配Bean
  * 控制Bean的创建和销毁

创建应用对象之间协作关系的行为通常被称为装配(wiring).这也是依赖注入的本质。
XML配置：
![](/data/dokuwiki/spring/pasted/20150805-031330.png)
` 注意：被注入的Bean必须包含无参构造器 `，或者采用工厂方法返回实例（看后面)
**通过构造器注入**
```

  <bean id="poeticDuke"
      class="com.springinaction.springidol.PoeticJuggler">
    <constructor-arg value="15" />
    <constructor-arg ref="sonnet29" />
  </bean>

```
**通过工厂方法创建Bean**
```

  <bean id="theStage"
      class="com.springinaction.springidol.Stage" factory-method="getInstance">
  </bean>

```
```

public class Stage {
  private Stage() { //私有化构造器
  }

  private static class StageSingletonHolder { //采用SingletonHolder模式防止出现多线程安全问题。
    static Stage instance = new Stage(); //<co id="co_lazyLoad"/>
  }

  public static Stage getInstance() {  //通过工厂方法返回实例。
    return StageSingletonHolder.instance; //<co id="co_returnInstance"/>
  }
}

```
###  Bean的作用域 
` **所有的Spring Bean默认都是单例的。声明一个bean，它只会实例化一次。总是返回同一个实例。（限于Spring上下文范围内，并非真正单例模式**） `那么就涉及到bean作用域的问题。因为默认情况不会总是满足我们的需求。

![](/data/dokuwiki/spring/pasted/20150805-032836.png)
![](/data/dokuwiki/spring/pasted/20150805-032954.png)
###  初始化和销毁Bean 
![](/data/dokuwiki/spring/pasted/20150805-033341.png)
![](/data/dokuwiki/spring/pasted/20150805-033401.png)
```

<bean id="auditorium"
      class="..."
      init-method="turnOnLights"
      destroy-method="turnOffLights"/>

```
![](/data/dokuwiki/spring/pasted/20150805-033432.png)
###  注入Bean属性 
```

  <bean id="kenny2"
      class="com.springinaction.springidol.Instrumentalist">
    <property name="song" value="Jingle Bells" />
    <property name="instrument" ref="saxophone" />
  </bean>

```
####  使用命名空间p简化属性装配 
xmlns:p="http://www.springframework.org/schema/p"
```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:p="http://www.springframework.org/schema/p"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
<bean id="kenny" class="com.springinaction.springidol.Instrumentalist"
    p:song = "Jingle Bells"
    p:instrument-ref = "saxophone" />

```
###  装配集合 
![](/data/dokuwiki/spring/pasted/20150805-034907.png)
```

  private Map<String, Instrument> instruments;

  public void setInstruments(Collection<Instrument> instruments) {
    this.instruments = instruments; //<co id="co_injectInstrumentMap"/>
  }

```
![](/data/dokuwiki/spring/pasted/20150805-035332.png)
```

  private Map<String, Instrument> instruments;

  public void setInstruments(Map<String, Instrument> instruments) {
    this.instruments = instruments; //<co id="co_injectInstrumentMap"/>
  }

```
![](/data/dokuwiki/spring/pasted/20150805-035442.png)
![](/data/dokuwiki/spring/pasted/20150805-035456.png)
```

  private Properties instruments;

  public void setInstruments(Properties instruments) {
    this.instruments = instruments; //<co id="co_injectInstrumentMap"/>
  }

```
![](/data/dokuwiki/spring/pasted/20150805-035547.png)
###  装配空值 
![](/data/dokuwiki/spring/pasted/20150805-035614.png)
###  使用表达式SpEL装配 
![](/data/dokuwiki/spring/pasted/20150805-035720.png)
![](/data/dokuwiki/spring/pasted/20150805-035740.png)
![](/data/dokuwiki/spring/pasted/20150805-035749.png)
引用Bean:
![](/data/dokuwiki/spring/pasted/20150805-035813.png)
![](/data/dokuwiki/spring/pasted/20150805-035825.png)
![](/data/dokuwiki/spring/pasted/20150805-035836.png)
![](/data/dokuwiki/spring/pasted/20150805-035855.png)
操作类的静态方法或属性:
![](/data/dokuwiki/spring/pasted/20150805-035929.png)
SpEL的正则表达式：
![](/data/dokuwiki/spring/pasted/20150805-040014.png)
![](/data/dokuwiki/spring/pasted/20150805-040021.png)
在SpEL中筛选集合：
首先定义一个util：list:
xmlns:util="http://www.springframework.org/schema/util"
```

<util:list id="cities">
  <bean class="com.habuma.spel.cities.City" 
     p:name="Chicago" p:state="IL" p:population="2853114"/>
  <bean class="com.habuma.spel.cities.City" 
     p:name="Atlanta" p:state="GA" p:population="537958"/>
  <bean class="com.habuma.spel.cities.City" 
     p:name="Dallas" p:state="TX" p:population="1279910"/>
  <bean class="com.habuma.spel.cities.City" 
     p:name="Houston" p:state="TX" p:population="2242193"/>
</util:list>

```
访问集合成员:#{songList[0]}
![](/data/dokuwiki/spring/pasted/20150805-040341.png)
![](/data/dokuwiki/spring/pasted/20150805-040350.png)
查询集合成员：查询运算符:`  .?[] ` ` .^[] ` ` .$[] `
![](/data/dokuwiki/spring/pasted/20150805-040453.png)
![](/data/dokuwiki/spring/pasted/20150805-040529.png)
![](/data/dokuwiki/spring/pasted/20150805-040537.png)
投影集合：` .![] `
![](/data/dokuwiki/spring/pasted/20150805-040644.png)
![](/data/dokuwiki/spring/pasted/20150805-040651.png)
![](/data/dokuwiki/spring/pasted/20150805-040659.png)
![](/data/dokuwiki/spring/pasted/20150805-040708.png)

##  最小化XML配置 
主要内容：
  * Bean的自动装配autowiring
  * Bean的自动检测:autodiscovery
  * 面向注解的Bean装配
![](/data/dokuwiki/spring/pasted/20150805-040901.png)
###  自动装配类型： 

![](/data/dokuwiki/spring/pasted/20150805-040934.png)
**byName:**
![](/data/dokuwiki/spring/pasted/20150805-041103.png)
**byType:**
![](/data/dokuwiki/spring/pasted/20150805-041042.png)
![](/data/dokuwiki/spring/pasted/20150805-041300.png)
![](/data/dokuwiki/spring/pasted/20150805-041324.png)
**constructor:**
![](/data/dokuwiki/spring/pasted/20150805-041354.png)
![](/data/dokuwiki/spring/pasted/20150805-041407.png)
**autodetect:**
![](/data/dokuwiki/spring/pasted/20150805-041419.png)
![](/data/dokuwiki/spring/pasted/20150805-041432.png)
**配置默认自动装配**：
在beans根元素下添加default-autowire=""属性
**混合使用自动装配和显式装配。**
![](/data/dokuwiki/spring/pasted/20150805-041553.png)
###  使用注解装配： 
Spring容器默认禁用注解装配。开启：
xmlns:context="http://www.springframework.org/schema/context"
` <context:annotation-config /> `
![](/data/dokuwiki/spring/pasted/20150805-041805.png)
####  使用@Autowired 

![](/data/dokuwiki/spring/pasted/20150805-041836.png)
![](/data/dokuwiki/spring/pasted/20150805-041946.png)
![](/data/dokuwiki/spring/pasted/20150805-042001.png)
![](/data/dokuwiki/spring/pasted/20150805-042022.png)
**如何让@Autowired远离失败**
可选依赖
![](/data/dokuwiki/spring/pasted/20150805-042122.png)
限定依赖
![](/data/dokuwiki/spring/pasted/20150805-042205.png)
创建自定义的Qualifier:
![](/data/dokuwiki/spring/pasted/20150805-042309.png)
![](/data/dokuwiki/spring/pasted/20150805-042333.png)
####  使用JSR330标准的注解@Inject 
与@Autowired不同之处：
![](/data/dokuwiki/spring/pasted/20150805-042450.png)
![](/data/dokuwiki/spring/pasted/20150805-042512.png)
![](/data/dokuwiki/spring/pasted/20150805-042547.png)
![](/data/dokuwiki/spring/pasted/20150805-042600.png)
####  在注解注入中使用SpEL 
![](/data/dokuwiki/spring/pasted/20150805-042634.png)
###  自动检测Bean 
![](/data/dokuwiki/spring/pasted/20150805-042714.png)
xmlns:context="http://www.springframework.org/schema/context"
```

  <context:component-scan 
      base-package="com.springinaction.springidol">
  </context:component-scan>

```
![](/data/dokuwiki/spring/pasted/20150805-042822.png)
![](/data/dokuwiki/spring/pasted/20150805-042839.png)
![](/data/dokuwiki/spring/pasted/20150805-042849.png)
**过滤组件扫描**
![](/data/dokuwiki/spring/pasted/20150805-042923.png)
![](/data/dokuwiki/spring/pasted/20150805-043005.png)
![](/data/dokuwiki/spring/pasted/20150805-043014.png)
##  面向切面AOP的Spring 
主要内容：
AOP的基本原理；
为POJO创建切面
使用注解创建切面
**横切关注点可以被模块化为特殊的类，这些类被称为切面。**
![](/data/dokuwiki/spring/pasted/20150805-080435.png)
![](/data/dokuwiki/spring/pasted/20150805-080506.png)
###  AOP术语 
![](/data/dokuwiki/spring/pasted/20150805-080620.png)
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

` Spring目前仅支持使用方法调用作为连接点（join point） `（在Spring bean上通知方法的执行）。 虽然可以在不影响到Spring AOP核心API的情况下加入对成员变量拦截器支持，但Spring并没有实现成员变量拦截器。 如果你需要把对成员变量的访问和更新也作为通知的连接点，可以考虑其它语法的Java语言，例如AspectJ。
![](/data/dokuwiki/spring/pasted/20150805-081225.png)

` Spring只支持方法连接点 `

**注意：Spring的切点表达式语言是AspectJ切点指示器的一个子集。**
Spring AOP支持的AspectJ切入点指示符如下：
	 execution：用于匹配方法执行的连接点；
         within：用于匹配指定类型内的方法执行；
         this：用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；
         target：用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配；
         args：用于匹配当前执行的方法传入的参数为指定类型的执行方法；
         @within：用于匹配所以持有指定注解类型内的方法；
         @target：用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；
         @args：用于匹配当前执行的方法传入的参数持有指定注解的执行；
         @annotation：用于匹配当前执行方法持有指定注解的方法；
         bean：Spring AOP扩展的，AspectJ没有对于指示符，用于匹配特定名称的Bean对象的执行方法；
         reference pointcut：表示引用其他命名切入点，只有@ApectJ风格支持，Schema风格不支持。
AspectJ切入点支持的切入点指示符还有： call、get、set、preinitialization、staticinitialization、initialization、handler、adviceexecution、withincode、cflow、cflowbelow、if、@this、@withincode；但Spring AOP目前不支持这些指示符，使用这些指示符将抛出IllegalArgumentException异常。这些指示符Spring AOP可能会在以后进行扩展。
**` 注意：只有execution指示器是唯一的执行匹配，在此基础上，其他的指示器都是用于限制匹配的。 `**
###  编写切点 
可以参考:http://jinnianshilongnian.iteye.com/blog/1415606
![](/data/dokuwiki/spring/pasted/20150805-081810.png)
**使用Spring bean()指示器**
![](/data/dokuwiki/spring/pasted/20150805-082051.png)
![](/data/dokuwiki/spring/pasted/20150805-082106.png)
![](/data/dokuwiki/spring/pasted/20150805-082113.png)
###  Spring AOP类型匹配语法 
类型匹配语法
首先让我们来了解下AspectJ类型匹配的通配符：
` * `：匹配任何数量字符；
` .. `：匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。
` + `：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。
例如：
```

java.lang.String    匹配String类型；  
java.*.String       匹配java包下的任何“一级子包”下的String类型；  
如匹配java.lang.String，但不匹配java.lang.ss.String  
java..*            匹配java包及任何子包下的任何类型;  
                  如匹配java.lang.String、java.lang.annotation.Annotation  
java.lang.*ing      匹配任何java.lang包下的以ing结尾的类型；  
java.lang.Number+  匹配java.lang包下的任何Number的子类型；  
                   如匹配java.lang.Integer，也匹配java.math.BigInteger 

```
###  在XML中声明切面 
![](/data/dokuwiki/spring/pasted/20150805-082142.png)
**声明前置通知后置通知**
xmlns:aop="http://www.springframework.org/schema/aop"
```

<aop:config>
  <aop:aspect ref="audience"><!--<co id="co_refAudienceBean"/>-->

    <aop:before pointcut=
         "execution(* com.springinaction.springidol.Performer.perform(..))"
      method="takeSeats" /> <!--<co id="co_beforePointcut"/>-->
       
    <aop:before pointcut=
         "execution(* com.springinaction.springidol.Performer.perform(..))"
      method="turnOffCellPhones" /> <!--<co id="co_beforePointcut2"/>-->
       
    <aop:after-returning pointcut=
         "execution(* com.springinaction.springidol.Performer.perform(..))" 
      method="applaud" /> <!--<co id="co_afterPointcut"/>-->
       
    <aop:after-throwing pointcut=
         "execution(* com.springinaction.springidol.Performer.perform(..))" 
      method="demandRefund" /> <!--<co id="co_afterThrowingPointcut"/>-->
       
 </aop:aspect>
</aop:config>

```
消除重复代码：` aop:pointcut ` ` pointcut-ref `
```

  <aop:config>
    <aop:aspect ref="magician">
      <aop:pointcut id="thinking" 
        expression="execution(* 
        com.springinaction.springidol.Thinker.thinkOfSomething(String)) 
             and args(thoughts)" />          
      <aop:before 
          pointcut-ref="thinking"
          method="interceptThoughts" 
          arg-names="thoughts" />
    </aop:aspect>
  </aop:config>

```
####  声明环绕通知： 
```

  <aop:config>
    <aop:aspect ref="audience">
      <aop:pointcut id="performance" expression=
          "execution(* com.springinaction.springidol.Performer.perform(..))" 
          />

      <aop:around 
          pointcut-ref="performance" 
          method="watchPerformance" />
    </aop:aspect>
  </aop:config>

```
![](/data/dokuwiki/spring/pasted/20150805-082536.png)
####  为通知传递参数 
```

  <aop:config>
    <aop:aspect ref="magician">
      <aop:pointcut id="thinking" 
        expression="execution(* 
        com.springinaction.springidol.Thinker.thinkOfSomething(String)) 
             and args(thoughts)" />          
      <aop:before 
          pointcut-ref="thinking"
          method="interceptThoughts" 
          arg-names="thoughts" />
    </aop:aspect>
  </aop:config>

```
![](/data/dokuwiki/spring/pasted/20150805-082825.png)
###  通过切面引入新功能 
![](/data/dokuwiki/spring/pasted/20150805-082907.png)
![](/data/dokuwiki/spring/pasted/20150805-082921.png)
```

<aop:aspect>
  <aop:declare-parents 
    types-matching="com.springinaction.springidol.Performer+" 
    implement-interface="com.springinaction.springidol.Contestant"
    default-impl="com.springinaction.springidol.GraciousContestant"
    />
</aop:aspect>

```
![](/data/dokuwiki/spring/pasted/20150805-083058.png)
###  注解切面 
` **注意：编写了注解形式的切面之后，我们需要把切面类声明为Spring应用上下文中的一个Bean.** `
先声明spring自动可以识别@Aspect注解及相关注解然后自动代理:
`  <aop:aspectj-autoproxy /> `
` @Aspect `注解.
```

@Aspect
public class Audience {
  @Pointcut(
        "execution(* com.springinaction.springidol.Performer.perform(..))")
  public void performance() { //<co id="co_definePointcut"/>
  }

  @Before("performance()")
  public void takeSeats() { //<co id="co_takeSeatsBefore"/>
    System.out.println("The audience is taking their seats.");
  }

  @Before("performance()")
  public void turnOffCellPhones() { //<co id="co_turnOffCellPhonesBefore"/>
    System.out.println("The audience is turning off their cellphones");
  }

  @AfterReturning("performance()")
  public void applaud() { //<co id="co_applaudAfter"/>
    System.out.println("CLAP CLAP CLAP CLAP CLAP");
  }

  @AfterThrowing("performance()")
  public void demandRefund() { //<co id="co_demandRefundAfterException"/>
    System.out.println("Boo! We want our money back!");
  }
}

```
**@Pointcut注解用于定义一个可以在@Aspect切面内可用的可重用切点。
切点的名称来源于所引用的方法名称，此处切点名为performance()，performance()方法的实际内容并不重要，在这里只是作为一个标识。**

####  注解环绕通知@Around 
```

 @Around("performance()")
  public void watchPerformance(ProceedingJoinPoint joinpoint) {
    try {
      System.out.println("The audience is taking their seats.");
      System.out.println("The audience is turning off their cellphones");

      long start = System.currentTimeMillis();
      joinpoint.proceed();
      long end = System.currentTimeMillis();

      System.out.println("CLAP CLAP CLAP CLAP CLAP");

      System.out.println("The performance took " + (end - start)
          + " milliseconds.");
    } catch (Throwable t) {
      System.out.println("Boo! We want our money back!");
    }
  }

```
####  传递参数给所标注的通知： 
```

 private String thoughts;

  @Pointcut("execution(* com.springinaction.springidol." //<co id="co_parameterizedPointcut"/>
      + "Thinker.thinkOfSomething(String)) && args(thoughts)")
  public void thinking(String thoughts) {
  }

  @Before("thinking(thoughts)") //<co id="co_passInParameters"/>
  public void interceptThoughts(String thoughts) {
    System.out.println("Intercepting volunteer's thoughts : " + thoughts);
    this.thoughts = thoughts;
  }

```
####  标注引入 
```

@Aspect
public class ContestantIntroducer {

  @DeclareParents( //<co id="co_declareParents"/>
      value = "com.springinaction.springidol.Performer+", 
      defaultImpl = GraciousContestant.class)
  public static Contestant contestant;
}

```
![](/data/dokuwiki/spring/pasted/20150805-084131.png)
![](/data/dokuwiki/spring/pasted/20150805-084336.png)