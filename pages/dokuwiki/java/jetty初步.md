title: jetty初步 

#  JETTY实战之安装 运行 部署 
1. 首先从Jetty的官方网站http://wiki.eclipse.org/Jetty/Starting/Downloads下
载最新的Jetty，上面有两个版本7.x和8.x，7.x是运行在JDK5及以上版
本，8.x是运行在JDK6及以上版本，这里我选择了8.0.4版本。
2. 解压压缩包到指定目录，且将其目录路径定义为${JETTY_HOME}
3. 启动Jetty服务
3.1 进入${JETTY_HOME}目录，然后运行“java -jar start.jar”，就可以启动Jetty server了
3.2 打开浏览器，访问http://localhost:8080，此时可以看到Jetty的欢迎页面了。
4. Jetty配置
4.1 Jetty的配置文件都是放在${JETTY_HOME}/etc目录下；
4.2 通过${JETTY_HOME}/etc/jetty-webapps.xml文件，可以看出Jetty中默认将所有的web app都放在了${JETTY_HOME}/webapps目录下；
4.3 在Jetty包中默认带了一个test.war的应用，可以${JETTY_HOME}/webapps目录下找到这个文件，在启动Jetty服务的时候默认已经部署了test.war应用。对于test.war文件，Jetty还定义了context文件，放在${JETTY_HOME}/contexts/test.xml，其中将contextPath定义成了“/”，这就是为什么默认访问http://localhost:8080/的时候为什么是访问test应用的原因了。
5 部署新的web应用程序
5.1 对于war包的部署，只需要将war文件放到${JETTY_HOME}/webapps目录下，然后就可以通过浏览器直接访问了；
5.2 对于web应用程序目录的部署，此时可以将web应用程序目录复制到${JETTY_HOME}/webapps/<myapp>目录下，然后在${JETTY_HOME}/contexts/<myapp>.xml文件，其中内容如下：
```

<?xml version=“1.0” encoding=“ISO-8859-1″?>
 <!DOCTYPE Configure PUBLIC “- //Jetty//Configure//EN” “http://www.eclipse.org/jetty/configure.dtd”>
 <Configure class=“org.eclipse.jetty.webapp.WebAppContext”>
 <Set name=“contextPath”>/myapp</Set>
 <Set name=“war”><SystemProperty name=“jetty.home” default=“.”/>/webapps/myapp</Set>
 </Configure>

```
重新启动Jetty服务，访问http://localhost:8080/myapp就可以看到新部署web应用程序了。

#  JETTY实战之嵌入式运行JETTY 
Jetty最常用的一种用法是把Jetty嵌入到自己的Java应用程序中，此时Jetty作为一个后台的Servlet容器运行，接受用户的http请求，下面是一个最简单的嵌入Jetty的用法。
创建一个Server类，用了启动Jetty server，并且通过一个HelloHandler来处理浏览器发送过来的请求；
```

import org.eclipse.jetty.server.Server;
04.
05. public class MyServer {
06. public static void main(String[] args) throws Exception {
07. Server server = new Server(8080);
08. server.setHandler(new HelloHandler());
09.
10. server.start();
11. server.join();
12. }
13. }

import org.eclipse.jetty.server.Request;
10. import org.eclipse.jetty.server.handler.AbstractHandler;
11.
12. public class HelloHandler extends AbstractHandler {
13. public void handle(String target, Request baseRequest, HttpServletRequest request, HttpSer
vletResponse response)
14. throws IOException, ServletException {
15. response.setContentType(“text/html;charset=utf-8″);
16. response.setStatus(HttpServletResponse.SC_OK);
17. baseRequest.setHandled(true);
18. response.getWriter().println(“<h1>Hello World</h1>”);
19. response.getWriter().println(“Request url: “ + target);
20. }
21. }


```
运行MyServer类，然后通过浏览器访问http://localhost:8080/，可以看到“Hello World！”和请求的url。

另外一种的嵌入式：
```

import org.eclipse.jetty.server.Server;
04. import org.eclipse.jetty.servlet.ServletContextHandler;
05. import org.eclipse.jetty.servlet.ServletHolder;
06.
07. public class ServletContextServer {
08. public static void main(String[] args) throws Exception {
09. Server server = new Server(8080);
10.
11. ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
12. context.setContextPath(“/”);
13. server.setHandler(context);
15. // http://localhost:8080/hello
16. context.addServlet(new ServletHolder(new HelloServlet()), “/hello”);
17. // http://localhost:8080/hello/kongxx
18. context.addServlet(new ServletHolder(new HelloServlet(“Hello Kongxx!”)), “/hello/kongxx”);
19.
20. // http://localhost:8080/goodbye
21. context.addServlet(new ServletHolder(new GoodbyeServlet()), “/goodbye”);
22. // http://localhost:8080/goodbye/kongxx
23. context.addServlet(new ServletHolder(new GoodbyeServlet(“Goodbye kongxx!”)), “/goodbye/kongxx”);
24.
25. server.start();
26. server.join();
27. }
28. }

```