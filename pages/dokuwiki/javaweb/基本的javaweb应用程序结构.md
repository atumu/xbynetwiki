title: 基本的javaweb应用程序结构 

#  基本的javaweb应用程序结构 
Java Web高级编程 --涵盖Xxx Nicholas S,Williams著读书笔记。
大量的组件共同组成了一个JavaEE Web应用程序。
##  Servlet、Filter、Listener、JSP 
  * Servlet是用于接受和响应HTTP请求的Java类。
  * Filter是拦截过滤发送给Servlet请求的组件。通过过滤器可以满足各种需求，包括数据格式化，对返回的数据进行压缩，认证和授权等。
  * Listener可以通知代码多种事件，例如应用程序启动，关闭，HTTP Session创建和销毁。
  * JSP:用于展示。包含JSTL,JUEL(即EL表达式)，自定义标签，国际化等。
##  目录结构和WAR文件 
META-INF/(使用于tomcat的context.xml )  不在classpath下，注意
WEB-INF/classes
WEB-INF/tags
WEB-INF/tld
WEB-INF/i18n
WEB-INF/lib
WEB-INF/META-INF/(persistence.xml,orm.xml)

WEB-INF和根目录下的META-INF都是受保护的，无法直接通过URL访问的

##  部署描述符web.xml,web-fragment.xml 
/WEB-INF/web.xml
该文件通过包含Servlet,Filter,Listener定义，以及HTTP Session,JSP和404、应用程序的配置等。
Servlet3.0提供注解配置方式。

web片段，即应用程序中的JAR文件可以包含Servlet、Filter、Listener的配置，配置在JAR包中META-INF/web-fragment.xml中。web片段也可以使用注解方式。
web-fragment.xml排序<ordering>

##  类加载器架构 
首先介绍**双亲优先类加载委托模式**
![](/data/dokuwiki/javaweb/pasted/20151024-211707.png)
这是传统JavaSE平台的ClassLoader加载模式。当低级别类加载器申请加载一个类时，它不是立马进行加载，而总是首先将该任务委托给它的父类加载器。继续向上委托直至根加载器。如果它的父加载器未能找到该类，那么当前的类加载器将尝试进行加载。

但是在web应用中，这种方式会存在问题。比如应用服务器与应用共同依赖log4j，但是应用服务器依赖的版本是log4j1.x,而应用依赖的是log4j2.x。那么这将导致冲突。为了解决该问题，
JavaEE Web应用服务器中采用的是**子女优先类加载委托模式。**
![](/data/dokuwiki/javaweb/pasted/20151024-212711.png)
在JavaEE Web应用服务器中，每个Web应用程序都被分配了一个自由的相互隔离的类加载器，它们都继承自公用的服务器类加载器。通过隔离不同的应用程序，它们不能访问相互的类。
web应用类加载器通常会先自己进行加载，只有在自己无法加载某个类的时候，才会请求它的父类加载器协助加载。通过这种方式Web应用中的类库会被优先使用，而不是服务器中的类库版本。
但是为了维持绑定的JavaSE类的安全状态，Web应用程序类加载器仍然会在尝试加载任何类之前与根加载器确认。

尽管对于Web应用程序，这种模式优点很明显，但是这种方式仍然有不适用的地方。通过兼容JavaEE的服务器通常会提供修改委托模式的方法，以便需要。

##  EAR企业级归档文件 
META-INF/application.xml特有的部署描述文件
Xxx.war
Xxx2.war
Xxxa.jar
为了允许EAR中多个模块，即war共享通用库。在服务器类加载器和为每个模块分配的web应用程序类加载器之间插入一个额外的EAR类加载器。
` 但是Tomcat并不支持EAR格式。 `

