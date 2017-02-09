title: 使用maven_profile实现多环境可移植构建 

#   使用maven profile实现多环境可移植构建 
参考：http://blog.csdn.net/mhmyqn/article/details/24501281
在开发过程中，**我们的软件会面对不同的运行环境，比如开发环境、测试环境、生产环境，**而我们的软件在不同的环境中，**有的配置可能会不一样，比如数据源配置、日志文件配置**、以及一些软件运行过程中的基本配置，那每次我们将软件部署到不同的环境时，都需要修改相应的配置文件**，这样来回修改，是个很麻烦的事情**。有没有一种方法能够让我们不用修改配置就能发布到不同的环境中呢？当然有，这就是接下来要做的事。

当然，这里的前提是使用maven做为构建工具。
**使用maven来实现多环境的构建可移植性，需要借助maven提供的profile功能，通过不同的环境激活不同的profile来达到构建的可移植性。**

##  一、配置profile 
首先是profile配置，在pom.xml中添加如下profile的配置：
```

<dependencies>....</dependencies>
<profiles>  
    <profile>  
        <!-- 本地开发环境 -->  
        <id>development</id>  
        <properties>  
            <profiles.active>development</profiles.active>  
            <deploy.url>http://host:port/manager/text</deploy.url>  
        </properties>  
        <activation>  
            <activeByDefault>true</activeByDefault>  
        </activation>  
    </profile>  
    <profile>  
        <!-- 测试环境 -->  
        <id>test</id>  
        <properties>  
            <profiles.active>test</profiles.active>  
            <deploy.url>http://host:port/manager/text</deploy.url>  
        </properties>  
    </profile>  
    <profile>  
        <!-- 生产环境 -->  
        <id>production</id>  
        <properties>  
            <profiles.active>production</profiles.active>  
            <deploy.url>http://host:port/manager/text</deploy.url>  
        </properties>  
    </profile>  
</profiles>  

```
这里定义了三个环境，分别是development（开发环境）、test（测试环境）、production（生产环境），其中开发环境是默认激活的（activeByDefault为true），这样如果在不指定profile时默认是开发环境。
同时每个profile还定义了两个属性，其中profiles.active表示被激活的profile的名称，deploy.url表示发布服务器的地址。我们需要在下面使用到这两个属性。
另外host和port分别是发布服务器的主机地址和端口号。

##  二、配置文件 

针对不同的环境，我们定义不同的配置文件，而这些配置文件都做为资源文件放到maven工程的resources目录下，即src/main/resources目录下，且各个环境的配置分别放到相应的目录下，而所有环境都公用的配置，直接放到src/main/resources目录下即可。如下图所示：
![](/data/dokuwiki/tooluse/pasted/20151019-161856.png)
如图所示，开发环境、测试环境、生产环境的**配置文件分别放到src/main/resources目录下的development、test、production三个子目录中**，而所有环境都公用的配置文件spring-applicationContext.xml直接放到src/main/resources目录下。其中jdbc.properties配置数据源、logback.xml配置日志。

##  三、maven资源插件配置 
在pom中的build节点下，配置资源文件的位置，如下所示：
```

<build>  
    <resources>  
        <resource>  
            <directory>${project.basedir}/src/main/resources</directory>  
            <!-- 资源根目录排除各环境的配置，使用单独的资源目录来指定 -->  
            <excludes>  
                <exclude>test/*</exclude>  
                <exclude>production/*</exclude>  
                <exclude>development/*</exclude>  
            </excludes>  
        </resource>  
        <resource>  
            <directory>${project.basedir}/src/main/resources/${profiles.active}</directory>  
        </resource>  
    </resources>  
    </plugins>  
</build>  

```
首先第一个资源文件位置src/main/resources需要排除提各个环境的配置文件，各个环境的配置我们在第二个<resource>节点中通过前面在profile中配置的profiles.active属性来指定。即` src/main/resources/${profiles.active} `。这样在激活指定的profile时，会加载指定目录下的配置文件，如当前激活的是production profile，那么这个资源目录就是src/main/resources/production。这样就达到了不同环境加载不同配置的目的。

##  四、配置tomcat-maven-plugin插件 
```

<plugin>  
    <groupId>org.codehaus.mojo</groupId>  
    <artifactId>tomcat-maven-plugin</artifactId>  
    <version>1.2-SNAPSHOT</version>  
    <configuration>  
        <url>${deploy.url}</url>  
        <server>tomcat</server>  
        <path>/appcontext</path>  
    </configuration>  
</plugin>  

```
其中发布的<url>节点就是在前面profile中配置的deploy.url属性，这样不同的环境就指定了不同的发布地址。<server>和<path>节点分别是发布服务器的用户配置的id以及应用的context名称。

##  五、构建或发布 
所有需要的配置就完成了，下面是见证奇迹的时候了。通过在运行maven命令时指定不同的profile即可构建不同环境需要的war包或发布到不同的环境了 。如：
` mvn clean package -Pproduction `即构建出生产环境需要的war包
` mvn tomcat:redeploy -Ptest ` 即发布到测试环境

由于默认的profile是development，所以如果我们不指定profile，那么加载就是开发环境deployment下的配置文件了。即我们在本地开发测试时，不用关心profile的问题。
` 而且本地开发时在eclipse中使用tomcat插件来进行热部署时也不需要额外的配置。 `**真正的做到了根据不同环境来自动切换，即可移植的构建。**
另外，在进行持续集成时，使用hudson集成maven同样是非常非常方便的。

##  附录 多环境打包js，css等文件也是可以的
```

		<resources>
			<resource>
				<directory>${project.basedir}/src/main/deploys/${profiles.active}</directory>
				<targetPath>static</targetPath>
			</resource>
		</resources>

```

##  具体示列 
```

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>net.xby1993</groupId>
	<artifactId>spring-test</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-test Maven Webapp</name>
	<url>http://maven.apache.org</url>
	<properties>
		<org.springframework.version>3.2.8.RELEASE</org.springframework.version>
		<!-- <org.springframework.version>4.0.2.RELEASE</org.springframework.version> -->
	</properties>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
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
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
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
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.7</version>
		</dependency>

		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>3.6.10.Final</version>
		</dependency>
		<dependency>
			<groupId>javassist</groupId>
			<artifactId>javassist</artifactId>
			<version>3.13.0.GA</version>
		</dependency>

		<!-- end -->
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
	</dependencies>
	<profiles>
		<profile>
			<!-- 本地开发环境 -->
			<id>development</id>
			<properties>
				<profiles.active>development</profiles.active>
			</properties>
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>
		</profile>
		<profile>
			<!-- 测试环境 -->
			<id>test</id>
			<properties>
				<profiles.active>test</profiles.active>

			</properties>
		</profile>
		<profile>
			<!-- 生产环境 -->
			<id>production</id>
			<properties>
				<profiles.active>production</profiles.active>

			</properties>
		</profile>
	</profiles>
	<repositories>
		<repository>
			<id>nexus</id>
			<name>local private nexus</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>nexus</id>
			<name>local private nexus</name>
			<url>http://maven.oschina.net/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</pluginRepository>
	</pluginRepositories>

	<build>
		<finalName>spring-test</finalName>
		<!-- 确定项目的开发编译版本都1.7 -->
		<resources>
			<resource>
				<directory>${project.basedir}/src/main/resources</directory>
				<!-- 资源根目录排除各环境的配置，使用单独的资源目录来指定 -->
				<excludes>
					<exclude>test/*</exclude>
					<exclude>production/*</exclude>
					<exclude>development/*</exclude>
					<exclude>**/lib/*</exclude>
				</excludes>
			</resource>
			<resource>
				<directory>${project.basedir}/src/main/resources/${profiles.active}</directory>
			</resource>
		</resources>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.0.2</version>
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

##  另外一种思路 
结合Profile与resource的filter来实现