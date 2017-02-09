title: 使用maven管理spring 

#  使用Maven管理Spring 
参考：http://blog.csdn.net/renfufei/article/details/35794985
http://www.cnblogs.com/csonezp/p/3642817.html
本教程向您展示如何通过 Maven 管理 Spring 的依赖关系.

**2. 使用Maven管理基本的Spring依赖关系** 
Spring被设计为可高度模块化的 —— 使用Spring中的一部分,不应该也不需要引用另一个不相关的部分. 例如, 使用基本的Spring Context可以不使用 Persistence或MVC相关的Spring库.
让我们从一个非常简单的Maven设置开始,这里只使用 ` spring-context ` 依赖 :
```

<properties>  
    <org.springframework.version>4.2.3.RELEASE</org.springframework.version>  
    <!-- <org.springframework.version>4.2.3.RELEASE</org.springframework.version> -->  
</properties>  
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-context</artifactId>  
    <version>${org.springframework.version}</version>   
</dependency>

```  
**spring-context 包定义了Spring注入(Injection)容器,并依赖很少的Spring包: spring-core, spring-expression, spring-aop 和 spring-beans.** 通过启用支持一些 Spring的核心技术增强了Spring容器: Spring表达式语言 (SpEL), 面向切面编程 支持以及 JavaBeans机制.
注意,我们将spring-context依赖的范围指定为 runtime scope —— 这将确保在编译时没有任何依赖Spring特定api的部分. 对于一些底层开发的情况,可以将 runtime scope 从选定的Spring依赖项中移除(Maven 默认是compile),但对于简单的项目来说,并不需要在编码时对Spring 的整个框架进行调用.
**还要注意,从Spring 3.2开始, 不需要定义CGLIB 依赖关系(现在升级到了CGLIB3.0)—— 它已经被重新打包(现在所有 net.sf.cglib 包变成了 org.springframework.cglib包)并直接集成在 spring-core 这个 JAR包中**(详情请参考 JIRA计划文档).

**3. Spring Persistence与Maven**
现在让我们看看 Spring持久化依赖项 —— 主要是 ` spring-orm ` :
```

<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-orm</artifactId>  
    <version>${org.springframework.version}</version>  
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.20.0-GA</version>
</dependency>

```  
它提供了Hibernate和JPA支持,如 HibernateTemplate 和 JpaTemplate —— 以及持久性相关的一些依赖关系: ` spring-jdbc 和 spring-tx `.
JDBC Data Access库定义了 Spring JDBC支持 以及 JdbcTemplate, 而 spring-tx 代表了非常灵活的 **事务管理的抽象**(Transaction Management Abstraction).

**4. Spring MVC与Maven**
要使用Spring Web和Servlet支持,需要在pom中添加两个依赖项, 当然,也需要上面所说的核心依赖:` spring-web spring-webmvc `
```

<!-- <dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-web</artifactId>  
    <version>${org.springframework.version}</version>  
</dependency>   -->
<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-webmvc</artifactId>  
    <version>${org.springframework.version}</version>  
</dependency>

	<dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>1.1.0.Final</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.2.Final</version>
        </dependency>

```  
spring-web 依赖包含Servlet和Portlet环境中常用的web特定工具,而 spring-webmvc 对Servlet环境提供了MVC支持.
因为 spring-webmvc 将 spring-web 作为一个依赖,**所以在使用 spring-webmvc时不需要显式地定义 spring-web.**
**4.Spring restful需要Jaskon**
```

<jackson.version>1.8.9</jackson.version>

<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-lgpl</artifactId>
  <version>${jackson.version}</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-lgpl</artifactId>
  <version>${jackson.version}</version>
</dependency>

```

**4、Spring扩展与Maven:**
如邮件发送，任务调度，外部化配置等等。
```

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context-support</artifactId>
  <version>${org.springframework.version}</version>
</dependency>
<dependency>
  <groupId>javax.mail</groupId>
  <artifactId>mail</artifactId>
  <version>1.4.7</version>
</dependency>

```
**5. Spring Security与Maven**
关于 Security Maven依赖的深入讨论请参考 [Spring Security 3.2.x与Spring 4.0.x的Maven依赖管理](http://blog.csdn.net/renfufei/article/details/35787159)

**6. Spring Test与Maven**
Spring Test框架可以通过以下依赖引入到项目中:
```

<dependency>  
    <groupId>org.springframework</groupId>  
    <artifactId>spring-test</artifactId>  
    <version>${spring.version}</version>  
    <scope>test</scope>  
</dependency>

```  
从Spring 3.2开始,Spring MVC Test项目 作为一个独立的项目在github上提供下载 ,并且已被列入 core Test框架,只需要依赖 spring-test 就够了.

**7.Spring-data-JPA**
```

	<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-jpa</artifactId>
			<version>1.6.0.RELEASE</version>
		</dependency>

```
