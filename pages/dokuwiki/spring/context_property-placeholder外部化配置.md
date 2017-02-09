title: context_property-placeholder外部化配置 

##  context property-placeholder外部化配置 
1.有些参数在某些阶段中是常量
比如 ：a、在开发阶段我们连接数据库时的连接url，username，password，driverClass等 
b、分布式应用中client端访问server端所用的server地址，port，service等  
c、配置文件的位置

2.而这些参数在不同阶段之间又往往需要改变
比如：在项目开发阶段和交付阶段数据库的连接信息往往是不同的，分布式应用也是同样的情况。
期望：能不能有一种解决方案可以方便我们在一个阶段内不需要频繁书写一个参数的值，而在不同阶段间又可以方便的切换参数配置信息
解决：spring3中提供了一种简便的方式就是` context:property-placeholder `元素
只需要在spring的配置文件里添加一句：
	<!-- 引入配置文件 -->
	<context:property-placeholder location="classpath:common.properties,classpath:datasource.properties"/>
即可，这里location值为参数配置文件的位置，参数配置文件通常放在src目录下，而参数配置文件的格式跟java通用的参数配置文件相同，即键值对的形式，例如：
```

#jdbc配置
test.jdbc.driverClassName=com.mysql.jdbc.Driver
test.jdbc.url=jdbc:mysql://localhost:3306/test
test.jdbc.username=root
test.jdbc.password=root

```
应用：
1.这样一来就可以为spring配置的bean的属性设置值了，比如spring有一个jdbc数据源的类DriverManagerDataSource
在配置文件里这么定义bean：
```

<bean id="testDataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${test.jdbc.driverClassName}"/>
    <property name="url" value="${test.jdbc.url}"/>
    <property name="username" value="${test.jdbc.username}"/>
    <property name="password" value="${test.jdbc.password}"/>
</bean>

```
2.甚至可以将${ }这种形式的变量用在spring提供的注解当中，为注解的属性提供值

##  存在的问题 
spring容器只允许配置一个<context:property-placeholder/>标签，当在多个模块分别配置时就会出现问题。
如A模块和B模块都分别拥有自己的Spring XML配置，并分别<context:property-placeholder/拥有自己的配置文件：
单独运行A模块，或单独运行B模块都是正常的，但将A和B两个模块集成后运行，Spring容器就启动不了了：
` Could not resolve placeholder 'moduleb.jdbc.driverClassName' in string value "${moduleb.jdbc.driverClassName}" `
原因：Spring容器采用反射扫描的发现机制，在探测到Spring容器中有一个org.springframework.beans.factory.config.PropertyPlaceholderConfigurer的Bean就**会停止对剩余PropertyPlaceholderConfigurer的扫描**
spring容器中最多只能定义一个context:property-placeholder，不然就出现那种个错误，那如何来解决上面的问题呢？
A和B模块去掉
<context:property-placeholder />
然后从先写个：
```

<?xml version="1.0" encoding="UTF-8" ?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:p="http://www.springframework.org/schema/p"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd  
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd">  
   <context:property-placeholder location="classpath*:conf/conf*.properties"/>  
   <import resource="a.xml"/>  
   <import resource="b.xml"/>  
</beans>

```
参考：http://blog.sina.com.cn/s/blog_4550f3ca0100ubmt.html
http://blog.csdn.net/sunhuwh/article/details/15813103