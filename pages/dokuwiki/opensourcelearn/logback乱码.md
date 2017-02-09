title: logback乱码 

#  linux下LogBack中文乱码问题 
` <charset>UTF-8</charset> 加到对应的<encoder>元素下 `
```

<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
	
	<!-- 控制台输出 -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
			<charset>UTF-8</charset> 
		</encoder>
	</appender>
	
	<!-- 按照每天生成日志文件 -->
	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!--日志文件输出的文件名 -->
			<FileNamePattern>/opt/lampp/tomcat7/webapps/log/app_%d{yyyyMMdd}.log</FileNamePattern>
			<!--日志文件保留天数 -->
			<MaxHistory>3000</MaxHistory>
		</rollingPolicy>
		<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
			<charset>UTF-8</charset> 
		</encoder>
		<!--日志文件最大的大小 -->
		<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
			<MaxFileSize>10MB</MaxFileSize>
		</triggeringPolicy>
	</appender>

	<!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
	<logger name="org.hibernate" level="DEBUG" />
	<logger name="org.apache.http" level="warn"/>
	<!--sql log configure -->
	<logger name="java.sql" level="DEBUG" />

	<!-- 日志输出级别 -->
	<root level="DEBUG">
		<appender-ref ref="STDOUT" />
		<appender-ref ref="FILE" />
	</root>
	<!--日志异步到数据库 -->
	<!-- <appender name="DB" class="ch.qos.logback.classic.db.DBAppender"> -->
	<!-- 日志异步到数据库 -->
	<!-- <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource"> -->
	<!-- 连接池 -->
	<!-- <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource"> -->
	<!-- <driverClass>com.mysql.jdbc.Driver</driverClass> -->
	<!-- <url>jdbc:mysql://127.0.0.1:3306/databaseName</url> -->
	<!-- <user>root</user> -->
	<!-- <password>root</password> -->
	<!-- </dataSource> -->
	<!-- </connectionSource> -->
	<!-- </appender> -->

</configuration>

```