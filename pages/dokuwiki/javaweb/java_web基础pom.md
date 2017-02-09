title: java_web基础pom 

#  java web pom 
目录：
1、基础pom
2、Spring+SpringMVC+Hibernate+C3P0
##  基础POM 
```

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

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.0.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.el</groupId>
			<artifactId>el-api</artifactId>
			<version>2.2.5</version>
			<scope>provided</scope>
		</dependency>

```
##  Spring+SpringMVC+Hibernate+C3P0 
```

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>net.xby1993.test</groupId>
	<artifactId>spring-test</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-test Maven Webapp</name>
	<url>http://maven.apache.org</url>
	<properties>
		<org.hibernate.version>3.6.10.Final</org.hibernate.version>
		<org.springframework.version>3.2.8.RELEASE</org.springframework.version>
		<jackson.version>1.8.9</jackson.version>
	</properties>
	<repositories>
		<repository>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
			<id>public</id>
			<name>Public Repositories</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>public</id>
			<name>Public Repositories</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
		</pluginRepository>
	</pluginRepositories>
	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
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
			<artifactId>spring-orm</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.tiles</groupId>
			<artifactId>tiles-extras</artifactId>
			<version>3.0.5</version>
		</dependency>
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
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.7</version>
		</dependency>
		<!-- <dependency> <groupId>org.codehaus.jackson</groupId> <artifactId>jackson-core-lgpl</artifactId> 
			<version>${jackson.version}</version> </dependency> <dependency> <groupId>org.codehaus.jackson</groupId> 
			<artifactId>jackson-mapper-lgpl</artifactId> <version>${jackson.version}</version> 
			</dependency> -->

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

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.el</groupId>
			<artifactId>el-api</artifactId>
			<version>2.2.1-b04</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>
		<dependency>
			<groupId>javassist</groupId>
			<artifactId>javassist</artifactId>
			<version>3.13.0.GA</version>
		</dependency>
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>
		<dependency>
			<groupId>Mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
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
			<artifactId>slf4j-api</artifactId>
			<version>1.7.9</version>
		</dependency>
		<!-- junit -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>

		<!-- struts2依赖包
		<dependency>
			<groupId>org.apache.struts</groupId>
			<artifactId>struts2-core</artifactId>
			<version>2.3.14</version>
		</dependency>
		 -->
	</dependencies>
	<build>
		<finalName>hibernate-test</finalName>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.6</version>
				<!-- <configuration> <failOnMissingWebXml>false</failOnMissingWebXml> 
					</configuration> -->
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.3</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>


```