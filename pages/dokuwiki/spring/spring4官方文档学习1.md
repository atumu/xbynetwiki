title: spring4官方文档学习1 

#  Spring4官方文档学习1介绍 
##  模块组成介绍 
Springframework大约由20个模块组成。分为几个大的块。如下图所示:
![](/data/dokuwiki/spring/pasted/20160328-162820.png)
![](/data/dokuwiki/spring/pasted/20160328-163126.png)
![](/data/dokuwiki/spring/pasted/20160328-163143.png)
```

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.2.5.RELEASE</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>

```
###  Core Container 
组成：spring-core, spring-beans, spring-context, spring-context-support, and spring-expression (Spring Expression Language) modules.
  * Core和Beans提供了框架的基础，包括IOC和DI特性。其中的` BeanFactory `是工厂模式的一个复杂实现，它允许对依赖进行管理。
  * spring-context模块基于Core和Beans.可以像使用JNDI注册表一样来查找依赖对象。同时还提供了国际化 (using, for example, resource bundles)，事件传播，资源加载、JMX，远程访问等。其中` ApplicationContext `是其中的核心接口。
  * spring-context-support提供了通用的第三方库的集成。比如caching (EhCache, Guava, JCache), mailing (JavaMail), scheduling (CommonJ, Quartz) and template engines (FreeMarker, JasperReports, Velocity).
  * spring-expression提供了SpEL表达式语言用于在运行时查询和操作对象图。它是JSP EL的一个扩展。

###  AOP和Instrumentation 
The ` spring-aop ` module provides an AOP Alliance-compliant aspect-oriented programming implementation allowing you to define, for example, method interceptors and pointcuts to cleanly decouple code that implements functionality that should be separated. 
The separate ` spring-aspects ` module provides integration with AspectJ.
The ` spring-instrument ` module** provides class instrumentation support and classloader implementations** to be used in certain application servers. The spring-instrument-tomcat module contains Spring’s instrumentation agent for Tomcat.

###  Messaging 
Spring4开始提供spring-messaging模块
###  Data Access/Integration 
The Data Access/Integration layer consists of the JDBC, ORM, OXM, JMS, and Transaction modules.
The ` spring-jdbc ` module provides a JDBC-abstraction layer that removes the need to do tedious JDBC coding and parsing of database-vendor specific error codes.
The ` spring-tx ` module supports programmatic and declarative transaction management for classes that implement special interfaces and for all your POJOs (Plain Old Java Objects).

The ` spring-orm ` module provides integration layers for popular object-relational mapping APIs, including **JPA, JDO, and Hibernate**. Using the spring-orm module you can use all of these O/R-mapping frameworks in combination with all of the other features Spring offers, such as the simple declarative transaction management feature mentioned previously.

The ` spring-oxm ` module provides an abstraction layer that supports Object/XML mapping implementations such as JAXB, Castor, XMLBeans, JiBX and XStream.

The ` spring-jms ` module (Java Messaging Service) contains features for producing and consuming messages. **Since Spring Framework 4.1, it provides integration with the spring-messaging module.**

###  Web 
The Web layer consists of the ` spring-web, spring-webmvc, spring-websocket, and spring-webmvc-portlet ` modules.
spring-web提供了面向web的一些特性,如文件上传,基于Servlet-listener的IoC容器初始化，面向web的ApplicationContext等，还包括了HttpClient和romting support。
spring-webmvc提供了MVC和REST WebServices实现。依赖于spring-web
spring-webmvc-portlet。略。
###  Test 
spring-test支持单元测试和集成测试，基于JUnit或TestNG.同时还提供了mock objects以便于隔离测试。

##  日志 
Spring-core默认使用commons-logging作为日志组件。如果我们想集成SLF4J。那么我们需要这么做:
```

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.2.5.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
      	<!--这个是最关键的 -->
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.9</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.9</version>
    </dependency>
	<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-classic</artifactId>
			<version>1.1.3</version>
		</dependency>
		<dependency>
			<groupId>ch.qos.logback</groupId>
			<artifactId>logback-core</artifactId>
			<version>1.1.3</version>
		</dependency>

```
