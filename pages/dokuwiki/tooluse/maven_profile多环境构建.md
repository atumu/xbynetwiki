title: maven_profile多环境构建 

#  Maven Profile多环境构建 
Profile 能让你为一个特殊的环境自定义一个特殊的构建；profile 使得不同环境间构建的可移植性成为可能。
不同的构建环境是什么意思？构建环境的两个例子是产品环境和开发环境。当你在开发环境中工作时，你的系统可能被配置成访问运行在你本机的开发数据库实例，而在产品环境中，你的系统被配置成从产品数据库读取数据。Maven 能让你定义任意数量的构建环境（构建profile），这些定义可以覆盖pom.xml 中的任何配置。
。Profile 也可以通过环境和平台被激活，你可以自定义一个构建，它根据不同的操作系统或者不同的JDK 版本有不同的行为。

##  通过Maven Profiles 实现可移植性 
**Maven 中的profile 是一组可选的配置，可以用来设置或者覆盖配置默认值。有了profile，你就可以为不同的环境定制构建。**profile 可以在pom.xml 中配置，并给定一个id。然后你就可以在运行Maven 的时候使用的命令行标记告诉Maven 运行特定profile 中的目标。

` 环境可移植性 `：针对不同的环境有特定的行为和配置。例如，一个项目在测试环境中包含一个对于测试数据库的引用，而在产品环境中则引用了产品数据库，那么该项目的构建是环境可移植的。这很有可能是因为该构建针对不同的环境有不同的属性组。

###  profile覆盖 
以下pom.xml 使用production profile 覆盖了默认的Compiler插件设置。
Example 11.1. 使用一个Maven Profile 覆盖Compiler 插件设置
```

<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>org.sonatype.mavenbook</groupId>
<artifactId>simple</artifactId>
<packaging>jar</packaging>
<version>1.0-SNAPSHOT</version>
<name>simple</name>
<url>http://maven.apache.org</url>
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>3.8.1</version>
		<scope>test</scope>
	</dependency>
</dependencies>
<profiles>
	<profile>
		<id>production</id>
		<build>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<configuration>
						<debug>false</debug>
						<optimize>true</optimize>
					</configuration>
				</plugin>
			</plugins>
		</build>
	</profile>
</profiles>
</project>

```
本例中，我们添加了一个名为production 的profile，它覆盖了Maven Compiler 插件的默认配置
` **一个 profile 元素可以包含很多其它元素，只要这些元素可以出现在POM XML 文档的project 元素下面**。 `本例中，我们覆盖了Compiler 插件的行为，因此必须覆盖插件配置，该配置通常位于一个build 和一个plugins 元素下面。
` 每个 profile 必须要有一个id 元素。这个id 元素包含的名字将在命令行调用profile 时被用到。我们可以通过传给Maven 一个-P<profile_id>参数来调用profile。 `

要使用 production profile 来运行mvn install，你需要在命令行传入-Pproduction 参数。要验证production profile 覆盖了默认的Compiler 插件配置，可以像这样以开启调试输出(-X) 的方式运行Maven。
```

mvn clean install -Pproduction -X

```

###  profile 激活 
当你需要基于一些变量如操作系统和JDK 版本来进行配置的时候怎么做呢？
Maven 提供了一种针对不同环境参数“激活”一个profile 的方式，这就叫做profile 激活。

假设我们拥有一个Java 类库，它有一个特定的功能只有在Java 6 下才可用
使用Profile 激活动态包含子模块
```

<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>org.sonatype.mavenbook</groupId>
<artifactId>simple</artifactId>
<packaging>jar</packaging>
<version>1.0-SNAPSHOT</version>
<name>simple</name>
<url>http://maven.apache.org</url>
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>3.8.1</version>
		<scope>test</scope>
	</dependency>
</dependencies>
<profiles>
	<profile>
		<id>jdk16</id>
		<activation> <!--激活配置 -->
                 	 <activeByDefault>false</activeByDefault> <!--默认是否激活 -->
			<jdk>1.6</jdk>
		</activation>
		<modules>
			<module>simple-script</module>
		</modules>
	</profile>
</profiles>
</project>

```
如果你在Java 1.6 下运行mvn install，你会看到Maven 下行到simple-script子目录构建simple-script 项目。如果你在Java 1.5 上运行mvn install，Maven就不会去构建simple-script 子模块。让我们详细看一下激活配置：
` activation 元素列出了所有激活profile 需要的条件。 `该例中，我们的配置为，当Java 版本以“1.6”开头的时候profile 会被激活。这包括“1.6.0_03”，“1.6.0_02”以及所有其它以“1.6”开头的字符串。激活参
数不局限于Java 版本，要了解激活参数的完整列表，请看激活配置。

**激活配置**
**激活配置元素下可以包含一个或者多个选择器：包含JDK 版本，操作系统参数，文件，以及属性。当所有标准都被满足的时候一个profile 才会被激活。**例如，一个profile可以要求操作系统家族为Windoes，JDK 版本为1.4，那么该profile 只有当构建在Windows 机器上的Java 1.4 上运行的时候才会被激活。如果该profile 被激活，那么它定义的所有配置都会覆盖原来POM 中对应层次的元素，**就像使用命令行参数-P 引入该profile 一样**。下面的例子展示了一个profile，它通过一个十分复杂的配置组合激活，包括操作系统参数，属性，以及JDK 版本。
Example 11.4. Profile 激活参数：JDK 版本，操作系统参数，以及属性
```

<project>
	...
	<profiles>
		<profile>
			<id>dev</id>
			<activation>
				<activeByDefault>false</activeByDefault>
				<jdk>1.5</jdk>
				<os>
					<name>Windows XP</name>
					<family>Windows</family>
					<arch>x86</arch>
					<version>5.1.2600</version>
				</os>
				<property>
					<name>mavenVersion</name>
					<value>2.0.5</value>
				</property>
				<file>
					<exists>file2.properties</exists>
					<missing>file1.properties</missing>
				</file>
			</activation>
			...
		</profile>
	</profiles>
</project>

```
**activeByDefault 元素控制一个profile 是否默认被激活。**
property 元素告诉Maven，当mavenVersion 属性的值被设置成2.0.5 的时候才激活profile**。mavenVersion 是一个在所有Maven 构建中可用的隐式属性。**
file 元素告诉我们只有当某些文件存在（或者不存在）的时候才激活profile。该例中的dev profile 只有**在项目基础目录中存在**file2.properties 文件，并且不存在file1.properties 文件的时候才被激活。

###  通过属性缺失激活 
你可以基于一个属性如environment.type 的值来激活一个profile。当environment.type 等于dev 的时候激活development profile，或者当environment.type 等于prod 的时候激活production profile。你也可以通过一个属性的缺失来激活一个profile。下面的配置中，只有在Maven 运行过程中属性environment.type 不存在profile 才被激活。
在属性缺失的情况下激活Profile
```

<project>
...
<profiles>
<profile>
<id>development</id>
<activation>
<property>
<name>!environment.type</name> 
</property>
</activation>
</profile>
</profiles>
</project>

```
注意属性名称前面的惊叹号。惊叹号通常表示“否定”的意思。当没有设置${environment.type}属性的时候，这个profile 被激活。

###  外部化Profile 
**如果你开始大量使用Maven profile，你会希望将profile 从POM 中分离，使用一个单独的文件如profiles.xml。**你可以混合使用定义在pom.xml 中和外部profiles.xml 文件中的profile。` 只需要将profiles 元素放到${basedir}目录下的profiles.xml 文件中 `，然后照常运行Maven 就可以。profiles.xml 文件的大概内容如下：
Example 11.6. 将profile 放到一个profiles.xml 文件中
```

<profiles>
<profile>
<id>development</id>
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-compiler-plugin</artifactId>
<configuration>
<debug>true</debug>
<optimize>false</optimize>
</configuration>
</plugin>
</plugins>
</build>
  </profile>
<profile>
<id>production</id>
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-compiler-plugin</artifactId>
<configuration>
<debug>false</debug>
<optimize>true</optimize>
</configuration>
</plugin>
</plugins>
</build>
</profile>
</profiles>

```


如果正确的使用profile，它可以为不同的平台很容易的自定义构建。如果你构建中的一些东西需要定义一个平台特定的路径，如应用程序服务器，你可以将这些配置点放到profile 中，然后由操作系统参数激活。如果你有一个项目需要为不同的环境生成不同的构件，你可以为不同的环境和平台自定义profile 特定的插件行为，从而自定义构建行为。使用profile，构建可以变得容易移植，没有必要为一个新的环境重写你的构建逻辑，只需要重写那些需要变化的配置，共享那些可以被共享的配置。