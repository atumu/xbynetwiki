title: spring与springmvc配置 

#  Spring与SpringMVC配置 
版本:Spring3.x, SpringMVC3.x
```

		<org.hibernate.version>4.2.21.Final</org.hibernate.version>
		<org.springframework.version>4.2.3.RELEASE</org.springframework.version>
		<jodd.version>3.5.2</jodd.version>
	<!-- junit -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<!-- JSTL -->
		<dependency>
			<groupId>org.apache.taglibs</groupId>
			<artifactId>taglibs-standard-spec</artifactId>
			<version>1.2.5</version>
		</dependency>
		<dependency>
			<groupId>org.apache.taglibs</groupId>
			<artifactId>taglibs-standard-impl</artifactId>
			<version>1.2.5</version>
		</dependency>
		<!--commons-fileupload处理文件上传 -->
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
		</dependency>
		<!--fastJson替代默认的Jackson处理json -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.7</version>
		</dependency>
		<!-- apache tiles3.x处理布局-->
  		<dependency>
			<groupId>org.apache.tiles</groupId>
			<artifactId>tiles-extras</artifactId>
			<version>3.0.5</version>
			<exclusions>
				<exclusion>
					<artifactId>javassist</artifactId>
					<groupId>jboss</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- 任务调度-->
		<dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz</artifactId>
			<version>2.2.1</version>
		</dependency>
		<dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz-jobs</artifactId>
			<version>2.2.1</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		
		<dependency>
 		 	<groupId>org.springframework</groupId>
  			<artifactId>spring-context-support</artifactId>
 		 	<version>${org.springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${org.springframework.version}</version>
			<scope>test</scope>
		</dependency>
	
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>
	<!-- 验证-->
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
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${org.springframework.version}</version>
                  	<scope>test</scope>
		</dependency>
	<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.20.0-GA</version>
</dependency>
	<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-jpa</artifactId>
			<version>1.6.0.RELEASE</version>
		</dependency>
	<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>

		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>17.0</version>
		</dependency>
	<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.8.2</version>
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
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.9</version>
    </dependency>
		<dependency>
    			<groupId>org.slf4j</groupId>
    			<artifactId>log4j-over-slf4j</artifactId>
    		<version>1.7.9</version>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.9</version>
		</dependency>
	<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>3.2.0</version>
		</dependency>
	<dependency>
			<groupId>javax.mail</groupId>
			<artifactId>mail</artifactId>
			<version>1.4.7</version>
		</dependency>
<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.7</version>
		</dependency>
		<!-- kaptcha验证码 -->
		<dependency>
			<groupId>com.google.code.kaptcha</groupId>
			<artifactId>kaptcha</artifactId>
			<version>2.3.2</version>
		</dependency>
<dependency>
			<groupId>Mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
		</dependency>
<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.14</version>
		</dependency>

```
**WEB-INF/web.xml**
```

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	<!-- 通过配置ContextLoaderListener与全局的contextConfigLocation参数用于启用一个全局的Spring应用上下文，
	作为所有的DispatcherServlet应用上下文的父亲，可以共享这个父亲的所有bean与配置 -->
  	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
         classpath:spring-hibernate.xml
        </param-value>
      </context-param>

  	<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
  <!-- 该servlet应用上下文的父亲为前面的全局应用上下文，它可以共享全局应用上下文里的配置与bean等-->
	<servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
             	<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-servlet.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>

	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern> <!-- 注意不要配置为/*,不然JSP机制不能直接处理JSP请求等，这不是我们期望的结果-->
	</servlet-mapping>
  
  	<!--静态资源配置到default servlet这比使用SpringMVC的<mvc:resources mapping="/static/**" location="/static/" />要有效，要更好
	更具体的URL模式总是会覆盖/，所以静态资源URL映射可以覆盖前面的配置不走DispatcherServlet，而走由Web服务器容器初始化的default servlet。
	-->
     <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/resource/*</url-pattern>
        <url-pattern>/static/*</url-pattern>
    </servlet-mapping>
	<!-- 解决乱码问题-->
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
          	<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
  
  <!--  用于普通类中获取session和request -->   
 <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
 </listener>
 
<listener>
    <listener-class>org.apache.tiles.extras.complete.CompleteAutoloadTilesListener</listener-class>
</listener>
<session-config>
  		<session-timeout>30</session-timeout>
</session-config>
  
    <error-page>
  	<error-code>404</error-code>
  	<location>/404.jsp</location>
  </error-page>
  <error-page>  
    <error-code>500</error-code>  
    <location>/500.jsp</location>  
</error-page>
  <error-page>  
    <error-code>401</error-code>  
    <location>/401.jsp</location>  
</error-page>  
</web-app>

```
**WEB-INF/springmvc-servlet.xml**
```

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:p="http://www.springframework.org/schema/p"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:jpa="http://www.springframework.org/schema/data/jpa"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xmlns:tx="http://www.springframework.org/schema/tx"
xmlns:util="http://www.springframework.org/schema/util"
 xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd
http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.2.xsd
http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.2.xsd">
	<context:component-scan base-package="net.xby1993.spring" />
  
  		<!-- 引入配置文件，外部化配置 -->
	<context:property-placeholder location="classpath:common.properties,classpath:datasource.properties"/>
  	<aop:aspectj-autoproxy   proxy-target-class="true"/><!-- 支持注解AOP-->
	<mvc:annotation-driven>
		<mvc:message-converters register-defaults="true">
			<!--<bean class="org.springframework.http.converter.StringHttpMessageConverter">
				<property name="supportedMediaTypes">
					<list>
						<value>text/html;charset=UTF-8</value>
						<value>application/json;charset=UTF-8</value>
					</list>
				</property>
			</bean> -->
			<!-- 配置Fastjson支持 -->
			<bean
				class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
				<!-- 避免IE执行AJAX时,返回JSON出现下载文件 -->
				<property name="supportedMediaTypes">
					<list>
						<!-- 这里顺序不能反，一定先写text/html,不然ie下出现下载提示 -->
						<value>text/html;charset=UTF-8</value>
						<value>application/json;charset=UTF-8</value>
					</list>
				</property>
                          <property name="features">
					<list>
						<value>WriteDateUseDateFormat</value>
						<value>WriteMapNullValue</value> <!-- 当为null值也写入，默认忽略 -->
						<value>QuoteFieldNames</value> 
						
					</list>
				</property>
			</bean>
		</mvc:message-converters>
	</mvc:annotation-driven>
	<!--静态资源配置 -->
	<mvc:resources mapping="/static/**" location="/static/" />



	<!-- 配置MultipartResolver 用于文件上传 使用spring的CommosMultipartResolver -->
	<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver"
		p:defaultEncoding="UTF-8" p:maxUploadSize="5400000000"
		p:uploadTempDir="fileUpload/temp">
	</bean>
  
  <!-- Tiles3配置-->
 <bean class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
     <property name="definitions">
       <list>
         <value>/WEB-INF/views/**/views.xml</value> <!-- Ant风格路径-->
       </list>
     </property>
   </bean>
     <!-- Tiles视图解析器--> 
<bean class="org.springframework.web.servlet.view.tiles3.TilesViewResolver">
<property name="order" value="1"/>
</bean> 
  
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
   <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
        <property name="order" value="10"/>
</bean>
 

```