title: weblogic_xml配置文件 

#  weblogic_xml配置文件 
位置：
1、非web项目，使用的配置是：
META-INF/weblogic-application.xml
2、web项目,是WEB-INF/weblogic.xml文件

weblogic.xml用处：
defines how named resources in the web.xml file are mapped to WebLogic Server resources.
Examples of weblogic.xml attributes include 
  * HTTP session parameters, 
  * HTTP cookie parameters, 
  * JSP parameters, 
  * resource references, 
  * security role assignments, 
  * and container attributes.

weblogic.xml示例：
```

<?xml version="1.0" encoding="UTF-8"?>  
<wls:weblogic-web-app xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/ejb-jar_3_0.xsd http://xmlns.oracle.com/weblogic/weblogic-web-app http://xmlns.oracle.com/weblogic/weblogic-web-app/1.4/weblogic-web-app.xsd">  
	<!-- 为了解决slf4j和log4j的兼容问题，不然slf4j打印的日志就不出来了 -->
	<wls:container-descriptor>
		<!-- wls:prefer-web-inf-classes 和  prefer-application-packages 只能二选一  -->
		<wls:prefer-application-packages>
			<wls:package-name>org.slf4j</wls:package-name>
			<wls:package-name>antlr.*</wls:package-name>  
			<wls:package-name>com.bea.xbean.*</wls:package-name>  
			<wls:package-name>com.bea.xml.*</wls:package-name>			
		</wls:prefer-application-packages>
		<!-- <wls:prefer-web-inf-classes>true</wls:prefer-web-inf-classes> -->
	</wls:container-descriptor>
	<!-- 设置jsp检查间隔时间，生产模式暂时使用 -->
	<wls:jsp-descriptor>
		<wls:page-check-seconds>1</wls:page-check-seconds>
	</wls:jsp-descriptor>
	<wls:container-descriptor>
		<wls:servlet-reload-check-secs>1</wls:servlet-reload-check-secs>
		<wls:resource-reload-check-secs>1</wls:resource-reload-check-secs>
	</wls:container-descriptor>
<!--
	<wls:session-descriptor>
		<wls:timeout-secs>7200</wls:timeout-secs>
		<wls:cookie-name>JSESSIONID</wls:cookie-name>
		<wls:cookie-path>mhpt</wls:cookie-path>
		<wls:encode-session-id-in-query-params>true</wls:encode-session-id-in-query-params>
	</wls:session-descriptor>
	-->
	<wls:context-root>/test</wls:context-root> 
	
	<wls:charset-params>
		<wls:input-charset>
			<wls:resource-path>/*</wls:resource-path>
			<wls:java-charset-name>UTF-8</wls:java-charset-name>
		</wls:input-charset>
	</wls:charset-params>
</wls:weblogic-web-app>

```


问题解决记录：
1、出现Java.lang.NoSuchMethodError: 引用的包没有问题，代码也没有问题。但是一启动weblogic，运行程序，就出现这个异常
原因分析：weblogic在启动时，会加载一些自己的jar包，需要做一些配置，才能优先加载自己项目中的jar包。可能项目中用到的jar包和weblogic预置的jar包冲突了。
解决：修改 web项目WEB-INF/weblogic.xml文件（非web项目，使用的配置是：META-INF/weblogic-application.xml）
```

<wls:container-descriptor>
  <wls:prefer-application-packages>  
        <wls:package-name>conflict-package-name</wls:package-name>  
  </wls:prefer-application-packages>  
</wls:container-descriptor>

```
如果冲突的包不好找，可以直接下面这样简单粗暴：
```

<wls:container-descriptor>  
   <wls:prefer-web-inf-classes>true</wls:prefer-web-inf-classes>  
</wls:container-descriptor> 

```
参考http://www.informit.com/articles/article.aspx?p=32085&seqNum=3
https://answers.yahoo.com/question/index?qid=20081108233649AAMb2ks
http://stackoverflow.com/questions/7990633/java-lang-nosuchmethoderror-javax-persistence-spi-persistenceunitinfo-getvalida
http://blog.csdn.net/llwan/article/details/52775927