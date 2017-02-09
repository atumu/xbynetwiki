title: maven打包技巧 

#  maven打包技巧 
参考：
http://www.infoq.com/cn/news/2011/06/xxb-maven-9-package/
http://blogcsdn.net/johnnywww/article/details/7964326

示例pom.xml :可以通过下载maven的源码包里的pom.xml中看到
“打包“这个词听起来比较土，比较正式的说法应该是”构建项目软件包“，具体说就是将项目中的各种文件，比如源代码、编译生成的字节码、配置文件、文档，按照规范的格式生成归档，最常见的当然就是JAR包和WAR包了，复杂点的例子是Maven官方下载页面的分发包，它有自定义的格式，方便用户直接解压后就在命令行使用。
本文就介绍一些常用的打包案例以及相关的实现方式，除了前面提到的一些包以外，你还能看到如何生成源码包、Javadoc包、以及从命令行可直接运行的CLI包。
##  打包资源文件 
```

<build>  
    <resources>  
        <resource>  
       
            <!-- 控制资源文件的拷贝 -->  
            <resource>  
                <directory>src/main/resources</directory>  
                <targetPath>${project.build.directory}</targetPath>  
            </resource>  
    
          <!--
            <targetPath>${project.build.directory}/classes</targetPath>  
            <directory>src/main/resources</directory>  
            <filtering>true</filtering>  
            <includes>  
                <include>**/*.xml</include>  
            </includes>  
	-->
        </resource>  
    </resources>  
  ...

```
##  Packaging的含义 
任何一个Maven项目都需要定义**POM元素**` packaging `（如果不写则默认值为jar）。顾名思义，该元素决定了项目的打包方式。实际的情形中，如果你不声明该元素，Maven会帮你生成一个JAR包；
如果你定义该元素的值为war，那你会得到一个WAR包；如果定义其值为POM（比如是一个父模块），那什么包都不会生成。
除此之外，Maven默认还支持一些其他的流行打包格式，例如ejb3和ear。你不需要了解具体的打包细节，你所需要做的就是告诉Maven，”我是个什么类型的项目“，这就是约定优于配置的力量。

对应于同样的` package `**生命周期阶段**，Maven为jar项目调用了` maven-jar-plugin `，为war项目调用了` maven-war-plugin `，
换言之，**packaging直接影响Maven的构建生命周期**。了解这一点非常重要，特别是当你需要自定义打包行为的时候，你就必须知道去配置哪个插件。一个常见的例子就是在打包war项目的时候排除某些web资源文件，这时就应该配置maven-war-plugin如下：
```

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.1.1</version>
    <configuration>
      <webResources>
        <resource>
          <directory>src/main/webapp</directory>
          <excludes>
            <exclude>**/*.jpg</exclude>
          </excludes>
        </resource>
      </webResources>
    </configuration>
  </plugin>


```
##  源码包和Javadoc包 
一个Maven项目只生成一个主构件，当需要生成其他附属构件的时候，就需要用上` classifier `。源码包和Javadoc包就是附属构件的极佳例子。
```

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.1.2</version>
    <executions>
      <execution>
        <id>attach-sources</id>
        <phase>verify</phase>
        <goals>
          <goal>jar-no-fork</goal>
        </goals>
      </execution>
    </executions>
  </plugin>


```
类似的，生成Javadoc包只需要配置插件如下：
```

  <plugin>          
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.7</version>
    <executions>
      <execution>
        <id>attach-javadocs</id>
          <goals>
            <goal>jar</goal>
          </goals>
      </execution>
    </executions>
  </plugin>

```
##  可执行CLI包 

除了前面提到了常规JAR包、WAR包，源码包和Javadoc包，另一种常被用到的包是在命令行可直接运行的CLI（Command Line）包。默认Maven生成的JAR包只包含了编译生成的.class文件和项目资源文件，
而要得到一个可以直接在命令行通过java命令运行的JAR文件，还要满足两个条件：
  * JAR包中的/META-INF/MANIFEST.MF元数据文件必须包含Main-Class信息。 
  * 项目所有的依赖都必须在Classpath中。

**Maven有好几个插件能帮助用户完成上述任务，**不过用起来` 最方便的还是maven-shade-plugin `，它可以让用户配置Main-Class的值，然后在打包的时候将值填入/META-INF/MANIFEST.MF文件。关于项目的依赖，它很聪明地将依赖JAR文件全部解压后，再将得到的.class文件连同当前项目的.class文件一起合并到最终的CLI包中，这样，在执行CLI JAR文件的时候，所有需要的类就都在Classpath中了。下面是一个配置样例：
```

  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>1.4</version>
    <executions>
      <execution>
        <phase>package</phase>
        <goals>
          <goal>shade</goal>
        </goals>
        <configuration>
          <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
              <mainClass>com.juvenxu.mavenbook.HelloWorldCli</mainClass>
            </transformer>
          <!--  
             <transformer   implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                 <resource>applicationContext.xml</resource>  
             </transformer>
	  -->
          </transformers>
	
        </configuration>
      </execution>
    </executions>
  </plugin>


```
上述例子中的，我的Main-Class是com.juvenxu.mavenbook.HelloWorldCli，
构建完成后，对应于一个常规的hello-world-1.0.jar文件，我还得到了一个**hello-world-1.0-cli.jar**文件。细心的读者可能已经注意到了，这里用的是cli这个classifier。
最后，我可以通过java -jar hello-world-1.0-cli.jar命令运行程序。
##  打包自定义格式包，分发包 

实际的软件项目常常会有更复杂的打包需求，例如我们可能需要为客户提供一份产品的分发包，这个包不仅仅包含项目的字节码文件，还得包含依赖以及相关脚本文件以方便客户解压后就能运行，此外分发包还得包含一些必要的文档。这时项目的源码目录结构大致是这样的：
pom.xml
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
NOTICE.txt	Notices and attributions required by libraries that the project depends on
README.txt	
Project's readme
除了基本的pom.xml和一般Maven目录之外，这里还有一个src/main/scripts/目录，该目录会包含一些脚本文件如run.sh和run.bat，
` src/main/assembly/会包含一个assembly.xml `，**这是打包的描述文件**，稍后介绍，最后的README.txt是份简单的文档。
我们希望最终生成一个zip格式的分发包，它包含如下的一个结构：
bin/
lib/
README.txt
其中bin/目录包含了可执行脚本run.sh和run.bat，lib/目录包含了项目JAR包和所有依赖JAR，README.txt就是前面提到的文档。
描述清楚需求后，我们就要搬出
` Maven最强大的打包插件：maven-assembly-plugin `。关于此插件可以参考官方文档：http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html
它支持各种打包文件格式，包括zip、tar.gz、tar.bz2等等，通过一个**打包描述文件（该例中是src/main/assembly/assembly.xml）**，它能够帮助用户选择具体打包哪些文件集合、依赖、模块、和甚至本地仓库文件，每个项的具体打包路径用户也能自由控制。
如下就是对应上述需求的打包描述文件**src/main/assembly/assembly.xml：**
```

<assembly>
  <id>bin</id>
  <formats>
    <format>zip</format>
  </formats>
  <dependencySets>
    <dependencySet>
      <useProjectArtifact>true</useProjectArtifact>
      <outputDirectory>lib</outputDirectory>
    </dependencySet>
  </dependencySets>
  <fileSets>
    <fileSet>
      <outputDirectory>/</outputDirectory>
      <includes>
        <include>README.txt</include>
      </includes>
    </fileSet>
    <fileSet>
      <directory>src/main/scripts</directory>
      <outputDirectory>/bin</outputDirectory>
      <includes>
        <include>run.sh</include>
        <include>run.bat</include>
      </includes>
    </fileSet>
  </fileSets>
</assembly>

```
首先这个` assembly.xml `文件的**id对应了其最终生成文件的classifier。** 
**其次formats定义打包生成的文件格式，这里是zip**。因此结合id我们会得到一个名为` hello-world-1.0-bin.zip `的文件。（假设artifactId为hello-world，version为1.0） 
` dependencySets `用来定义选择依赖并定义最终打包到什么目录，这里我们声明的一个` depenencySet `默认包含所有所有依赖，而` useProjectArtifact `表示**将项目本身生成的构件也包含在内**，最终打包至输出包内的lib路径下（由` outputDirectory `指定）。 
` fileSets `允许用户通过文件或目录的粒度来控制打包。这里的第一个` fileSet `打包README.txt文件至包的根目录下，第二个fileSet则将src/main/scripts下的run.sh和run.bat文件打包至输出包的bin目录下。 

最后，我们需要配置` maven-assembly-plugin `使用打包描述文件，并绑定生命周期阶段使其自动执行打包操作：
```

 <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.4</version>
    <configuration>
      <descriptors>
        <descriptor>src/main/assembly/assembly.xml</descriptor> <!--指定assembly.xml配置文件路径 -->
      </descriptors>
      <appendAssemblyId>false</appendAssemblyId> <!-- 设为 FALSE, 防止 WAR 包名加入 assembly.xml 中的 ID -->
    </configuration>
    <executions>
      <execution>
        <id>make-assembly</id>  <!-- ID 标识，命名随意 -->
        <phase>package</phase><!-- 绑定到 PACKAGE 生命周期阶段 -->
        <goals>
          <goal>single</goal> <!-- 在 PACKAGE 生命周期阶段仅执行一次 -->
        </goals>
      </execution>
    </executions>
  </plugin>

```
运行` mvn clean package `之后，我们就能在target/目录下得到名为hello-world-1.0-bin.zip的分发包了。

**assembly.xml使用示列**
环境配置：
```

在你的 pom.xml 文件中添加如下配置： 
<profiles>
  <profile> <!-- 可以通过 -P ID 来激活 -->
    <id>PROD</id> <!-- ID 标识符 -->
    <properties>
      <env>PROD</env> <!-- properties 定义 key-value, 这里 key 是 env, value 是 PROD --> <!--注意这个env与profiles.active同理只不过去值的方式不同，前者通过${env}后者通过${profiles.active} -->
    </properties>
    <activation>
      <activeByDefault>true</activeByDefault> <!-- 默认激活 -->
    </activation>
  </profile>
  <profile> <!-- 可以通过 -P ID 来激活 -->
    <id>TEST</id> <!-- ID 标识符 -->
    <properties>
      <env>TEST</env> <!-- properties 定义 key-value, 这里 key 是 env, value 是 TEST -->
    </properties>
  </profile>
</profiles>

```
assembly插件
```

<build>
  <plugins>
    <plugin>
      <artifactId>maven-assembly-plugin</artifactId> <!-- 官网给出的配置，没有配置 groupId，这里也不配置 -->
      <version>2.4</version>
      <executions>
        <execution>
          <id>make-assembly</id> <!-- ID 标识，命名随意 -->
          <phase>package</phase> <!-- 绑定到 PACKAGE 生命周期阶段 -->
          <goals>
            <goal>single</goal>  <!-- 在 PACKAGE 生命周期阶段仅执行一次 -->
          </goals>
        </execution>
      </executions>
      <configuration>
        <descriptors>
          <descriptor>assembly.xml</descriptor> <!-- 自定义打包的配置文件 -->
        </descriptors>
        <appendAssemblyId>false</appendAssemblyId> <!-- 设为 FALSE, 防止 WAR 包名加入 assembly.xml 中的 ID -->
      </configuration>
    </plugin>
  </plugins>
</build>

```
assembly.xml
```

<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3 
  http://maven.apache.org/xsd/assembly-1.1.3.xsd">
  <!-- ID 标识，命名随意 -->
  <id>${project.artifactId}-assembly-${project.version}</id>
  <!-- 默认为 TRUE, 设为 FALSE, 防止将 ${project.finalName} 作为根目录打进 WAR 包 -->
  <!-- TRUE  结构: ${project.finalName}.war/${project.finalName}/WEB-INF -->
  <!-- FALSE 结构: ${project.finalName}.war/WEB-INF -->
  <includeBaseDirectory>false</includeBaseDirectory>
  <!-- 设置为 WAR 包格式 -->
  <formats>
    <format>war</format>
  </formats>
  <fileSets>
    <!-- 将 target/classes 下的文件输出到 WEB-INF/classes, 同时排除 target/classes/conf/*.properties -->
    <fileSet>
      <directory>${project.build.outputDirectory}</directory> <!-- target/classes -->
      <outputDirectory>WEB-INF/classes</outputDirectory>
      <excludes>
        <exclude>**/conf/*.properties</exclude>
      </excludes>
    </fileSet>
    <!-- 将 env/${env}/conf 下的文件输出到 WEB-INF/classes/conf, 实现 -P 不同的参数打包出不同的配置 -->
    <!-- ${env} 的值由 -P 的参数传递进来, 如：-PTEST, 那么, ${env} 的值就是 TEST -->
    <fileSet>
      <directory>${project.basedir}/env/${env}/conf</directory>
      <outputDirectory>WEB-INF/classes/conf</outputDirectory>
    </fileSet>
    <!-- 将 webapp 下的文件输出到 WAR 包 -->
    <fileSet>
      <directory>${project.basedir}/src/main/webapp</directory>
      <outputDirectory>/</outputDirectory>
    </fileSet>
  </fileSets>
  <!-- 将项目依赖的JAR包输出到 WEB-INF/lib -->
  <dependencySets>
    <dependencySet>
      <outputDirectory>WEB-INF/lib</outputDirectory>
    </dependencySet>
  </dependencySets>
</assembly>

```
打包出适应各个环境的 WAR 包
选中项目右键 --> Run As --> Maven build... --> Goals 栏输入`  -PTEST ` clean package 。 

参考：http://www.blogjava.net/fancydeepin/archive/2015/06/27/maven-p.html