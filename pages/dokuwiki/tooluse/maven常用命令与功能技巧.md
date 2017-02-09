createAt:2017-01-18 10:21:21
author:xbynet
modifyAt:2017-01-18 10:21:21
location:dokuwiki/tooluse/maven常用命令与功能技巧
title:maven常用命令与功能技巧

# 通过Maven将指定Jar包下载到指定的本地目录
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
 
    <groupId>Spider</groupId>
    <artifactId>Spider</artifactId>
    <version>1.0</version>
 
    <dependencies>
        <dependency>
            <!-- jsoup HTML parser library @ http://jsoup.org/ -->
            <groupId>org.jsoup</groupId>
            <artifactId>jsoup</artifactId>
            <version>1.9.2</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.9.13</version>
        </dependency>
    </dependencies>
 
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-dependency-plugin</artifactId>
                <configuration>
                    <excludeTransitive>false</excludeTransitive> 
                    <stripVersion>true</stripVersion>
                    <outputDirectory>./lib</outputDirectory>
                </configuration>
                 
            </plugin>
        </plugins>
    </build>
 
</project>
```

然后在命令窗口内输入命令：
mvn dependency:copy-dependencies
回车运行后，就可以在当前目录下面发现一个lib文件夹，这个文件夹内，就是我们需要的jar包。