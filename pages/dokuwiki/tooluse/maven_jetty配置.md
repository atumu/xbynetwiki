title: maven_jetty配置 

#  Maven+Jetty配置 
```

<plugin>
                    <groupId>org.eclipse.jetty</groupId>
                    <artifactId>jetty-maven-plugin</artifactId>
                    <version>9.3.6.v20151106</version>
                    <configuration>
                        <scanIntervalSeconds>10</scanIntervalSeconds>
                      <webApp>
      			<contextPath>/test</contextPath>
                         <tempDirectory>${project.build.directory}/work</tempDirectory>
                         <defaultsDescriptor>src/main/resources/webdefault.xml</defaultsDescriptor>
   		     </webApp>
                         <httpConnector>
         		 	<port>8080</port>
       			 </httpConnector>
                    </configuration>
                </plugin>

```
配置说明：
参数：
scanIntervalSeconds： 热部署时间
webAppConfig-->contextPath: 热部署的项目名
webAppConfig-->defaultsDescriptor：解决jetty热部署不能修改静态资源的问题
需要把对应版本jetty-webapp中的webdefault.xml拷贝到src/main/resources/目录中，并且修改下面的参数 ( 要解决静态文件锁定问题，需要修改$maven_repo$\org\eclipse\jetty\jetty-webapp\9.3.6.v20151106\jetty-webapp-9.3.6.v20151106.jar\org\eclipse\jetty\webapp\webdefault.xml文件)
```

 <init-param>
      <param-name>useFileMappedBuffer</param-name>
      <param-value>false</param-value><!--原来是true-->
 </init-param>

```

http://localhost:8080/test访问项目的主页

(jetty:run -Djetty.port=9080)


##  开启jetty调试 
 右键maven工程，在弹出的菜单中选择[Debug As]，首次选择[Maven build...]，以后选择[Maven build]来读取保存的配置启动： 
![](/data/dokuwiki/tooluse/pasted/20151223-231655.png)
在浏览器输入地址http://localhost:8080/prospect/already/mosaic.htm，在代码上加断点，命中后IDE提示：Source not found：
![](/data/dokuwiki/tooluse/pasted/20151223-231710.png)
解决： 
点击[Edit Source Lookup Path...]添加源代码工程或目录 完成后即可调试代码： 

参考：
官方文档：http://www.eclipse.org/jetty/documentation/current/jetty-maven-plugin.html
http://www.blogjava.net/fancydeepin/archive/2012/06/23/maven-jetty-plugin.html
http://czj4451.iteye.com/blog/1942437
http://my.oschina.net/KingPan/blog/273505
http://itstarting.iteye.com/blog/598695