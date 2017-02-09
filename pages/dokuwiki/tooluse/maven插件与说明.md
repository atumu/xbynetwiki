title: maven插件与说明 

#  Maven插件与说明 
**` 一个插件(plugin)可以设置多个目标（goal）。而每个目标都会绑定至Maven默认生命周期的一个阶段(phase)上。 `**
##  Maven 生命周期阶段 
###  1、清理生命周期 (clean) 
运行mvn clean 将调用清理生命周期，它包含了三个生命周期阶段：
• pre-clean
• clean
• post-clean
在清理生命周期中最有意思的阶段是clean 阶段。Clean 插件的clean 目标（clean:clean）被绑定到清理生命周期中的clean 阶段。目标clean:clean 通过删除构建目录删除整个构建的输出
### 2、 Maven 默认生命周期阶段 
生命周期阶段 描述
validate 验证项目是否正确，以及所有为了完整构建必要的信息是否可用
generate-sources 生成所有需要包含在编译过程中的源代码
process-sources 处理源代码，比如过滤一些值
generate-resources 生成所有需要包含在打包过程中的资源文件
` process-resources ` 复制并处理资源文件至目标目录，准备打包
` compile ` 编译项目的源代码
process-classes后处理编译生成的文件，例如对Java 类进行字节码增强（bytecodeenhancement）
generate-test-sources 生成所有包含在测试编译过程中的测试源码
process-test-sources 处理测试源码，比如过滤一些值
generate-test-resources 生成测试需要的资源文件
process-test-resources 复制并处理测试资源文件至测试目标目录
test-compile 编译测试源码至测试目标目录
` test `使用合适的单元测试框架运行测试。这些测试应该不需要代码被打包或发布
prepare-package在真正的打包之前，执行一些准备打包必要的操作。这通常会产生一个包的展开的处理过的版本（将会在Maven 2.1+中实现）
` package ` 将编译好的代码打包成可分发的格式，如JAR，WAR，或者EAR
pre-integration-test执行一些在集成测试运行之前需要的动作。如建立集成测试需要的环境
integration-test 如果有必要的话，处理包并发布至集成测试可以运行的环境
post-integration-test 执行一些在集成测试运行之后需要的动作。如清理集成测试环境。
verify 执行所有检查，验证包是有效的，符合质量规范
` install ` 安装包至本地仓库，以备本地的其它项目作为依赖使用
` deploy `复制最终的包至远程仓库，共享给其它开发人员和项目（通常和一次正式的发布相关）

###  3、站点生命周期 (site) 

• pre-site
• site
• post-site
• site-deploy
默认绑定到站点生命周期的目标是：
• site - site:site
• site-deploy -site:deploy

##  打包相关生命周期 
**1、JAR 打包默认的目标**
` JAR 是默认的打包类型 `，是最常用的，因此也就是生命周期配置中最经常遇到的打包类型
生命周期阶段 目标
process-resources resources:resources
compile compiler:compile
process-test-resources resources:testResources
test-compile compiler:testCompile
test surefire:test
package jar:jar
install install:install
deploy deploy:deploy

**2、POM 打包默认的目标**
POM 是最简单的打包类型。不像一个JAR，SAR，或者EAR，**它生成的构件只是它本身**。没有代码需要测试或者编译，也没有资源需要处理
生命周期阶段 目标
package site:attach-descriptor
install install:install
deploy deploy:deploy

**3、WAR 打包默认的目标**
WAR 打包类型和JAR 以及EJB 类似。例外是这里的package 目标是war:war。注意war:war 插件需要一个web.xml 配置文件在项目的src/main/webapp/WEB-INF 目录中
生命周期阶段 目标
process-resources resources:resources
compile compiler:compile
process-test-resources resources:testResources
test-compile compiler:testCompile
test surefire:test
package war:war
install install:install
deploy deploy:deploy


##  单元测试Surefire 插件 
是负责运行单元测试的插件。支持Junit4与TestNG
surefire 插件用来在maven构建生命周期的test phase执行一个应用的单元测试。它会产生两种不同形式
的测试结果报告：
1）.纯文本
2）.xml文件格式的
默认情况下，这些文件生成在工程的${basedir}/target/surefire-reports，目录下（basedir指的是pom文件所在的目录）。

如何使用？
使用该插件很简单,使用mvn surefire:test或者mvn test都可以运行工程下的单元测试。

如果你使用了Junit,那么引入这个依赖:
```

<dependency>
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>4.11</version>
<scope>test</scope>
</dependency>

```

```

Maven Surefire 插件有一个test 目标，该目标被绑定在了test 阶段。 test 目标执行项目中所有能在src/test/java 找到的并且文件名与**/Test*.java，**/*Test.java 和 **/*TestCase.java 匹配的所有单元测试

```
  
在默认情况下，maven-surefire-plugin的test目标会自动执行测试源码路径（默认为src/test/java/）下所有的测试类。
测试资源目录：src/test/resources
跳过测试
要想跳过测试，在命令行加入参数skipTests就可以了。如：
```

mvn package -DskipTests 

``` 
也可以在pom配置中提供该属性。
```

<plugin> 
			    <groupId>org.apache.maven.plugins</groupId> 
			    <artifactId>maven-surefire-plugin</artifactId>
			  <version>2.19</version>
			    <configuration>  
			        <skip>true</skip>  <!--跳过单元测试 如果你想要整个的跳过测试，你也可以运行如下的命令：$ mvn install -Dmaven.test.skip=true 而不是直接写死在pom.xml中-->
                                <testFailureIgnore>true</testFailureIgnore> <!--测试失败时继续构建 -->
			    </configuration> 
			</plugin>	

```

##  Compiler编译插件 
```

<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.3</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>

```
**如果你想要自定义源码的位置，你也可以更改构建配置**。如果你想要存储项目的源码至src/java 而非src/main/java，让构建输出至classes 而非target/classes，你可以覆盖定义在超级POM 中的sourceDirectory 的默认值。
Example 10.10. 覆盖默认的源码和输出目录
```

<build>
...
<sourceDirectory>src/java</sourceDirectory>
<outputDirectory>classes</outputDirectory>
...
</build>

```

##  maven-war-plugin 
更多参数请参考：http://maven.apache.org/plugins/maven-war-plugin/exploded-mojo.html
http://docs.iteye.com/blog/1276411
http://www.infoq.com/cn/news/2011/06/xxb-maven-9-package/
```

<plugin>
			    <groupId>org.apache.maven.plugins</groupId>
			    <artifactId>maven-war-plugin</artifactId>
			    <version>2.6</version>				    
			    <configuration>
					<webResources>  
			            <resource>  
			                <directory>target/js</directory>  
			            </resource>  
			        </webResources> 
			    </configuration>
			  </plugin>

```
```

<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-war-plugin</artifactId>  
  <version>2.6</version>
    <configuration>
      <warName>${project.build.finalName}</warName>
       <warSourceExcludes>WEB-INF/lib/log4j-${log4j.version}.jar</warSourceExcludes>   
      <warSourceIncludes></warSourceIncludes>
        <webResources>  
            <resource>  
                <directory>src/main/package1</directory>  
              <excludes>
           	 <exclude>**/*.jpg</exclude>
          	</excludes>
                <targetPath>WEB-INF</targetPath>   <!-- defaults to root of webapp structure 默认相对于webapp路径-->
            </resource>  
        </webResources>  
    </configuration>  
</plugin> 

```
##  maven-resources-plugin 
为了使项目结构更为清晰，Maven区别对待Java代码文件和资源文件，maven-compiler-plugin用来编译Java代码，maven-resources-plugin则用来处理资源文件。默认的主资源文件目录是src/main/resources，很多用户会需要添加额外的资源文件目录，这个时候就可以通过配置maven-resources-plugin来实现。此外，资源文件过滤也是Maven的一大特性，你可以在资源文件中使用${propertyName}形式的Maven属性，然后配置maven-resources-plugin开启对资源文件的过滤，之后就可以针对不同环境通过命令行或者Profile传入属性的值，以实现更为灵活的构建。
配置选项请参考：http://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html
http://maven.apache.org/shared/maven-filtering/
http://ju.outofmemory.cn/entry/83155
http://www.tuicool.com/articles/JfaA7r
http://m.blog.csdn.net/blog/inte_sleeper/10128041
maven的resources插件职责是将项目的资源文件拷贝到目标目录。maven将资源划分为两类：main resources 和 test resources。因此有如下三个相应的goal:
1. resources:resources
2. resources:testResources
3. resources:copy-resources

大部分生命周期将resources:resources 目标绑定到process-resources 阶段。process-resources 阶段处理资源并将资源复制到输出目录。如果你没有自己自定义超级POM 中的默认目录位置，Maven 就会将**${basedir}/src/main/resources** 中的文件复制到${basedir}/target/classes，或者是由**${project.build.outputDirectory}**定义的目录。除了复制资源文件至输出目录，Maven同时也会在资源上应用` 过滤器，能让你替换资源文件中的一些符号。 `就像在POM中我们通过${...}标记引用变量一样，你也可以使用同样的语法在你项目的资源文件中引用变量**。与profile 联系起来，这样的特性就能用来生成针对不同部署平台的构件。**maven默认src/main/resources下的资源。如果你有资源文件不是存放在这个目录下，那么可以通过<resources>元素指定：

```

 <build>
           ...
    <filters>
        <filter>${user.home}/antx.properties</filter>
    </filters>
           <resources>
                <resource>
                     <directory>${basedir}/apk</directory>
                     <includes>
                     	<include>**/*.apk</include>
                   </includes>
                   <excludes>  
                    <exclude>*.xml</exclude>  
                </excludes>  
                  <!--   maven default生命周期，process-resources阶段执行maven-resources-plugin插件的resources目标处理主资源目下的资源文件时，指定处理后的资源文件输出目录，默认是${build.outputDirectory}指定的目录   -->
                     <targetPath>META-INF/apk</targetPath>
                   <!-- maven default生命周期，process-resources阶段执行maven-resources-plugin插件的resources目标处理主资源目下的资源文件时，是否对主资源目录开启资源过滤 -->  
                     <filtering>true</filtering>  
                </resource>
                <resource>
                     <directory>${basedir}/src/main/resources</directory>
                </resource>
           </resources>
           ...
      </build>

```
除了指定原资源文件和目标目录，我们还可以指定包含或者排除哪些目录或者文件。
```

 <build>
           ...
           <resources>
                <resource>
                     <directory>${basedir}/sample</directory>
                     <targetPath>META-INF/sample</targetPath>
                </resource>
                <resource>
                     <directory>${basedir}/docs</directory>
                     <targetPath>META-INF/docs</targetPath>
                </resource>
                <!-- 把源代码打在一起，方便后面SDK打包 -->
                <resource>
                     <directory>${project.build.sourceDirectory}</directory>
                     <includes>
                          <include>com/qq/buy/api/client/**/*.java</include>
                     </includes>
                     <targetPath>META-INF/source</targetPath>
                </resource>
                <resource>
                     <directory>${basedir}/src/main/resources</directory>
                </resource>
           </resources>
      </build>

```
最后，我们还可以**对资源文件进行变量替换**，这个maven称之为**resources filter**。变量值可以来自 system properties, project properties, filter resources 和 command line.
resources的三个goals都支持resources filter。
```

<build>
           ...
           <resources>
                <resource>
                     <directory>src/main/webapp/WEB-INF</directory>
                     <filtering>true</filtering>
                     <includes>
                          <include>**/*.xml</include>
                     </includes>
                </resource>
                <resource>
                     <directory>src/main/resources</directory>
                     <filtering>false</filtering>
                </resource>
           </resources>
           <filters>
                <filter>config.properties</filter>
           </filters>
      </build>

```
###  资源过滤器 

下面具体介绍一下**资源过滤器**：
为了阐述资源过滤，假设你有个带有XML 文件src/main/resources/META-INF/service.xml 的项目。你想要提取出一些配置变量至一个属性文件。换句话说，你可能想要为你的数据库引用JDBC URL，用户名和密码，
并且你不想将这些值直接放到service.xml 文件里，而是想要使用一个属性文件来存储你程序中的所有配置点。这么做能让你将所有配置信息固定到单独的一个属性文件中，当你需要面对一个新的部署环境的时候，就很容易更改配置的值。首先，看一下src/main/resources/META-INF/service.xml 的内容。
Example 10.4. 在项目资源中使用属性
```

<service>
<!-- This URL was set by project version ${project.version} -->
<url>${jdbc.url}</url>
<user>${jdbc.username}</user>
<password>${jdbc.password}</password>
</service>

```
这些自定义的变量在一个属性文件src/main/filters/default.properties 中定义。
Example 10.5. src/main/filters 中的default.properties
```

jdbc.url=jdbc:hsqldb:mem:mydb
jdbc.username=sa
jdbc.password=

```
要**配置使用该default.properties 文件的资源过滤**，我们需要在这个项目的POM 中指定两样东西：
  * 构建配置的filters 元素中的属性文件列表，
  * 以及一个标记告诉Maven资源目录需要过滤。
` 默认的Maven 行为会跳过过滤，只是将资源复制到输出目录；你需要显式的配置资源过滤，否则Maven 就会置之不理。 `这种Maven 资源过滤的默认行为是为了确保不让Maven 替换掉一些你不想替换的${...}引用。
Example 10.6. 过滤资源 (替换属性)
```

<build>
<filters>
<filter>src/main/filters/default.properties</filter>
</filters>
<resources>
<resource>
<directory>src/main/resources</directory>
<filtering>true</filtering>
</resource>
</resources>
</build>

```
正如 Maven 中所有目录一样，` 资源目录并非一定要在src/main/resources。这只是定义在超级POM 中的默认值。 `你应该也注意到你不需要将你所有的资源合并到一个单独的目录中。**你可以将资源分离至src/main 目录下的独立的目录中**。假设你有个项目包含了数百个XML 文档和数百个图片。你可能希望创建两个目录src/main/xml 和src/main/images 来存储这些内容，而不是将它们混合在src/main/resources 目录
中。为了添加资源目录列表，你需要在你的构建配置中加入如下的resource 元素：
Example 10.7. 配置额外的资源目录
```

<build>
...
<resources>
<resource>
<directory>src/main/resources</directory>
</resource>
<resource>
<directory>${basedir}/src/main/xml</directory>
  <includes>
<include>run.bat</include>
<include>run.sh</include>
</includes>
<targetPath>cmd</targetPath> <!--默认相对于target目录 -->
</resource>
<resource>
<directory>src/main/images</directory>
</resource>  

```
@echo off
java -jar ${project.build.finalName}.jar %*
在运行了mvn process-resources 之后，目录${basedir}下会有一个含有如下内
容的名为run.bat 的文件：
@echo off
java -jar simple-cmd-2.3.1.jar %*

在带有很多不同种类资源的复杂的项目中，我们会发现将资源分离到多个目录下十分有利，原因之一就是我们可以为特定的资源子集自定义过滤器。针对不同的过滤需求，除了将资源存储到不同的目录下，我们还可以使用更复杂的一组包含/排除模式来匹配那些符合模式的资源文件。
##  Maven Assembly打包分发插件 
Maven Assembly 插件是一个用来创建你应用程序特有分发包的插件。你可以使用Maven Assembly 插件**以你希望的任何形式来装配输出**，只需定义一个自定义的装配描述符。后面的章节我们会说明如何创建一个自定义装配描述符，为应用程序生成一个更复杂的存档文件。本章我们将会**使用预定义的jar-with-dependencies 格式**。要配置 Maven Assembly 插件， 我们需要在pom.xml 中的 build 配置中添加如下的plugin 配置。
```

<build>
<plugins>
<plugin>
<artifactId>maven-assembly-plugin</artifactId>
<configuration>
<descriptorRefs>
<descriptorRef>jar-with-dependencies</descriptorRef>
</descriptorRefs>
</configuration>
</plugin>
</plugins>
</build>

```
添加好这些配置以后，你可以通过` 运行mvn assembly:assembly ` 来构建这个装配。$ mvn install assembly:assembly

##  JS/CSS压缩插件 
```

<!-- 正式生产环境 -->
		<profile>
			<id>product</id>
			<properties>
				<profiles.active>product</profiles.active>
			</properties>
			<build>
			<plugins>
			  <!-- YUI Compressor Maven压缩插件 -->			
		      <plugin>
		        <groupId>net.alchim31.maven</groupId>
		        <artifactId>yuicompressor-maven-plugin</artifactId>
		        <version>1.5.1</version> 
				<executions>
					<execution>
						<phase>compile</phase>
						<goals>
							<goal>compress</goal>
						</goals>
					</execution>
				</executions>		        
				<configuration>
					<!-- 读取js,css文件采用UTF-8编码 -->
					<encoding>UTF-8</encoding>
					<!-- 不显示js可能的错误 -->
					<jswarn>false</jswarn>
					<!-- 若存在已压缩的文件，会先对比源文件是否有改动有改动便压缩，无改动就不压缩 -->
					<force>false</force>
					<!-- 在指定的列号后插入新行 -->
					<linebreakpos>-1</linebreakpos>
					<!-- 压缩之前先执行聚合文件操作 -->
					<preProcessAggregates>false</preProcessAggregates>
					<!-- 压缩后保存文件后缀 -->
					<!--<suffix>.min</suffix> -->
					<nosuffix>true</nosuffix>
					<!-- 源目录，即需压缩的根目录 -->
					<sourceDirectory>${project.basedir}/src/main/webapp/static</sourceDirectory>
					<!-- 压缩js和css文件 -->
					<includes>
						<include>**/*.js</include>
					</includes>
					<!-- 以下目录和文件不会被压缩 -->
					<excludes>
						<exclude>**/*.min.js</exclude>
						<exclude>**/*.debug.js</exclude>
						<exclude>**/*.min.css</exclude>
						<exclude>**/*.css</exclude>
					</excludes>
					<!-- 压缩后输出文件目录 -->
					<outputDirectory>${project.basedir}/target/js/static</outputDirectory>
				</configuration>
		      </plugin>	
			  <plugin>
			    <groupId>org.apache.maven.plugins</groupId>
			    <artifactId>maven-war-plugin</artifactId>
			    <version>2.6</version>				    
			    <configuration>
					<webResources>  
			            <resource>  
			                <directory>target/js</directory>  
			            </resource>  
			        </webResources> 
			    </configuration>
			  </plugin>			      
		      </plugins>		
			</build>
		</profile>

```
