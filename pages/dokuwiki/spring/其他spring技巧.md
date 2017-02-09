title: 其他spring技巧 

#  Spring外部化配置、任务调度、Email、JNDI 
主要内容：
  * 外部化配置
  * 将JNDI资源装配到spring
  * 发送E-mail
  * 调度任务
  * 异步方法
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
##  外部化配置 
两种方式：**属性占位符和属性重写**
###  属性占位符 
先使用
```

<context:property-placeholder location="classpath:/db.properties"/>
或者<context:property-placeholder location="file:///etc/db.properties"/>

```
配置外部文件位置和启用属性占位符。
然后使用：
```

<bean id="dataSource" 
   class="org.springframework.jdbc.datasource.DriverManagerDataSource"
  p:driverClassName="${jdbc.driverClassName}"
  p:url="${jdbc.url}"
  p:username="${jdbc.username}"
  p:password="${jdbc.password}" />

```
```

jdbc.protocol=hsqldb:hsql
db.server=localhost
db.name=spitter
jdbc.url=jdbc:${jdbc.protocol}://${db.server}/${db.name}/${db.name}

```
属性占位符还可以用来配置@Value注解的属性。
```

@Value("${jdbc.url}")
String databaseUrl;

```
**提供默认值：**
```

<context:property-placeholder
  location="file:///etc/myconfig.properties"
  ignore-resource-not-found="true"
  ignore-unresolvable="true" 
  properties-ref="defaultConfiguration"/>
<util:properties id="defaultConfiguration">
  <prop key="jdbc.url">jdbc:hsqldb:hsql://localhost/spitter/spitter</prop>
  <prop key="jdbc.driverClassName">org.hsqldb.jdbcDriver</prop>
  <prop key="jdbc.username">spitterAdmin</prop>
  <prop key="jdbc.password">t0ps3cr3t</prop>
</util:properties>

```
###  重写属性 
```

<context:property-override   配置启用属性重写。
    location="classpath:/db.properties" />

<bean id="dataSource" 
   class="org.springframework.jdbc.datasource.DriverManagerDataSource"
  p:driverClassName="org.hsqldb.jdbcDriver"
  p:url="jdbc:hsqldb:hsql://localhost/spitter/spitter"
  p:username="spitterAdmin"
  p:password="t0ps3cr3t" />


```
![](/data/dokuwiki/spring/pasted/20150811-081747.png)
所以外部配置文件写法为：
```

dataSource.driverClassName=org.hsqldb.jdbcDriver
dataSource.url=jdbc:hsqldb:hsql://localhost/spitter/spitter
dataSource.username=spitterAdmin
dataSource.password=t0ps3cr3t

```
###  加密外部属性 
利用Jasypt加密类库与spring集成。
略。。。

##  装配JNDI对象 
![](/data/dokuwiki/spring/pasted/20150811-081958.png)
传统jndi写法：
```

InitialContext ctx = null; 
try {
  ctx = new InitialContext();

  DataSource ds = 
      (DataSource) ctx.lookup("java:comp/env/jdbc/SpitterDatasource");
} catch (NamingException ne) {
  // handle naming exception ...
} finally { 
  if(ctx != null) {
      try {
        ctx.close();
      } catch (NamingException ne) {}
  }
}

```
spring的简化配置：
```

<jee:jndi-lookup id="dataSource"
    jndi-name="/jdbc/SpitterDS"
    resource-ref="true" 
    proxy-interface="javax.sql.DataSource" />

```
它就相当于一个普通的bean可用于装配其他bean。
```

<bean id="sessionFactory" 
  class="org.springframework.orm.hibernate3.annotation.[CA]
                                             AnnotationSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
  ...
</bean>

```
![](/data/dokuwiki/spring/pasted/20150811-082405.png)
**缓存JNDI对象：**
默认为自动缓存。如果禁用缓存：
```

<jee:jndi-lookup id="dataSource"
    jndi-name="/jdbc/SpitterDS"
    resource-ref="true" 
    cache="false"
    proxy-interface="javax.sql.DataSource" />

```
**懒加载模式：**
默认为spring容器已启动就查找jndi对象。
```

<jee:jndi-lookup id="dataSource"
    jndi-name="/jdbc/SpitterDS"
    resource-ref="true" 
    lookup-on-startup="false"
    proxy-interface="javax.sql.DataSource" />

```
**配置默认datasource：**
开发环境用本地配置的datasource，生成环境用jndi中查找到地对象：
```

<bean id="devDataSource" 
   class="org.springframework.jdbc.datasource.DriverManagerDataSource"
   lazy-init="true"> 
  <property name="driverClassName"
            value="org.hsqldb.jdbcDriver" />
  <property name="url"
            value="jdbc:hsqldb:hsql://localhost/spitter/spitter" />
  <property name="username" value="sa" />
  <property name="password" value="" />
</bean>

<jee:jndi-lookup id="dataSource"
    jndi-name="/jdbc/SpitterDS"
    resource-ref="true" 
    default-ref="devDataSource" />

```
##  发送邮件 
![](/data/dokuwiki/spring/pasted/20150811-082807.png)
```

<dependency>
			<groupId>javax.mail</groupId>
			<artifactId>mail</artifactId>
			<version>1.4.7</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>

```
**配置邮件发送器：**
JavaMailSenderImpl为MailSender的实现
```

<bean id="mailSender" 
    class="org.springframework.mail.javamail.JavaMailSenderImpl"
  p:host="${mailserver.host}" 
  p:port="${mailserver.port}"
  p:username="${mailserver.username}"
  p:password="${mailserver.password}" >
<property name="defaultEncoding" value="UTF-8" />  
  <!-- SMTP服务器验证 -->  
       <property name="javaMailProperties">  
           <props>  
               <!-- 验证身份 -->  
               <prop key="mail.smtps.auth">true</prop>  
            
          </props>  
       </property> 
</bean>

```
**将邮件发送器装配到bean:**
```

@Autowired
JavaMailSender mailSender

```
###  发送简单邮件： 
` 问题：会造成中文乱码 `
![](/data/dokuwiki/spring/pasted/20150811-083209.png)
###  JavaMailSender发送邮件中文乱码解决 
MimeMessage mime=mailSender.createMimeMessage();
MimeMessageHelper msg=new MimeMessageHelper` (mime,"utf-8"); `
```

		JavaMailSender mailSender=SpringContextUtil.getBean("mailSender", JavaMailSender.class);
		String subject=subjectPrefix+getNowDateStr();
//	会导致乱码	SimpleMailMessage msg=new SimpleMailMessage();

		/**为防止linux下发送的邮件中文乱码，使用MimeMessage，并通过MimeMessageHelper明确编码*/
		MimeMessage mime=mailSender.createMimeMessage();
		MimeMessageHelper msg=new MimeMessageHelper(mime,"utf-8");
		try {
			msg.setFrom(mailFrom);
			msg.setTo(mailto);
			msg.setSubject(subject);
			msg.setText(content);
			log.debug("邮件初始化成功");
		} catch (MessagingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			log.error("邮件发送失败{}",e);
		}
		
		mailSender.send(mime);
		log.debug("邮件发送成功");
	}

```
###  发送带附件邮件 
```

MimeMessage message = mailSender.createMimeMessage();
MimeMessageHelper helper = new MimeMessageHelper(message, true,"utf-8");
String spitterName = spittle.getSpitter().getFullName();
helper.setFrom("noreply@spitter.com");
helper.setTo(to);
helper.setSubject("New spittle from " + spitterName);
helper.setText(spitterName + " says: " + spittle.getText());
FileSystemResource couponImage = 
      new FileSystemResource("/collateral/coupon.png");
helper.addAttachment("Coupon.png", couponImage);
mailSender.send(message);

```
###  发送富文本邮件 
```

helper.setText("<html><body><img src='cid:spitterLogo'>" +     cid:spitterLogo为内联图片
    "<h4>" + spittle.getSpitter().getFullName() + " says...</h4>" +
    "<i>" + spittle.getText() + "</i>" +
		"</body></html>", true);  注意setText()第二个参数要为true
添加嵌入式内联图片：
ClassPathResource image = new ClassPathResource("spitter_logo_50.png");
helper.addInline("spitterLogo", image);

```

###  创建邮件模板 

![](/data/dokuwiki/spring/pasted/20150811-084900.png)
![](/data/dokuwiki/spring/pasted/20150811-084917.png)
![](/data/dokuwiki/spring/pasted/20150811-084937.png)
##  调度与后台任务 
有两种后台任务可供选择：调度任务和异步方法：
先配置：
` <task:annotation-driven/> `这个元素将使spring自动支持调度和异步方法。这些方法分别使用` @Scheduled和@Async `注解
参考http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#scheduling
```

<task:annotation-driven executor="myExecutor" scheduler="myScheduler"/>
<task:executor id="myExecutor" pool-size="5"/>
<task:scheduler id="myScheduler" pool-size="10"/>

```
###  声明调度方法 
` 注意@Scheduler方法没有参数 `
```

@Scheduled(fixedRate=86400000)   //每个86400000ms执行一次，周期性执行.固定每86400000ms执行一次,不管上次是否结束
public void archiveOldSpittles() {
  // ...
}

@Scheduled(fixedDelay=86400000)  //上次任务结束后等86400000ms后再次执行
public void archiveOldSpittles() {
  // ...
}
@Scheduled(cron="0 0 0 * * SAT") 采用cron配置。
public void archiveOldSpittles() {
  // ...
}

```
**Cron表达式语法：**
![](/data/dokuwiki/spring/pasted/20150811-085513.png)
![](/data/dokuwiki/spring/pasted/20150811-085529.png)
当然Spring还定义了执行器的一些接口：
TaskExecutor和TaskScheduler
ThreadPoolTaskScheduler

###  声明异步方法 
```

@Async
public void addSpittle(Spittle spittle) {
    ...
}
@Async
public Future<Long> performSomeReallyHairyMath(long input) {
    // ...
    
    return new AsyncResult<Long>(result); 使用spring提供的AsyncResult包装结果。
}

```
![](/data/dokuwiki/spring/pasted/20150811-085840.png)