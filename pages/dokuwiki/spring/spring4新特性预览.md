title: spring4新特性预览 

#  Spring4新特性预览 
参考：http://jinnianshilongnian.iteye.com/blog/2102278
http://ningandjiao.iteye.com/blog/1993481
从目前来看Spring 4.1并没有特别吸引眼球的地方，主要还是增强和一些依赖的版本升级。主要改进如下：
1、核心部分基本上无变化，提供了DirectFieldAccessor用于直接字段访问、yaml配置、SpEL的字节码编译化、BackOff退避算法的基本实现、Base64Utils、SmartInitializingSingleton等；
2、在任务调度和事件机制上加入了异常处理部分；
3、cache部分加入jcache的集成、类级别的@CacheConfig的支持、CacheResolver；
4、**mvc部分**提供了一些视图解析器的mvc标签实现简化配置、提供了GroovyWebApplicationContext用于Groovy web集成、提供了` Gson、protobuf的HttpMessageConverter `、静态资源处理方面添加了resolver和transformer、提供了对groovy-templates模板的支持、**JSONP的支持、对Jackson的@JsonView的支持**等；
5、提供了页面自动化测试框架Spring MVC Test HtmlUnit；
6、test部分提供了更便利的@sql标签来执行测试脚本的初始化、MockRestServiceServer对AyncRestTemplate支持、MockMvcConfigurer来全局配置MockMvc；
7、提供了对Java 8 Optional的支持（ObjectToOptionalConverter实现；可以在MVC中如@RequestParam等注解的绑定）、从Spring 4.1起Ehcache支持需要EhCache 2.5或更高版本、兼容Quartz 2.1.4、支持javax.annotation.Priority（等价于@Order）、JMS API的增强、SpringJUnit4ClassRunner需要 JUnit 4.9或更高版本支持。

具体说明下：
**JDK8** 
对JDK8的支持，应该没有公司会在目前的情况下使用JDK8做产品开发。因此，暂时可忽略。 

**Groovy Bean Definition DSL** 
即可以通过Groovy class来配置Bean，以及Bean之间的相互依赖，即Spring2时代推出的xml配置，Spring3推出的Annotation配置，Spring4为配置方式又增添了一位新成员Groovy DSL。举个栗子： 
```

import org.hibernate.SessionFactory  
import org.apache.commons.dbcp.BasicDataSource  
  
beans {  
    dataSource(BasicDataSource) {  
        driverClassName = "org.hsqldb.jdbcDriver"  
        url = "jdbc:hsqldb:mem:grailsDB"  
        username = "sa"  
        password = ""  
        settings = [mynew:"setting"]  
    }  
    sessionFactory(SessionFactory) {  
        dataSource = dataSource  
    }  
    myService(MyService) {  
        nestedBean = { AnotherBean bean ->  
            dataSource = dataSource  
        }  
    }  
}  

```
DSL配置在概念上和其他配置方式是一样的，只是提供了一种更简洁的语法，这个方式的实现得益于Grails的BeanBuilder，所有支持的DSL语法也来自于这儿：http://grails.org/doc/latest/guide/spring.html#theBeanBuilderDSLExplained， 如果你想追踪这个想法的起源，可以看看这篇文章：http://spring.io/blog/2007/11/29/spring-dynamic-language-support-and-a-groovy-dsl/ 另外，这儿有一篇非常好的文章详细描述了如何使用该特性：http://jinnianshilongnian.iteye.com/blog/1991830。总的来说， 个人觉得使用DSL的配置方式，就像Build工具界的Gradle之于Maven，它极大的灵活了Spring的配置文件，可以通过groovy脚本实现非常复杂的Bean定义和依赖关系，甚至玩出很多魔幻语法，但与之对应的是， 我们是否应该在配置文件里面玩那么复杂？不过，多一个选择总是好的，让大家有得挑。 

**核心容器功能的改进** 
这部分是应该是当前Spring用户最关注点： 
` 支持泛型依赖注入 `，即对自动注入依赖的识别扩展到了泛型的类，以前，如果有GenericInterface<A>, GenericInterface<B>两个Bean时，当想注入GenericInterface<A>依赖时， 容器是无法识别的，你需要使用@Qualifier指定具体的bean id，Spring4.0中则可以直接找到对应的Bean。这个特性对程序员的好处，请查看这篇文章：http://jinnianshilongnian.iteye.com/blog/1989330
使用meta-annoation方式定义Annotation时， 该Annotation可以访问源Annotation的部分属性，以更加方便的定制自己想要的Annotation。
` Bean依赖注入到Map和List，Array中 `， 即提供了一种方式获取到某个类型的所有Bean，当注入到Map中时， Key为Bean的名字，value为Bean实例。
如果，你对Bean在Array或List中的位置有特殊需求，Spring4.0还提供了@Order annotation和Ordered接口来定义Bean注入到Array/List中的顺序. 

扩展@Lazy annotation，除了延迟加载Bean，依赖注入也可以延迟了。
提供了@Description annotation为Bean添加描述。
增加了@Condition annotation， 使用该Annotation之后，在做依赖注入的时候，会检测是否满足某个条件，这样可以更灵活的决定注入的类，具体用法参见：http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Conditional.html，或者说这篇文章：http://jinnianshilongnian.iteye.com/blog/1989379。
基于CGLIB的代理类不在强制要求空参构造函数了，提供了一种“一旦注册，不许修改”的注入策略。

` Web开发改进 ` 
` 增加了@RestController annotation `， 就是**把@Controller和@ReponseBody打包了**，省得大家再去每个方法上加一个@ResponseBody了。
新加了AsyncRestTemplate类，可以用来构建异步调用的Restful Client， 具体用法看这儿：http://jinnianshilongnian.iteye.com/blog/1989381，或者这儿：http://docs.spring.io/spring/docs/4.0.0.RELEASE/spring-framework-reference/htmlsingle/#rest-async-resttemplate
Spring4.0基于Servlet3.0+版本开发，尤其是Spring MVC的测试框架中的Mock都是基于Servlet3.0包中的一些类的，因此使用时必须把兼容Servlet 3.0的包添加到Classpath中。
` 为Spring MVC应用增加了Timezone的支持 `，可以在` RequestContext `获取，设置TimeZone信息，` Spring还提供Datetime的转换功能 `：http://docs.spring.io/spring/docs/4.0.0.RELEASE/spring-framework-reference/htmlsingle/#mvc-timezone
` 提供了 WebSocket, SockJS, and STOMP Messaging的支持， `在一个Controller中，除了可以处理 @RequestMapping对应的Http请求，还可以` 处理对应@MessageMapping的WebSocket Client发来的Message请求 `，哪些不支持WebSocket的浏览器，Spring4.0提供了基于SockJS协议的Message处理，即你可以在浏览器基于SockJS协议模拟一个Web Socket的请求，Spring4.0也可以处理。具体的说明：http://docs.spring.io/spring/docs/4.0.0.RELEASE/spring-framework-reference/htmlsingle/#websocket
支持STOMP Message协议

测试框架改进 
几乎所有spring-test模块下的annotation（比方说：@ContextConfiguration, @WebAppConfiguration, @ContextHierarchy, @ActiveProfiles）都可以做元annoation, 这样开发者就可以更方便得定制自己的annotation，以增强代码表现力和减少多个Test之间的重复代码。
增加了一种更灵活的ActiveProfiles的决定方式，定制一个ActiveProfilesResolver并把它设置到@ActiveProfiles的resolver属性上。
添加了SocketUtils类帮忙扫描本地机器上的可用Socket端口，当需要在本地起一个mock server时这个功能非常实用。
org.springframework.mock.web包下的Mock类都与Servlet 3.0兼容了