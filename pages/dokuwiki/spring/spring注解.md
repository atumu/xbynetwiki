title: spring注解 

#  Spring注解与标签 
参考：http://blog.csdn.net/kobejayandy/article/details/12690709
` <context:component-scan> ` 扫描指定的包中的类上的注解，常用的注解有：
@Controller 声明Action组件
@Service    声明Service组件    @Service("myMovieLister") 
@Repository 声明Dao组件
@Component   泛指组件, 当不好归类时. 
@RequestMapping("/menu")  请求映射
@Resource  用于注入，( j2ee提供的 ) 默认按名称装配，@Resource(name="beanName") 
@Autowired 用于注入，(srping提供的) 默认按类型装配 
@Transactional( rollbackFor={Exception.class}) 事务管理
@ResponseBody
@Scope   设定bean的作用域，如"prototype"多实列
以及其他。

` <mvc:annotation-driven /> ` 是一种简写形式，完全可以手动配置替代这种简写形式，简写形式可以让初学都快速应用默认配置方案。<mvc:annotation-driven /> 会自动注册` DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter ` 两个bean,是spring MVC为@Controllers分发请求所必须的。
并提供了：数据绑定支持，@NumberFormatannotation支持，@DateTimeFormat支持，@Valid支持，读写XML的支持（JAXB），读写JSON的支持（Jackson）。

` <mvc:interceptors/> ` 是一种简写形式。我们可以配置多个HandlerMapping。<mvc:interceptors/>会为每一个HandlerMapping，注入一个拦截器。其实我们也可以手动配置为每个HandlerMapping注入一个拦截器。
<mvc:default-servlet-handler/> 使用默认的Servlet来响应静态文件。
```

<mvc:resources mapping="/images/**" location="/images/" cache-period="31556926"/> 匹配URL  /images/**

```  的URL被当做静态资源，由Spring读出到内存中再响应http。
