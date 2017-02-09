title: eclipse问题记录 

#  eclipse问题记录 
1、生成javadoc时出现错误：错误: 编码GBK的不可映射字符。
添加javadoc命令参数 -encoding UTF-8 

2.maven新建web项目时pom.xml文件里<packaging>war</packaging>报错；web.xml is missing and <failOnMissingWebXml> is set to true
```

<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-war-plugin</artifactId>
<version>2.6</version>
<configuration>
<failOnMissingWebXml>false</failOnMissingWebXml>
</configuration>
</plugin>

```

3.新建maven项目修改project facets中的dynamic module版本时出现错误：failed while installing dynamic web module 3.0
解决：http://stackoverflow.com/questions/19661135/dynamic-web-module-3-0-3-1
问题描述：
项目右键-> properties>project-facets将dynamic web module打上勾并选择3.0（跟web.xml中的版本号对应），勾选runtimes下的apache tomcatv7.0
![](/data/dokuwiki/tooluse/pasted/20150907-174755.png)
修改dynamic web module过程中出现问题failed while installing dynamic web module 3.0，
进入项目的硬盘目录，**修改.settings目录下的org.eclipse.wst.common.project.facet.core.xml**，修改如
![](/data/dokuwiki/tooluse/pasted/20150907-174847.png)
修改之后进行刷新。

112、eclipse部署，在tomcat中找不到eclipse发布的项目。eclipse更改项目发布路径
在新版的eclipse中，配置好项目，发布之后，发现在tomcat的webapps下找不到该项目，而是在d:\workspace\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\wtpwebapps
下，解决办法就是重新配置一下eclipse在tomcat下对项目的发布路径。
首先在工作面板中找到“servers”，然后右键点击当前的tomcat，然后选择remove（最好先把项目停掉），然后再右键选择“clean..”
其次双击那个tomcat，就会进入它的配置界面，然后找到左边第二个 Server Locations，你那个单选框选中的应该是第一个，你选择第二个就行。
如果你想自由选择发布目录，而不是发布在tomcat的webapps下面，你就选择第三个，在 Server Path 中输入你想要的路径后，保存即可以了。
选择第二个之后，再在下面的depoly path中选择tomcat下的webapps就行。
![](/data/dokuwiki/tooluse/pasted/20150902-111721.png)

##  maven java.lang.ClassNotFoundException: org.springframework.web.context.ContextLoaderListener 
![](/data/dokuwiki/tooluse/pasted/20151101-011318.png)![](/data/dokuwiki/tooluse/pasted/20151101-011323.png)![](/data/dokuwiki/tooluse/pasted/20151101-011328.png)![](/data/dokuwiki/tooluse/pasted/20151101-011334.png)
参考：http://www.csdn123.com/html/topnews201408/77/6177.htm http://stackoverflow.com/questions/6322711/tomcat-spring-web-class-not-found-exception-org-springframework-web-context