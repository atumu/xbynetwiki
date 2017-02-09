location:dokuwiki/javaweb/tomcat_memcached实现session共享
modifyAt:2016-12-15 22:17:08
author:xbynet
createAt:2016-12-15 22:17:08
title: tomcat_memcached实现session共享 

#  Tomcat+memcached实现session共享 
需要的jar包：
![](/data/dokuwiki/javaweb/pasted/20151016-092548.png)
将上述jar包放入到tomcat的安装目录lib中。
![](/data/dokuwiki/javaweb/session_share.zip|jar包下载)
配置tomcat. 在%TOMCAT_HOME%\config\` context.xml `文件中加入：
```

<?xml version='1.0' encoding='utf-8'?>
<Context>
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager" memcachedNodes="n1:localhost:11211,n2:localhost:11222" 
    requestUriIgnorePattern=".*\.(png|gif|jpg|css|js){1}quot;" sessionBackupAsync="false" sessionBackupTimeout="1800000" 
    copyCollectionsForSerialization="false" 
    transcoderFactoryClass="de.javakaffee.web.msm.serializer.javolution.JavolutionTranscoderFactory"/>  
</Context>

```


##  进行测试是否session共享 
```

<%@ page language="java" import="java.util.*" pageEncoding="ISO-8859-1"%>  
<%  
String path = request.getContextPath();  
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";  
%>  
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">  
<html>  
  <head>  
    <base href="<%=basePath%>">    
    <title>My JSP 'session.jsp' starting page</title>  
    <meta http-equiv="pragma" content="no-cache">  
    <meta http-equiv="cache-control" content="no-cache">  
    <meta http-equiv="expires" content="0">      
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">  
    <meta http-equiv="description" content="This is my page">  
    <!-- 
    <link rel="stylesheet" type="text/css" href="styles.css"> 
    -->  
  </head>  
  <body>  
    <%  
     System.out.println(session.getId());  
     out.println("<br> This is (TOMCAT1|TOMCAT2), SESSION ID:" + session.getId()+"<br>");  
    %>  
  </body>  
</html>

```
要打印出n1，tomcat等字样才表示session共享成功。
##  配置说明 
1、 session存储到memchached实现方案时。他主要功能是**修改tomcat的session存储机制，使之能够把session序列化存放到memcached中**。
2、Manager标签属性说明：
             className
                    此属性是必须的。
             memcachedNodes
                        此属性是必须的。这个属性必须包含你所有运行的memcached节点。每个节点的定义格式为<id>:<host>:<port>。
                   多个之间用空格或半角逗号隔开(如：memcachedNodes="n1:localhost:11211,n2:localhost:11212")。
                       如果你设置单个memcache节点<id>是可选的，所以它允许设置为<host>:<port>（memcachedNodes="localhost:11211"）。
             failoverNodes
                      可选项，属性只能用在非粘连Session机制中。
                      此属性必须包含memcached节点的Id，此节点是Tomcat作为备份使用。多个之间用空格或逗号隔开
              memcachedProtocol
                   可选项，默认为text。出属性指明memcached使用的存储协议。只支持text或者binary。
              sticky 可选项，默认为true。
                    指定使用粘性的还是非粘性的Session机制。
              lockingMode 可选项， 此属性只对非粘性Session有用，默认为none。
                     指定非粘性Session的锁定策略。他的只有
                        (1)、none:从来不加锁
                        (2)、all: 当请求时对Session锁定，直到请求结束
                        (3)、auto:对只读的request不加锁，对非只读的request加锁
                        (4)、uriPattern:<regexp>: 使用正则表达式来比较requestRUI + "?" + queryString来决定是否加锁，
             requestUriIgnorePattern  可选项
                        此属性是那些不能改备份Session的请求的正则表达式。如果像css,javascript,图片等静态文件被同一个Tomcat和同一个应用上下文来提供，这些
                   请求也会通过memcached-session-manager。但是这些请求在一个http会话中几乎没什么改变，所以他们没必要触发Session备份。所以那些静态文件
                   没必要触发Session备份，你就可以使用此属性定义。此属性必须符合java regex正则规范。
            sessionBackupAsync 可选项，默认true
                        指定Session是否应该被异步保存到Memcached中。 如果被设置为true，backupThreadCount设置起作用，如果设置false，通过sessionBackupTimeout
                   设置的过期时间起作用。
            backupThreadCount 可选项，默认为CPU内核数。
                       用来异步保存Session的线程数(如果sessionBackupAsync="true")。
            sessionBackupTimeout  可选项，默认100，单位毫秒
                       设置备份一个Session所用的时间，如果操作超过时间那么保存失败。此属性只在sessionBackupAsync="false"是起作用。默认100毫秒
            sessionAttributeFilter 可选项 从1.5.0版本有
                       此属性是用来控制Session中的那个属性值保存到Memcached中的正则表达式。郑则表达式被用来匹配Session中属性名称。如
                  sessionAttributeFilter="^(userName|sessionHistory)$" 指定了只有"userName"和"sessionHistory"属性保存到Memcached中。
                  依赖于选择的序列化策略。
            transcoderFactoryClass 可选，默认为 de.javakaffee.web.msm.JavaSerializationTranscoderFactory
                       此属性值是创建序列化和反序列化保存到Memcached中的Session的编码转换器的工厂类名。这个指定的类必须实现了de.javakaffee.web.msm.TranscoderFactory
                 和提供一个无参的构造方法。例如其他的有效的实现在其他packages/jars中提供如：msm-kryo-serializer,msm-xstrea-serializer和msm-javolution-serializer.
            copyCollectionsForSerialization 可选项，默认false。
            customConverter 可选项
                  
            enableStatistics 可选项，默认true
                   用来指定是否进行统计。
            enabled 可选项，默认true
                    指定Session保存到Memcached中是否可用和是否可以通过JMX进行改变。只用于粘性Session。
参考：http://blog.csdn.net/bluejoe2000/article/details/24883967