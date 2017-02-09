title: maven高级之分war_jar打包及合并多个war 

#  maven高级之分war，jar打包及合并多个war 
场景描述：
有一些公用的前后台代码需要在各项目中使用，每次又不想拷贝。而这个公用的部分又是一个完整的web基项目，并且还在持续完善中。
思路：将这个web基项目，分不同profile打包成jar或war。其它项目通过overlays合并war
```

<?xml version="1.0"?>
<project
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
	xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.demo</groupId>
	<artifactId>${artifactId.id}</artifactId>
	<version>0.0.1</version>
	<name>${artifactId}</name>
	<packaging>${packaging.type}</packaging>
	<url>http://maven.apache.org</url>
	<properties>
		
	</properties>

	<dependencies>

		<!-- MySQL jar包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.25</version>
		</dependency>
		<!-- 注册与登录验证加密算法 -->
		<dependency>
			<groupId>com.lambdaworks</groupId>
			<artifactId>scrypt</artifactId>
			<version>1.4.0</version>
		</dependency>
		
		<!-- jstl -->
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>

		<!-- RSA加密实现，登录时密码传输加密 -->
		<dependency>
			<groupId>org.bouncycastle</groupId>
			<artifactId>bcprov-jdk15</artifactId>
			<version>1.45</version>
		</dependency>

		<!-- RSAUtils中使用 -->
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>
		<!-- RSAUtils中使用 -->
		<dependency>
			<groupId>commons-lang</groupId>
			<artifactId>commons-lang</artifactId>
			<version>2.6</version>
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
			<version>2.3.2-b01</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
	
	<!-- 定义profile -->
	<profiles>
		<!-- 开发调试时使用 -->
		<profile>
			<id>default</id>
			<activation>
                <activeByDefault>true</activeByDefault>
            </activation>
			<properties>
				<artifactId.id>base</artifactId.id>
                <packaging.type>war</packaging.type>
            </properties>
        </profile>
		<!-- 打war包 -->
		<profile>
			<id>war</id>
            <properties>
            	<artifactId.id>base-war</artifactId.id>
                <packaging.type>war</packaging.type>
            </properties>
			<build>
				<plugins>
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-war-plugin</artifactId>
						<version>2.6</version>
						<configuration>
							<!-- 打成war包时排除class文件，排除所有lib下的文件 -->
							<packagingExcludes>
								WEB-INF/classes/com/demo/**,
								WEB-INF/lib/*.xml,
								WEB-INF/lib/*.properties,
								WEB-INF/lib/*.jar
							</packagingExcludes>
						</configuration>
					</plugin>
				</plugins>
			</build>
		</profile>
		<!-- 打jar包 -->
		<profile>
			<id>jar</id>
			<properties>
				<artifactId.id>demo-jar</artifactId.id>
                <packaging.type>jar</packaging.type>
            </properties>
			<build>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-jar-plugin</artifactId>
					<version>2.6</version>
					<executions>
						<execution>
							<phase>process-classes</phase>
							<goals>
								<goal>jar</goal>
							</goals>
							<configuration>
								<classesDirectory>target/classes</classesDirectory>
								<finalName>${artifactId}-${version}</finalName>
								<!-- <outputDirectory>src/main/webapp/WEB-INF/lib</outputDirectory> -->
								<includes>
									<include>com/demo/**</include>
								</includes>
							</configuration>
						</execution>
					</executions>
				</plugin>
			</plugins>
			</build>
		</profile>
    </profiles>
	
	<build>
		<finalName>${artifactId}-${version}</finalName>
		<plugins>
			<!-- 确定项目的开发编译版本都1.7 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.0.2</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF8</encoding>
				</configuration>
				<dependencies>
					<dependency>
						<groupId>org.codehaus.plexus</groupId>
						<artifactId>plexus-compiler-javac</artifactId>
						<version>1.8.1</version>
					</dependency>
				</dependencies>
			</plugin>
		</plugins>
	</build>
</project>

```

我们打包deploy的时候就可以直接通过
mvn -Pwar clean deploy
mvn -Pjar clean deploy
即可分开打包部署到私服

其它项目要依赖该项目可以这么做：
```

	<dependencies>
		<dependency>
			<groupId>com.demo</groupId>
			<artifactId>base-war</artifactId>
			<version>${base.version}</version>
			<type>war</type>
		</dependency>
		<dependency>
			<groupId>com.css.sword</groupId>
			<artifactId>base-jar</artifactId>
			<version>${base.version}</version>
		</dependency>
	</dependencies>
	<build>

		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.3</version>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
					<encoding>${project.encoding}</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<overlays>
						<overlay>
							<groupId>com.demo</groupId>
							<artifactId>base-war</artifactId>
						</overlay>
					</overlays>
				</configuration>
			</plugin>
		</plugins>
	</build>

```

新建运行项目的时候需要运行mvn clean package,然后在target下生成解压的war与当前项目代码合并的项目。
添加至eclipse servers需要配置的**deployment assembly**如下：
![](/data/dokuwiki/tooluse/pasted/20160922-104808.png)