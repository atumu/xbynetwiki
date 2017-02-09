title: maven学习初步 

##  Maven学习初步 
Maven官网：
Maven插件:http://maven.apache.org/plugins/index.html
Maven生命周期：http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html，https://maven.apache.org/ref/3.3.9/maven-core/lifecycles.html


https://my.oschina.net/huangyong/blog/194583

Maven 是 Apache 组织下的一个跨平台的项目管理工具，它主要用来帮助实现项目的构建、测试、打包和部署。Maven 提供了标准的软件生命周期模型和构建模型，通过配置就能对项目进行全面的管理。它的跨平台性保证了在不同的操作系统上可以使用相同的命令来完成相应的任务。Maven 将构建的过程抽象成一个个的生命周期过程，在不同的阶段使用不同的已实现插件来完成相应的实际工作，这种设计方法极大的避免了设计和脚本编码的重复，极大的实现了复用。
#  索引预览： 

maven基础,坐标，依赖配置
插件和仓库，设置代理
依赖，聚合，继承，传递依赖，可选依赖，依赖归类。
maven属性，
maven生命周期,phases阶段，goal目标。
maven仓库
补充内容：
          生成可执行jar
          依赖树
          配置文件profile
          maven goal目标
##  表 1. Maven 目录结构 


src/main/java	Application/Library sources
src/main/resources	Application/Library resources
src/main/filters	Resource filter files
src/main/assembly	Assembly descriptors
src/main/config	Configuration files
src/main/scripts	Application/Library scripts
src/main/webapp	Web application sources
src/test/java	Test sources
src/test/resources	Test resources
src/test/filters	Test resource filter files
src/site	Site
LICENSE.txt	Project's license
README.txt	Project's readme



#  项目对象模型 POM-Maven 的灵魂 

POM 即 Project Object Module，项目对象模型，在 pom.xml 文件中定义了项目的基本信息、源代码、配置文件、开发者的信息和角色、问题追踪系统、组织信息、项目授权、项目的 url、以及构建项目所用的插件，依赖继承关系。开发人员需按照 maven 定义的规则进行 POM 文件的编写，清单 1 为一个 POM 文件示例。
清单 1. POM 文件示例
```

<project xmlns="http://maven.apache.org/POM/4.0.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                     http://maven.apache.org/maven-v4_0_0.xsd">
 <modelVersion>4.0.0</modelVersion>
 <! – The Basics – >
 <groupId> … </groupId>
 <artifactId> … </artifactId>
 <version> … </version>
 <packaging> … </packaging>
 <dependencies> … </dependencies>
 <parent> … </parent>
 <dependencyManagement> … </dependencyManagement>
 <modules> … </modules>
 <properties> … </properties>
 <! – Build Settings – >
 <build> … </build>
 <reporting> … </reporting>
 <! – More Project Information – >
 <name> … </name>
 <description> … </description>
 <url> … </url>
 <inceptionYear> … </inceptionYear>
 <licenses> … </licenses>
 <organization> … </organization>
 <developers> … </developers>
 <contributors> … </contributors>
 <! – Environment Settings – >
 <issueManagement> … </issueManagement>
 <ciManagement> … </ciManagement>
 <mailingLists> … </mailingLists>
 <scm> … </scm>
 <prerequisites> … </prerequisites>
 <repositories> … </repositories>
 <pluginRepositories> … </pluginRepositories>
 <distributionManagement> … </distributionManagement>
 <profiles> … </profiles>
</project>

```

**pom.xml隐式变量**
**env**
env 变量暴露了你操作系统或者shell 的环境变量。例如，在Maven POM 中一个对${env.PATH}的引用将会被${PATH}环境变量替换（或者Windows 中的%PATH%）。

**project**
project 变量暴露了POM。你可以使用点标记（.）的路径来引用POM 元素的值。例如，在本节中我们使用过groupId 和artifactId 来设置构建配置中的finalName 元素。这个属性引用的语法是：
${project.groupId}-${project.artifactId}。

**settings**
settings 变量暴露了Maven settings 信息。可以使用点标记（.）的路径来引用settings.xml 文件中元素的值。例如，${settings.offline}会引用~/.m2/settings.xml 文件中offline 元素的值。

**Java 系统属性**
所有可以通过java.lang.System 中getProperties()方法访问的属性都被暴露成POM 属性。一些系统属性的例子是：${user.name}，${user.home}，${java.home}，和${os.name}。一个完整的系统属性列表可以在java.lang.System 类的Javadoc 中找到。

**自定义属性：**
```

<properties>
<foo>bar</foo>
</properties>

```
#  依赖 
会包含基本的groupId, artifactId,version等元素，根元素project下的dependencies可以包含一个或者多个dependency元素，以声明一个或者多个依赖。
下面详细讲解每个依赖可以包含的元素：
 groupId,artifactId和version：依赖的基本坐标，对于任何一个依赖来说，基本坐标是最重要的，Maven根据坐标才能找到需要的依赖
 type: 依赖的类型，对应于项目坐标定义的packaging。大部分情况下，该元素不必声明，其默认值是jar,war
 scope: 依赖的范围，下面会进行详解
 optional: 标记传递依赖是否可选
 exclusions: 用来排除传递性依赖，下面会进行详解
classifier代表附属构建如javadoc,source等
 大部分依赖声明只包含基本坐标。

#  Maven 插件和仓库 

Maven 本质上是一个插件框架，它的核心并不执行任何具体的构建任务，仅仅定义了抽象的生命周期，所有这些任务都交给插件来完成的。每个插件都能完成至少一个任务，每个任务即是一个功能，将这些功能应用在构建过程的不同生命周期中。这样既能保证拿来即用，又能保证 maven 本身的繁杂和冗余。
例如清单 2 中的代码就是要在 validate 这个阶段执行 maven-antrun-plugin 的 run 目标，具体的任务在 <target></target> 元素中定义。
清单 2. 插件
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-antrun-plugin</artifactId>
<version>1.6</version>
<executions>
<execution>
<id>version</id>
   <phase>validate</phase>
   <configuration>
<target>
   具体任务
</target>
</configuration>
<goals>
<goal>  run  </goal>
</goals>
</execution>
</executions>
</plugin>
</plugins>
##  设置代理： 

当个人所在的网络无法访问公共的 Maven 仓库时，可以在 settings.xml 中设置代理服务器。打开 ~\.m2\settings.xml，如果没有则复制 $Maven_HOME/conf/settings.xml 到此路径下，加入清单 3 中的代码：
清单 3. 代理
<proxies>
  <proxy>
     <active>true</active>
     <protocol>http</protocol>
     <host> 代理地址 </host>
     <port>8080</port>
     <username> 用户名 </username>
     <password> 密码 </password>
   </proxy>
</proxies>
#  依赖、聚合和继承 

   ##  依赖 

我们项目中依赖的 Jar 包可以通过依赖的方式引入，通过在 dependencies 元素下添加 dependency 子元素，可以声明一个或多个依赖。通过控制依赖的范围，可以指定该依赖在什么阶段有效。Maven 的几种依赖范围：
表 2. 依赖范围
名称 有效范围
compile 编译，测试，运行。默认的依赖范围。
test 测试，如 Junit。
runtime 运行，如 JDBC。
provided 编译，测试，如 ServletAPI。
system 编译，测试，依赖于系统变量。
清单 4 中表示引入对 Junit 的依赖 , 这个依赖关系产生作用的阶段是 <scope>test</scope>。
清单 4. 依赖
   <dependency>  
     <groupId> </groupId>  
     <artifactId> </artifactId>  
     <version> </version>  
     <optional>true<optional>  
   </dependency>
依赖是具有传递性的，例如 Project A 依赖于 Project B，B 依赖于 C，那么 B 对 C 的依赖关系也会传递给 A，如果我们不需要这种传递性依赖，也可以用 <optional> 去除这种依赖的传递，如清单 5。
清单 5. 选择性依赖
<dependency>
<groupId>commons-logging</groupId>
<artifactId>commons-logging</artifactId>
<version>1.1.1</version>
<optional>true<optional>
</dependency>
假设第三方的 jar 包中没有使用 <optional> 来去除某些依赖的传递性，那么可以在当前的 POM 文件中使用 <exclusions> 元素声明排除依赖，exclusions 可以包含一个或者多个 exclusion 子元素，因此可以排除一个或者多个传递性依赖。如清单 6。
清单 6. 排除依赖
   <dependency>    
        <groupId>org.springframework</groupId>  
        <artifactId>spring-core</artifactId>  
        <exclusions>  
              <exclusion>      
                   <groupId>commons-logging</groupId>          
                   <artifactId>commons-logging</artifactId>  
              </exclusion>  
        </exclusions>  
   </dependency>
   聚合
现实中一个项目往往是由多个 project 构成的，在进行构建时，我们当然不想针对多个 project 分别执行多次构建命令，这样极容易产生遗漏也会大大降低效率。Maven 的聚合功能可以通过一个父模块将所有的要构建模块整合起来，将父模块的打包类型声明为 POM，通过 <modules> 将各模块集中到父 POM 中。如清单 7，其中 <module></module> 中间的内容为子模块工程名的相对路径。
清单 7. 聚合
 <modules>    
<module>../com.dugeng.project1</module>
<module>../com.dugeng.project2</module>
 </modules>
父类型的模块，不需要有源代码和资源文件，也就是说，没有 src/main/java 和 src/test/java 目录。Maven 会首先解析聚合模块的 POM 文件，分析要构建的模块，并通过各模块的依赖关系计算出模块的执行顺序，根据这个潜在的关系依次构建模块。将各子模块聚合到父模块中后，我们就可以对父模块进行一次构建命令来完成全部模块的构建。
   继承
在面向对象的编程中我们学会了继承的概念，继承是可重用行即消除重复编码的行为。Maven 中继承的用意和面向对象编程中是一致的。与聚合的实现类似，我们通过构建父模块将子模块共用的依赖，插件等进行统一声明，在聚合和继承同时使用时，我们可以用同一个父模块来完成这两个功能。
例如将 com.dugeng.parent 这个模块声明为 project1 和 project2 的父模块，那么我们在 project1 和 2 中用如下代码声明父子关系，如清单 8：
清单 8. 继承
<parent>
 <groupId>com.dugeng.mavenproject</groupId>
 <artifactId>com.dugeng.parent</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <relativePath>../com.dugeng.parent/pom.xml</relativePath>
</parent>
由于父模块只是用来声明一些可共用的配置和插件信息，所以它也像聚合模块一样只需要包括一个 POM 文件，其它的项目文件如 src/main/java 是不需要的。
聚合和继承存在一些共性和潜在的联系，在实际的应用中，经常将聚合模块的父模块和继承的父模块定义为同一个。
并不是所有的 POM 元素都可以被继承

##  可选依赖 

有时候我们不想让依赖传递，那么可配置该依赖为可选依赖，` 将元素optional设置为true即可 `,例如：
<dependency>  
     <groupId>commons-logging</groupId>  
     <artifactId>commons-logging</artifactId>  
     <version>1.1.1</version>  
     <optional>true<optional>  
   </dependency>  

那么依赖该项目的另以项目将不会得到此依赖的传递
##  5. 排除依赖 

      当我们引入第三方jar包的时候，难免会引入传递性依赖，有些时候这是好事，然而有些时候我们不需要其中的一些传递性依赖

比如上例中的项目，我们不想引入传递性依赖commons-logging，我们可以使用exclusions元素声明排除依赖，exclusions可以包含一个或者多个exclusion子元素，因此可以排除一个或者多个传递性依赖。需要注意的是，声明exclusions的时候只需要groupId和artifactId，而不需要version元素，这是因为只需要groupId和artifactId就能唯一定位依赖图中的某个依赖。换句话说，Maven解析后的依赖中，不可能出现groupId和artifactId相同，但是version不同的两个依赖。
如下是一个排除依赖的例子：
   <dependency>    
        <groupId>org.springframework</groupId>  
        <artifactId>spring-core</artifactId>  
        <version>2.5.6</version>  
        <exclusions>  
              <exclusion>      
                   <groupId>commons-logging</groupId>          
                   <artifactId>commons-logging</artifactId>  
              </exclusion>  
        </exclusions>  
   </dependency>  

#  Maven 属性 

在 POM 文件中常常需要引用已定义的属性以降低代码的冗余，提高代码的可重用性，这样不仅能降低代码升级的工作量也能提高代码的正确率。有些属性是用户自定义的，有些属性是可以直接引用的已定义变量。
Maven 的可用属性类型可分为 5 种，它们分别是：
   内置属性。这种属性跟 Maven Project 自身有关，比如要引入当前 Project 的版本信 息，那么只需要在使用的位置引用 ${version} 就行了。
   Setting 属性。上文中已经提到 Maven 自身有一个 settings.xml 配置文件，它里面含有包括仓库，代理服务器等一些配置信息，利用 ${settings.somename} 就可以得到文件里相应元素的值。
   POM 属性。这种属性对应 POM 文件中对应元素的值，例如 ${project.groupId} 对应了 <groupId></groupId> 中的值，${project.artifactId} 对应了 <artifactId> </ artifactId > 中的值。
   系统环境变量。可以使用 env.${name} 来获得相应 name 对应的环境变量的值，例如 ${env.JAVA_HOME} 得到的就是 JAVA_HOME 的环境变量值。
   用户自定义变量。这种类型的变量是使用最频繁和广泛的变量，完全由用户自己定义。在 POM 文件中加入 <properties> 元素并将自定义属性作为其子元素。格式如清单 9。
清单 9. 自定义属性
<properties>
 <path>../../sourcecode</path>
</properties>


#  Maven 生命周期 

在上一篇文章中，我们用的第二个命令是：mvn package。这里的 package 是一个maven的生命周期阶段 (lifecycle phase )。生命周期指项目的构建过程，它包含了一系列的有序的阶段 (phase)，而一个阶段就是构建过程中的一个步骤。
那么生命周期阶段和上面说的插件目标之间是什么关系呢？插件目标可以绑定到生命周期阶段上。一个生命周期阶段可以绑定多个插件目标。当 maven 在构建过程中逐步的通过每个阶段时，会执行该阶段所有的插件目标。
maven 能支持不同的生命周期，但是最常用的是默认的Maven生命周期 (default Maven lifecycle )。如果你没有对它进行任何的插件配置或者定制的话，那么上面的命令 mvn package 会依次执行默认生命周期中直到包括 package 阶段前的所有阶段的插件目标：

##  Maven生命周期phases阶段 

   validate: validate the project is correct and all necessary information is available
   compile: compile the source code of the project
   test: test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed

   package: take the compiled code and package it in its distributable format, such as a JAR.
   integration-test: process and deploy the package if necessary into an environment where integration tests can be run
   verify: run any checks to verify the package is valid and meets quality criteria
   install: install the package into the local repository, for use as a dependency in other projects locally
   deploy: done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.
There are two other Maven lifecycles of note beyond the default list above. They are
   clean: cleans up artifacts created by prior builds
   site: generates site documentation for this project

##  依赖的范围： 

provided: 已提供依赖范围。使用此依赖范围的Maven依赖，对于编译和测试classpath有效，但在运行时无效。典型的例子是servlet-api，编译和测试项目的时候需要该依赖，但在运行项目的时候，由于容器已经提供，就不需要Maven重复地引入一遍。
runtime: 运行时依赖范围。使用此依赖范围的Maven依赖，对于测试和运行classpath有效，但在编译主代码时无效。典型的例子是JDBC驱动实现，项目主代码的编译只需要JDK提供的JDBC接口，只有在执行测试或者运行项目的时候才需要实现上述接口的具体JDBC驱动。
system: 系统依赖范围。该依赖与三种classpath的关系，和provided依赖范围完全一致。但是，使用system范围依赖时必须通过systemPath元素显式地指定依赖文件的路径。由于此类依赖不是通过Maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植，因此应该谨慎使用。systemPath元素可以引用环境变量
   依赖范围与maven编译和执行过程中的三种classpath相关：编译，测试，执行classpath
    依赖范围种类：
    compile：编译依赖范围，使用此范围的依赖对于编译测试运行均有效。默认的依赖范围
    test:测试依赖范围。只对测试有效。例如junit
    provided:对于编译测试有效，对于运行无效。如：servlet-api
    runtime:运行时依赖范围。对于测试和运行均有效。但在编译时无效。典型的就是JDBC驱动实现
    system:易造成不可移植谨慎使用。
    import：对三种classpath均无影响。
###  goal,phases,lifeclycle的区别： 

maven对构建(build)的过程进行了抽象和定义，这个过程被称为构建的生命周期(lifecycle)。生命周期(lifecycle)由多个阶段(phase)组成,每个阶段(phase)会挂接一到多个goal。
goal是maven里定义任务的最小单元，相当于ant里的target。
因此，goal其实是由存在于Maven的repository中的plugin提供的一个个小的功能程序
它是Maven的lifecycle以及phase的基本组成元素。同时，我们也可以通过将各种各样的goal加入到Maven的phase中，从而根据自己的实际需求，灵活实现各种定制功能。

#  构建： 

以phase为目标构建
以phase为目标进行构建是最常见的，如我们平时经常执行的mvn compile,mvn test,mvn package...等等,compile,test,package都是maven生命周期(lifecycle)里的phase,通过mvn命令，你可以指定一次构建执行到那一个阶段，在执行过程中，所有经历的执行阶段(phase)上绑定的goal都将得到执行。例如，对于一个jar包应用，当执行mvn package命令时，maven从validate阶段一个阶段一个阶段的执行，在执行到compile阶段时，compiler插件的compile goal会被执行，因为这个goal是绑定在compile阶段(phase)上的。
以goal为目标构建
虽然以phase为目标的构建最常见，但是有时候我们会发现，一些插件的goal并不适合绑定到任何阶段(phase)上，或者是，这些goal往往是单独执行，不需要同某个阶段(phase)绑定在一起，比如hibernate插件的导入\导出goal多数情况下是根据需要要手动执行的(当然，也可以绑定到某个阶段上，比如进行单元测试时，可考虑将其绑定到test阶段上)。
Maven拥有三套相互独立的生命周期，它们分别为clean，default和site。 
每个生命周期包含一些阶段，这些阶段是有顺序的，并且后面的阶段依赖于前面的阶段，用户和Maven最直接的交互方式就是调用这些生命周期阶段。

每个阶段对应的内置插件目标goal
Clean Lifecycle Bindings
clean clean:clean

Default Lifecycle Bindings - Packaging ejb / ejb3 / jar / par / rar / war
process-resources resources:resources
compile compiler:compile
process-test-resources resources:testResources
test-compile compiler:testCompile
test surefire:test
package ejb:ejb or ejb3:ejb3 or jar:jar or par:par or rar:rar or war:war
install install:install
deploy deploy:deploy

Default Lifecycle Bindings - Packaging ear
generate-resources ear:generate-application-xml
process-resources resources:resources
package ear:ear
install install:install
deploy deploy:deploy

Default Lifecycle Bindings - Packaging maven-plugin
generate-resources plugin:descriptor
process-resources resources:resources
compile compiler:compile
process-test-resources resources:testResources
test-compile compiler:testCompile
test surefire:test
package jar:jar and plugin:addPluginArtifactMetadata
install install:install
deploy deploy:deploy

Default Lifecycle Bindings - Packaging pom
package site:attach-descriptor
install install:install
deploy deploy:deploy

Site Lifecycle Bindings
site site:site
site-deploy site:deploy

#  Maven 库 

当第一次运行 maven 命令的时候，你需要 Internet 连接，因为它要从网上下载一些文件。那么它从哪里下载呢？它是从 maven 默认的远程库(http://repo1.maven.org/maven2) 下载的。这个远程库有 maven 的核心插件和可供下载的 jar 文件。
但是不是所有的 jar 文件都是可以从默认的远程库下载的，比如说我们自己开发的项目。这个时候，有两个选择：要么在公司内部设置定制库，要么手动下载和安装所需的jar文件到本地库。
本地库是指 maven 下载了插件或者 jar 文件后存放在本地机器上的拷贝。在 Linux 上，它的位置在 ~/.m2/repository，在 Windows XP 上，在 C:\Documents and Settings\username\.m2\repository ，在 Windows 7 上，在 C:\Users\username\.m2\repository。当 maven 查找需要的 jar 文件时，它会先在本地库中寻找，只有在找不到的情况下，才会去远程库中找。
运行下面的命令能把我们的 helloworld 项目安装到本地库：
    $mvn install
一旦一个项目被安装到了本地库后，你别的项目就可以通过 maven 坐标和这个项目建立依赖关系

#  补充说明几点技巧： 
##  指定java版本 
```

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

```
##  打包可执行的 JAR 文件 
###  方式一 
使用 Maven 构建一个 JAR 文件比较容易：只要定义项目包装为 “jar”，然后执行包装生命周期阶段即可。但是定义一个可执行 JAR 文件却比较麻烦。采取以下步骤可以更高效：
   在您定义可执行类的 JAR 的 MANIFEST.MF 文件中定义一个 main 类。（MANIFEST.MF 是包装您的应用程序时 Maven 生成的。）
   找到您项目依赖的所有库。
   在您的 MANIFEST.MF 文件中包含那些库，便于您的应用程序找到它们。
您可以手工进行这些操作，或者要想更高效，您可以使用两个 Maven 插件帮助您完成：maven-jar-plugin 和 maven-dependency-plugin。
maven-jar-plugin
maven-jar-plugin 可以做很多事情，但在这里，我们只对使用它来修改默认 MANIFEST.MF 文件的内容感兴趣。在您的 POM 文件的插件部分添加清单 1 所示代码：
清单 1. 使用 maven-jar-plugin 修改 MANIFEST.MF
  ```

         <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-jar-plugin</artifactId>
               <configuration>
                   <archive>
                       <manifest>
                           <addClasspath>true</addClasspath>
                           <classpathPrefix>lib/</classpathPrefix>
                           <mainClass>com.mypackage.MyClass</mainClass>
                       </manifest>
                   </archive>
               </configuration>
           </plugin>

```

所有 Maven 插件通过一个 <configuration> 元素公布了其配置，在本例中，maven-jar-plugin 修改它的 archive 属性，特别是存档文件的 manifest 属性，它控制 MANIFEST.MF 文件的内容。包括 3 个元素：

   addClassPath：将该元素设置为 true 告知 maven-jar-plugin 添加一个 Class-Path 元素到 MANIFEST.MF 文件，以及在 Class-Path 元素中包括所有依赖项。
   classpathPrefix：如果您计划在同一目录下包含有您的所有依赖项，作为您将构建的 JAR，那么您可以忽略它；否则使用 classpathPrefix 来指定所有依赖 JAR 文件的前缀。在清单 1 中，classpathPrefix 指出，相对存档文件，所有的依赖项应该位于 “lib” 文件夹。
   mainClass：当用户使用 lib 命令执行 JAR 文件时，使用该元素定义将要执行的类名。
maven-dependency-plugin
当您使用这 3 个元素配置好了 MANIFEST.MF 文件之后，下一步是将所有的依赖项复制到 lib 文件夹。为此，使用 maven-dependency-plugin，如清单 2 所示：
清单 2. 使用 maven-dependency-plugin 将依赖项复制到库
```

	  <!-- 拷贝依赖的jar包到lib目录 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-dependency-plugin</artifactId>  
                <executions>  
                    <execution>  
                        <id>copy</id>  
                        <phase>package</phase>  
                        <goals>  
                            <goal>copy-dependencies</goal>  
                        </goals>  
                        <configuration>  
                            <outputDirectory>  
                                ${project.build.directory}/lib  
                            </outputDirectory>  
                        </configuration>  
                    </execution>  
                </executions>  
            </plugin> 

``` 
maven-dependency-plugin 有一个 copy-dependencies，目标是将您的依赖项复制到您所选择的目录。本例中，我将依赖项复制到 build 目录下的 lib 目录（project-home/target/lib）。
然后运行` mvn clean package `
**` 但是这还没有完，这两步没有将依赖拷贝到jar中。所以需要将jar，lib目录拷贝到同一文件夹下。不过这种方式也是比较推荐的 `**
###  方式二 
```

<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-assembly-plugin</artifactId>
				<version>2.3</version>
				<configuration>
					<appendAssemblyId>false</appendAssemblyId>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
							<mainClass>com.css.licencegen.App</mainClass>
						</manifest>
					</archive>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>assembly</goal>
						</goals>
					</execution>
				</executions>
			</plugin>

```
然后运行运行` mvn assembly:assembly `即可得到可执行包含依赖jar包
3.如果不希望依赖的JAR包变成CLASS的话,可以修改ASSEMBLY插件.
 3.1 找到assembly在本地的地址,一般是c:/users/${your_login_name}/.m2/\org\apache\maven\plugins\maven-assembly-plugin\2.4
3.2 用WINZIP或解压工具打开此目录下的maven-assembly-plugin-2.4.jar, 找到assemblies\jar-with-dependencies.xml
3.3 把里面的UNPACK改成FALSE即可
这种方式能成功打包包含依赖可执行jar。但是若是生成exe时会有些问题。


##  2.依赖项树 


Maven 一个最有用的功能是它支持依赖项管理：您只需要定义您应用程序依赖的库，Maven 找到它们、下载它们、然后使用它们编译您的代码。
必要时，您需要知道具体依赖项的来源 — 这样您就可以找到同一个 JAR 文件的不同版本的区别和矛盾。这种情况下，您将需要防止将一个版本的 JAR 文件包含在您的构建中，但是首先您需要定位保存 JAR 的依赖项。
一旦您知道下列命令，那么定位依赖项往往是相当容易的：
mvn dependency:tree
dependency:tree 参数显示您的所有直接依赖项，然后显示所有子依赖项（以及它们的子依赖项，等等）。例如，清单 5 节选自我的一个依赖项所需要的客户端库：
清单 5. Maven 依赖项树
[INFO] ------------------------------------------------------------------------
[INFO] Building Client library for communicating with the LDE
[INFO]    task-segment: [dependency:tree]
[INFO] ------------------------------------------------------------------------
[INFO] [dependency:tree {execution: default-cli}]
[INFO] com.lmt.pos:sis-client:jar:2.1.14
[INFO] +- org.codehaus.woodstox:woodstox-core-lgpl:jar:4.0.7:compile
[INFO] |  \- org.codehaus.woodstox:stax2-api:jar:3.0.1:compile
[INFO] +- org.easymock:easymockclassextension:jar:2.5.2:test
[INFO] |  +- cglib:cglib-nodep:jar:2.2:test
[INFO] |  \- org.objenesis:objenesis:jar:1.2:test

在 清单 5 中您可以看到 sis-client 项目需要 woodstox-core-lgpl 和 easymockclassextension 库。easymockclassextension 库反过来需要 cglib-nodep 库和 objenesis 库。如果我的 objenesis 出了问题，比如出现两个版本，1.2 和 1.3，那么这个依赖项树可能会向我显示，1.2 工件是直接由 easymockclassextension 库导入的。

dependency:tree 参数为我节省了很多调试时间，我希望对您也同样有帮助。
##  3.使用配置文件 


多数重大项目至少有一个核心环境，由开发相关的任务、质量保证（QA）、集成和生产组成。管理所有这些环境的挑战是配置您的构建，这必须连接到正确的数据库中，执行正确的脚本集、并为每个环境部署正确的工件。使用 Maven 配置文件让您完成这些任务，而无需为每个环境分别建立明确指令。

关键在于环境配置文件和面向任务的配置文件的合并。每个环境配置文件定义其特定的位置、脚本和服务器。因此，在我的 pox.xml 文件中，我将定义面向任务的配置文件 “deploywar”，如清单 6 所示：
清单 6. 部署配置文件

   <profiles>
       <profile>
           <id>deploywar</id>
           <build>
               <plugins>
                   <plugin>
                       <groupId>net.fpic</groupId>
                       <artifactId>tomcat-deployer-plugin</artifactId>
                       <version>1.0-SNAPSHOT</version>
                       <executions>
                           <execution>
                               <id>pos</id>
                               <phase>install</phase>
                               <goals>
                                   <goal>deploy</goal>
                               </goals>
                               <configuration>
                                   <host>${deploymentManagerRestHost}</host>
                                   <port>${deploymentManagerRestPort}</port>
                                   <username>${deploymentManagerRestUsername}</username>
                                   <password>${deploymentManagerRestPassword}</password>
                                   <artifactSource>
                                     address/target/addressservice.war
                                   </artifactSource>
                               ....

这个配置文件（通过 ID “deploywar” 区别）执行 tomcat-deployer-plugin，被配置来连接一个特定主机和端口，以及指定用户名和密码证书。所有这些信息使用变量来定义，比如 ${deploymentmanagerRestHost}。这些变量在我的 profiles.xml 文件中定义，如清单 7 所示：
清单 7. profiles.xml

       <!-- Defines the development deployment information -->
       <profile>
           <id>dev</id>
           <activation>
               <property>
                   <name>env</name>
                   <value>dev</value>
               </property>
           </activation>
           <properties>
               <deploymentManagerRestHost>10.50.50.52</deploymentManagerRestHost>
               <deploymentManagerRestPort>58090</deploymentManagerRestPort>
               <deploymentManagerRestUsername>myusername</deploymentManagerRestUsername>
               <deploymentManagerRestPassword>mypassword</deploymentManagerRestPassword>
           </properties>
       </profile>

       <!-- Defines the QA deployment information -->
       <profile>
           <id>qa</id>
           <activation>
               <property>
                   <name>env</name>
                   <value>qa</value>
               </property>
           </activation>
           <properties>
               <deploymentManagerRestHost>10.50.50.50</deploymentManagerRestHost>
               <deploymentManagerRestPort>58090</deploymentManagerRestPort>
               <deploymentManagerRestUsername>
                 myotherusername
               </deploymentManagerRestUsername>
               <deploymentManagerRestPassword>
                 myotherpassword
               </deploymentManagerRestPassword>
           </properties>
       </profile>
部署 Maven 配置文件
在 清单 7 的 profiles.xml 文件中，我定义了两个配置文件，并根据 env （环境）属性的值激活它们。如果 env 属性被设置为 dev，则使用开发部署信息。如果 env 属性被设置为 qa，那么将使用 QA 部署信息，等等。
这是部署文件的命令：
mvn -Pdeploywar -Denv=dev clean install
-Pdeploywar 标记通知要明确包含 deploywar 配置文件。-Denv=dev 语句创建一个名为 env 的系统属性，并将其值设为 dev，这激活了开发配置。传递 -Denv=qa 将激活 QA 配置。
##  4.goal目标与phases阶段 

Maven 中的 goal 类似 Ant 中的 target 。两者都包含实现 goal（或 target）时会执行的任务。要在命令行中实现特定的 goal，可输入 maven <goal> 。
要列出所有已定义的 goal，可使用 maven -g 。表 4 列出了常用的 goal。
表 4. 常用的 goal
java:compile 编译所有 Java 源代码。
jar 创建已编译的源代码的 JAR 文件。
jar:install 将已创建的 JAR 文件发布到本地资源库，使得其它项目可访问该 JAR 文件。
site 创建项目站点文档。缺省站点文档包含关于项目的有用信息，如包／类相关性、编码风格一致性、源代码交叉引用、单元测试结果或 Javadoc。要生成的报告列表是可定制的。
site:deploy 部署生成的站点文档。
Maven 的 goal 是可扩展和可重用的。知道了这一点后，在编写自己的 goal 之前，可先在 Maven 站点上或 ${MAVEN_HOME}/plugins 中查看 Maven 插件列表。另一个关于免费 Maven 插件的较佳资源是 SourceForge 上的 Maven 插件项目。



参考：http://www.ibm.com/developerworks/cn/java/j-lo-maven/
https://www.ibm.com/developerworks/cn/java/j-5things13/
http://www.ibm.com/developerworks/cn/java/j-maven/
http://www.oracle.com/technetwork/cn/community/java/apache-maven-getting-started-2-405568-zhs.html
http://blog.csdn.net/bluishglc/article/details/6632280
https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html
http://maven.apache.org/ref/3.2.1/maven-core/lifecycles.html
关于生命周期等其他maven教程更多参考：http://tangyanbo.iteye.com/blog/1503890