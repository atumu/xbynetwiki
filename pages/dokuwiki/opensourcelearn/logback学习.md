title: logback学习 

#  logback学习 

Logback是由log4j创始人设计的又一个开源日志组件。
logback当前分成三个模块：logback-core,logback- classic和logback-access。
  * logback-core是其它两个模块的基础模块。
  * logback-classic是log4j的一个 改良版本。此外**logback-classic完整实现SLF4J API**使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging。
  * logback-access访问模块与Servlet容器集成提供通过Http来访问日志的功能。
Logback是要与SLF4J结合起来用。
官网：http://logback.qos.ch/
SLF4J官网：http://www.slf4j.org
```

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
			<artifactId>slf4j-api</artifactId>
			<version>1.7.9</version>
		</dependency>
		<dependency>
    			<groupId>org.slf4j</groupId>
    			<artifactId>log4j-over-slf4j</artifactId>
    		<version>1.7.9</version>
		</dependency>

```
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
			<FileNamePattern>${catalina.base}/log/app_%d{yyyyMMdd}.log</FileNamePattern>
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
	<logger name="org.hibernate" level="ERROR" />

	<!--sql log configure -->
	<logger name="java.sql" level="ERROR" />
	
	<!-- druid 数据库连接池 -->
	<logger name="com.alibaba.druid" level="ERROR" />

	<!-- spring -->
	<logger name="org.springframework" level="INFO" />
	<logger name="org.springframework.web" level="DEBUG" />

	<logger name="com.css" level="ERROR" >
		<appender-ref ref="NOSQL" />
	</logger>

	<!-- 日志输出级别 -->
	<root level="INFO">
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
```

: import org.slf4j.Logger;
 2: import org.slf4j.LoggerFactory;
 3: 
 4: public class Wombat {
 5:  
 6:   final Logger logger = LoggerFactory.getLogger(Wombat.class);
 7:   Integer t;
 8:   Integer oldT;
 9:
10:   public void setTemperature(Integer temperature) {
11:    
12:     oldT = t;        
13:     t = temperature;
14:
15:     logger.debug("Temperature set to {}. Old temperature was {}.", t, oldT);
16:
17:     if(temperature.intValue() > 50) {
18:       logger.info("Temperature has risen above 50 degrees.");
19:     }
20:   }
21: }

``` 
#  Logback的配置介绍 
 1、Logger、appender及layout
  * **Logger作为日志的记录器**，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。
  * **Appender主要用于指定日志输出的目的地**，目的地可以是控制台、文件、远程套接字服务器、 MySQL、 PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。 
  * **Layout 负责把事件转换成字符串，格式化的日志信息的输出**。

2、logger context
**各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各 logger**。**其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。** getLogger方法以 logger 名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。

3、有效级别及级别的继承
Logger 可以被分配级别。级别包括：**TRACE、DEBUG、INFO、WARN 和 ERROR，定义于 ch.qos.logback.classic.Level类**。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。**root logger 默认级别是 DEBUG**。
^Logger name	^Assigned level	^Effective level^
|root	      |DEBUG	     |DEBUG|
|X	      |INFO	     |INFO|
|X.Y	      |none	     |INFO|
|X.Y.Z	      |ERROR	     |ERROR |           
```

package org.slf4j; 
public interface Logger {

  // Printing methods: 
  public void trace(String message);
  public void debug(String message);
  public void info(String message); 
  public void warn(String message); 
  public void error(String message); 
}

```

4、打印方法与基本的选择规则
打印方法决定记录请求的级别。
该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR。 

#  Logback的默认配置 

如果配置文件 **logback-test.xml 和 logback.xml** 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。最小化配置由一个关联到根 logger 的ConsoleAppender 组成。输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。root logger 默认级别是 DEBUG。

1、Logback的配置文件
Logback 配置文件的语法非常灵活。正因为灵活，所以无法用 DTD 或 XML schema 进行定义。尽管如此，可以这样描述配置文件的基本结构：以<configuration>开头，后面有零个或多个<appender>元素，有零个或多个<logger>元素，有最多一个<root>元素。

2、Logback默认配置的步骤
(1). 尝试在 classpath 下查找文件 logback-test.xml；
(2). 如果文件不存在，则查找文件 logback.xml；
(3). 如果两个文件都不存在，logback 用 Bas icConfigurator 自动对自己进行配置，这会导致记录输出到控制台。
```

<configuration debug="false">
  	<!--设置LogContext的名称，默认为default.可以配置以区分不同程序的log.-->
  	 <contextName>myAppName</contextName>  
    <!--property用于定义变量,引用${LOG_HOME}，这里我们定义日志文件的存储地址，以便接下来使用。 勿在 LogBack 的配置中使用相对路径-->  
    <property name="LOG_HOME" value="/home" />  
    <!-- 控制台输出 -->   
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder"> 
             <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>   
        </encoder> 
    </appender>
    <!-- 按照每天生成日志文件 -->   
    <appender name="FILE"  class="ch.qos.logback.core.rolling.RollingFileAppender">   
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern> 
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>   
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder"> 
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符--> 
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>   
        </encoder> 
        <!--日志文件最大的大小-->
       <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
         <MaxFileSize>10MB</MaxFileSize>
       </triggeringPolicy>
    </appender> 
   <!-- show parameters for hibernate sql 专为 Hibernate 定制 --> 
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder"  level="TRACE" />  
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor"  level="DEBUG" />  
    <logger name="org.hibernate.SQL" level="DEBUG" />  
    <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />  
    
    <!--myibatis log configure--> 
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>
    
 	 <!-- 使用logback设置某一个包或者具体的某一个类的日志打印.name为java中的包  
	<logger name="net.xby1993.logback">  
          <appender-ref ref="STDOUT"/>  
  	</logger>
 	--> 
    <!-- 日志输出级别 -->
    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root> 
     <!--日志异步到数据库 -->  
    <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
        <!--日志异步到数据库 --> 
        <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
           <!--连接池 --> 
           <dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <driverClass>com.mysql.jdbc.Driver</driverClass>
              <url>jdbc:mysql://127.0.0.1:3306/databaseName</url>
              <user>root</user>
              <password>root</password>
            </dataSource>
        </connectionSource>
  </appender>
</configuration>

```
##  关于<rollingPolicy>与<triggeringPolicy >的说明 

<rollingPolicy>:当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。
<triggeringPolicy>: 告知 RollingFileAppender 合适激活滚动。
###  rollingPolicy： 

**TimeBasedRollingPolicy**： **最常用的滚动策略**，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：
  * <fileNamePattern>:
必要节点，包含文件名及“%d”转换符， “%d”可以包含一个 java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。 RollingFileAppender 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
  * <maxHistory>:
可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且 <maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

**FixedWindowRollingPolicy**： 根据固定窗口算法重命名文件的滚动策略。有以下子节点：
  * <minIndex>:窗口索引最小值
  * <maxIndex>:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
  * <fileNamePattern >:
必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip
 
###  triggeringPolicy: 
**SizeBasedTriggeringPolicy**： 查看当前活动文件的大小，如果超过指定大小会告知 RollingFileAppender 触发当前活动文件滚动。只有一个节点:
  * <maxFileSize>:这是活动文件的大小，默认值是10MB。
例如：每天生成一个日志文件，保存30天的日志文件。
##  关于Layout生成日志格式的说明 
|Conversion specifier	Logger name	|Result|
|%logger	mainPackage.sub.sample.Bar	|mainPackage.sub.sample.Bar|
|%logger{0}	mainPackage.sub.sample.Bar	|Bar|
|%logger{5}	mainPackage.sub.sample.Bar	|m.s.s.Bar|
|%logger{10}	mainPackage.sub.sample.Bar	|m.s.s.Bar|
|%logger{15}	mainPackage.sub.sample.Bar	|m.s.sample.Bar|
|%logger{16}	mainPackage.sub.sample.Bar	|m.sub.sample.Bar|
|%logger{26}	mainPackage.sub.sample.Bar	|mainPackage.sub.sample.Bar|

|Conversion Pattern	|Result|
|%d	|2006-10-20 14:06:49,812|
|%date	2|006-10-20 14:06:49,812|
|%date{ISO8601}	|2006-10-20 14:06:49,812|
|%date{HH:mm:ss.SSS}	|14:06:49.812|
|%date{dd MMM yyyy;HH:mm:ss.SSS}	|20 oct. 2006;14:06:49.812|

###  生成颜色的日志： 

PatternLayout recognizes "%black", "%red", "%green","%yellow","%blue", "%magenta","%cyan", "%white", "%gray", "%boldRed","%boldGreen", "%boldYellow", "%boldBlue", "%boldMagenta""%boldCyan", "%boldWhite" and "%highlight" as conversion words. 
```

<encoder>
      <pattern>[%thread] %highlight(%-5level) %cyan(%logger{15}) - %msg %n</pattern>
    </encoder>

```
#  使用简介： 
依赖：
  *  logback-access.jar
  * logback-classic.jar
  * logback-core.jar
  * slf4j-api.jar
maven方式：
```

<dependency>  
       <groupId>ch.qos.logback</groupId>  
       <artifactId>logback-classic</artifactId>  
    <version>1.1.3</version>  
</dependency>  

```
```

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Demo{
     //定义一个全局的记录器，通过LoggerFactory获取
     private final static Logger logger = LoggerFactory.getLogger("net.xbynet.Demo"); 
     /**
     * @param args
     */
    public static void main(String[] args) {
        logger.info("logback 成功了");
        logger.error("logback 成功了");
    }
}

```
##  参数化日志记录 
```

Object entry = new SomeObject(); 
logger.debug("The entry is {}.", entry);

```

##  Hibernate4 JbossLogging配置使用slf4j 
http://stackoverflow.com/questions/11639997/how-do-you-configure-logging-in-hibernate-4-to-use-slf4j
方式一:
```

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.21</version>
</dependency>

```
方式二:
```

static { //runs when the main class is loaded.
    System.setProperty("org.jboss.logging.provider", "slf4j");
}

```
可以参考的地址：
http://aub.iteye.com/blog/1103685
http://www.cnblogs.com/yuanermen/archive/2012/02/13/2348942.html
http://www.cnblogs.com/yuanermen/archive/2012/02/13/2349609.html
http://stackoverflow.com/questions/11639997/how-do-you-configure-logging-in-hibernate-4-to-use-slf4j