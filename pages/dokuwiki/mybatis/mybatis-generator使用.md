title: mybatis-generator使用 

#  mybatis:mybatis-generator使用 

由于MyBatis属于一种半自动的ORM框架，所以主要的工作将是书写Mapping映射文件，但是由于手写映射文件很容易出错，所以查资料发现有现成的工具可以自动生成底层模型类、Dao接口类甚至Mapping映射文件。
#  一、建立表结构 

```

CREATE TABLE `user` (
  `id` varchar(50) NOT NULL,
  `username` varchar(18) CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL,
  `password` varchar(18) DEFAULT NULL,
  `email` varchar(50) DEFAULT NULL,
  `name` varchar(18) DEFAULT NULL,
  `sex` varchar(2) DEFAULT NULL,
  `birthday` varchar(50) DEFAULT NULL,
  `address` varchar(500) DEFAULT NULL,
  `tel` varchar(18) DEFAULT NULL,
  `qq` varchar(18) DEFAULT NULL,
  `image` varchar(50) DEFAULT NULL,
  `sfjh` varchar(1) DEFAULT NULL,
  `sfzx` varchar(1) DEFAULT NULL,
  `sfhf` varchar(1) DEFAULT NULL,
  `sfpl` varchar(1) DEFAULT NULL,
  `sffx` varchar(1) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf-8;

```
#  二、下载mybatis-generator-core 

http://search.maven.org/
#  三、生成配置文件 
**github有个gui配置生成工具：https://github.com/astarring/mybatis-generator-gui**

新建一个空的XML配置文件，名称可以随便取，这里以generatorConfig.xml为名。最好将这个文件放在下载后的lib目录中，如图：
![](/data/dokuwiki/mybatis/pasted/20150511-054918.png)
其中mysql的驱动可以随便放在非中文路径的地方，这里为了方便就放在lib目录下。

自动生成最重要的就是配置文件的书写，现在就开始介绍generatorConfig.xml这个文件的具体内容：
```

<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE generatorConfiguration  
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"  
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">  
<generatorConfiguration>  
<!-- 数据库驱动-->  
    <classPathEntry  location="mysql-connector-java-5.0.6-bin.jar"/>  
    <context id="DB2Tables"  targetRuntime="MyBatis3">  
        <commentGenerator>  
            <property name="suppressDate" value="true"/>  
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->  
            <property name="suppressAllComments" value="true"/>  
        </commentGenerator>  
        <!--数据库链接URL，用户名、密码 -->  
        <jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost/test" userId="test" password="test">  
        </jdbcConnection>  
        <javaTypeResolver>  
            <property name="forceBigDecimals" value="false"/>  
        </javaTypeResolver>  
        <!-- 生成模型的包名和位置-->  
        <javaModelGenerator targetPackage="test.model" targetProject="src">  
            <property name="enableSubPackages" value="true"/>  
            <property name="trimStrings" value="true"/>  
        </javaModelGenerator>  
        <!-- 生成映射文件的包名和位置-->  
        <sqlMapGenerator targetPackage="test.mapping" targetProject="src">  
            <property name="enableSubPackages" value="true"/>  
        </sqlMapGenerator>  
        <!-- 生成DAO的包名和位置-->  
        <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao" targetProject="src">  
            <property name="enableSubPackages" value="true"/>  
        </javaClientGenerator>  
        <!-- 要生成哪些表-->  
        <table tableName="about" domainObjectName="AboutDto" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>  
        <table tableName="user" domainObjectName="UserDto" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>  
        <table tableName="syslogs" domainObjectName="SyslogsDto" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>  
    </context>  
</generatorConfiguration>  

```
1、其中需要注意的有数据库驱动、数据库URL、用户名、密码、生成模型的包名和位置、生成映射文件的包名和位置、生成DAO的包名和位置以及最后需要生成的表名和对应的类名。

#  四、运行 

需要通过CMD命令行方式来运行，首先可以先准备一个运行的脚本，这里使用的脚本是：java -jar mybatis-generator-core-1.3.2.jar -configfile generatorConfig.xml -overwrite
需要注意的是：mybatis-generator-core-1.3.2.jar为下载的对应版本的jar，generatorConfig.xml 为配置文件名，如果不为这个可以在这里进行修改。
启动cmd进入到“F:\soft\mybatis-generator-core-1.3.2\lib”这个目录下，如图：
![](/data/dokuwiki/mybatis/pasted/20150511-055028.png)
生成成功后进到src目录下，可以看到已经生成了对应的model、dao、mapping，如图：
![](/data/dokuwiki/mybatis/pasted/20150511-055034.png)
原文地址：http://blog.csdn.net/wyc_cs/article/details/9023117