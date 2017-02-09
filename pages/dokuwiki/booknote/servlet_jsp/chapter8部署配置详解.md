title: chapter8部署配置详解 

#  Chapter8Servlet部署配置详解 
##  1 定义头和根元素 
```

<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>
</web-app>

```
##  2 部署描述符文件内的元素次序 
下面的列表给出了所有可直接出现在web-app元素内的合法元素所必需的次序。例如，` 此列表说明servlet元素必须出现在所有servlet-mapping元素之前 `。请注意，所有这些元素都是可选的。因此，可以省略掉某一元素，但不能把它放于不正确的位置。
```

l icon icon元素指出IDE和GUI工具用来表示Web应用的一个和两个图像文件的位置。
l display-name display-name元素提供GUI工具可能会用来标记这个特定的Web应用的一个名称。
l description description元素给出与此有关的说明性文本。
l context-param context-param元素声明应用范围内的初始化参数。
l filter 过滤器元素将一个名字与一个实现javax.servlet.Filter接口的类相关联。
l filter-mapping 一旦命名了一个过滤器，就要利用filter-mapping元素把它与一个或多个servlet或JSP页面相关联。
l listener servlet API的版本2.3增加了对事件监听程序的支持，事件监听程序在建立、修改和删除会话或servlet环境时得到通知。Listener元素指出事件监听程序类。
l servlet 在向servlet或JSP页面制定初始化参数或定制URL时，必须首先命名servlet或JSP页面。Servlet元素就是用来完成此项任务的。
l servlet-mapping 服务器一般为servlet提供一个缺省的URL：http://host/webAppPrefix/servlet/ServletName。但是，常常会更改这个URL，以便servlet可以访问初始化参数或更容易地处理相对URL。在更改缺省URL时，使用servlet-mapping元素。
l session-config 如果某个会话在一定时间内未被访问，服务器可以抛弃它以节省内存。可通过使用HttpSession的setMaxInactiveInterval方法明确设置单个会话对象的超时值，或者可利用session-config元素制定缺省超时值。
l mime-mapping 如果Web应用具有想到特殊的文件，希望能保证给他们分配特定的MIME类型，则mime-mapping元素提供这种保证。
l welcom-file-list welcome-file-list元素指示服务器在收到引用一个目录名而不是文件名的URL时，使用哪个文件。
l error-page error-page元素使得在返回特定HTTP状态代码时，或者特定类型的异常被抛出时，能够制定将要显示的页面。
l taglib taglib元素对标记库描述符文件（Tag Libraryu Descriptor file）指定别名。此功能使你能够更改TLD文件的位置，而不用编辑使用这些文件的JSP页面。
l resource-env-ref resource-env-ref元素声明与资源相关的一个管理对象。
l resource-ref resource-ref元素声明一个资源工厂使用的外部资源。
l security-constraint security-constraint元素制定应该保护的URL。它与login-config元素联合使用
l login-config 用login-config元素来指定服务器应该怎样给试图访问受保护页面的用户授权。它与sercurity-constraint元素联合使用。
l security-role security-role元素给出安全角色的一个列表，这些角色将出现在servlet元素内的security-role-ref元素的role-name子元素中。分别地声明角色可使高级IDE处理安全信息更为容易。
l env-entry env-entry元素声明Web应用的环境项。
l ejb-ref ejb-ref元素声明一个EJB的主目录的引用。
l ejb-local-ref ejb-local-ref元素声明一个EJB的本地主目录的应用。

```
##  分配名称和定制的URL 
在 web.xml 中完成的一个最常 见 的任 务 是 对 servlet 或 JSP 页 面 给 出名称和定制的 URL 。 用 servlet 元素分配名称，使用 servlet-mapping 元素将定制的 URL 与 刚 分配的名称相 关联 。 
```

<servlet>
    <servlet-name>Test</servlet-name>
    <servlet-class>moreservlets.TestServlet</servlet-class>
</servlet>
<!-- ... -->
<servlet-mapping>
    <servlet-name>Test</servlet-name>
    <url-pattern>/UrlTest</url-pattern>
</servlet-mapping>

```
##  初始化和预装载servlet与JSP页面 
```

//提供应用范围内的初始化参数
<context-param>
    <param-name>support-email</param-name>
    <param-value>blackhole@mycompany.com</param-value>
</context-param>
  
<servlet>
<servlet-name>InitTest</servlet-name>
<servlet-class>moreservlets.InitServlet</servlet-class>
  //启动初始化，优先级为1
 <load-on-startup>1</load-on-startup>
    <init-param>
        <param-name>param1</param-name>
        <param-value>value1</param-value>
    </init-param>
    <init-param>
        <param-name>param2</param-name>
        <param-value>2</param-value>
    </init-param>
</servlet>


```
##  声明过滤器 
```

<filter>
    <filter-name>Reporter</filter-name>
    <filter-class>moresevlets.ReportFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>Reporter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```
##  指定欢迎页 
```

<welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>index.html</welcome-file>
</welcome-file-list>

```
##  指定处理错误的页面 
 **error-page** 元素就是用来克服 这 些 问题 的。它有两个可能的子元素，分 别 是： **error-code 和 exception- type** 。第一个子元素 error-code 指出在 给 定的 HTTP 错误 代 码 出 现时 使用的 URL 。第二个子元素 excpetion-type 指出在出 现 某个 给 定的 Java 异常但未捕捉到 时 使用的 URL 。** error-code 和 exception-type 都利用 location 元素指出相 应 的 URL** 。**此  URL 必 须 以 / 开 始**。 location 所指出的位置 处 的 页 面可通 过查 找 HttpServletRequest 对 象的两个 专门 的属性来 访问关 于 错误 的 信息， 这 两个属性分 别 是：** javax.servlet.error.status_code 和 javax.servlet.error.message** 。 
可回 忆 一下，在 web.xml 内以正确的次序声明 web-app 的子元素很重要。 这 里只要 记 住， error-page 出 现 在 web.xml 文件的末尾附近， servlet 、 servlet-name 和 welcome-file-list 之后即可。 
##  指定应用事件监听程序 
```

<listener>
<listener-class>package.ListenerClass</listener-class>
</listener>

```
参考：
http://my.oschina.net/JiangTun/blog/300183