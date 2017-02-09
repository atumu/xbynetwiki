title: maven分模块构建企业级项目 

#  maven分模块构建企业级项目 
在平时的Javaweb项目开发中为了便于后期的维护，我们一般会进行**分层开发**，
最常见的就是分为domain（域模型层）、dao（数据库访问层）、service（业务逻辑层）、web（表现层），这样分层之后，各个层之间的职责会比较明确，后期维护起来也相对比较容易，
今天我们就是使用Maven来构建以上的各个层。项目结构如下：
```

　　system-parent
    　　　　|----pom.xml
    　　　　|----system-domain
        　　　　　　　　|----pom.xml
    　　　　|----system-dao
        　　　　　　　　|----pom.xml
    　　　　|----system-service
        　　　　　　　　|----pom.xml
    　　　　|----system-web
        　　　　　　　　|----pom.xml

```

##  一、创建项目 
创建system-parent项目，用来给各个子模块继承。搭建多模块项目，必须要有一个packaging为pom的根目录。进入命令行，输入以下命令：
```

mvn archetype:create -DgroupId=me.gacl -DartifactId=system-parent -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

```
命令执行完成之后可以看到在当前目录(C:\Documents and Settings\Administrator)生成了system-parent目录，里面有一个src目录和一个pom.xml文件，如下图所示：
![](/data/dokuwiki/tooluse/pasted/20151207-140045.png)
将src文件夹删除，然后修改pom.xml文件，将<packaging>jar</packaging>修改为` <packaging>pom</packaging> `，pom表示它是一个被继承的模块，修改后的内容如下：
```

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>me.gacl</groupId>
  <artifactId>system-parent</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>system-parent</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>

```
##  二、创建sytem-domain模块 
在命令行进入创建好的system-parent目录，然后执行下列命令：
mvn archetype:create -DgroupId=me.gacl -DartifactId=system-domain -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
![](/data/dokuwiki/tooluse/pasted/20151207-140157.png)
命令执行完成之后可以看到在system-parent目录中生成了system-domain，里面包含src目录和pom.xml文件。如下图所示：
![](/data/dokuwiki/tooluse/pasted/20151207-140234.png)
同时，在system-parent目录中的pom.xml文件自动添加了如下内容：
```

<modules>
    <module>system-domain</module>
</modules>

```
这时，system-parent的pom.xml文件如下：
```

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>me.gacl</groupId>
  <artifactId>system-parent</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>system-parent</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  
  <modules>
    <module>system-domain</module>
  </modules>
  
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

</project>

```
　修改system-domain目录中的pom.xml文件，**把<groupId>me.gacl</groupId>和<version>1.0-SNAPSHOT</version>去掉**，加上<packaging>jar</packaging>，因为**groupId和version会继承system-parent中的groupId和version**，packaging设置打包方式为jar.修改过后的pom.xml文件如下：
```

<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>me.gacl</groupId>
    <artifactId>system-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
     <relativePath>../pom.xml</relativePath>
  </parent>
  
  <artifactId>system-domain</artifactId>
  <packaging>jar</packaging>
  
  <name>system-domain</name>
  <url>http://maven.apache.org</url>
</project>

```
##  三、创建sytem-dao模块 
与前面相同，不同之处在于:
修改system-dao目录中的pom.xml文件，，把<groupId>me.gacl</groupId>和<version>1.0-SNAPSHOT</version>去掉，加上<packaging>jar</packaging>，因为groupId和version会继承system-parent中的groupId和version，packaging设置打包方式为jar，` 同时添加对system-domain模块的依赖，修改后的内容如下 `：
```

<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>me.gacl</groupId>
    <artifactId>system-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
     <relativePath>../pom.xml</relativePath>
  </parent>

  <artifactId>system-dao</artifactId>
  <packaging>jar</packaging>

  <name>system-dao</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
    <!--system-dao需要使用到system-domain中的类，所以需要添加对system-domain模块的依赖-->
     <dependency>
      <groupId>me.gacl</groupId>
      <artifactId>system-domain</artifactId>
      <version>${project.version}</version>
    </dependency>
  </dependencies>
</project>

```
其余模块略。。

##  安装与运行 

最后同时，在system-parent目录中的pom.xml文件自动变成如下内容：
```

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>me.gacl</groupId>
  <artifactId>system-parent</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>system-parent</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <modules>
    <module>system-domain</module>
    <module>system-dao</module>
    <module>system-service</module>
    <module>system-web</module>
  </modules>
</project>

```

在命令行进入system-parent目录，然后执行下列命令：
```

mvn clean install

```
**然后运行可以分模块分别运行。不一定要运行整个项目。**
![](/data/dokuwiki/tooluse/pasted/20151207-140743.png)![](/data/dokuwiki/tooluse/pasted/20151207-140749.png)

##  全局依赖与自定义依赖、dependencyManagement 
**继承功能之全局依赖和自定义依赖：**
**1.全局依赖**：在所有继承parent的子模块中都会引入jar包
```

<!-- 全局依赖 -->
<dependencies>
<dependency>
<groupId>junit</groupId>
<artifactId>junit</artifactId>
<version>${junit.version}</version>
<scope>test</scope>
</dependency>
</dependencies>

```
**2、自定义依赖：**在继承parent的子模块中通过指定来引入依赖
**父模块通过dependencyManagement指定一些依赖**
```

<!-- 自定义依赖 -->
<dependencyManagement>
 	<dependencies>
<dependency>
<groupId>org.springframework</groupId>
 	<artifactId>spring-web</artifactId>
 	<version>3.1.1.RELEASE</version>
</dependency>
 	</dependencies>
</dependencyManagement>

```
然后子模块如果需要那么就可以直接指定：
```

<dependencies>
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-web</artifactId>
</dependency>
</dependencies>

```
不需要指定versin
参考：http://www.cnblogs.com/xdp-gacl/p/4242221.html
http://www.cnblogs.com/quanyongan/archive/2013/05/28/3103243.html
http://jingyan.baidu.com/article/b87fe19e93b8905218356880.html