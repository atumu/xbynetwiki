title: springmvc_httpmessageconverter与restful 

#  SpringMVC HttpMessageConverter与Restful 
参考：
https://www.ibm.com/developerworks/cn/web/wa-restful/
关于版本说明：
SpringMVC3.x 关于JSON的支持，只支持Jackson,但是我们可以用fastJson替代,Gson没有实现HttpMessageConverter的接口。
**SpringMVC 4.x已经支持Gson了。**
本文基于` SpringMVC3.x `

##  Spring MVC 中的 REST 支持 

本部分提供了支持 RESTful web 服务的主要 Spring 功能（或注释）的概述。
` @Controller `
使用 @Controller 注释对将成为 MVC 中控制器的类进行注释并处理 HTTP 请求。
` @RequestMapping `
使用 @RequestMapping 注释对函数进行注释，该函数处理某些 HTTP 方法、URI 或 HTTP 头。此注释是 Spring REST 支持的关键。可以更改 method 参数以处理其他 HTTP 方法。
例如：
@RequestMapping(method=RequestMethod.GET, value="/emps", 
` headers `="Accept=application/xml, application/json")指客户端请求头中包含可接受类型

` @PathVariable `
使用 @PathVariable 注释可将 URI 中的路径变量作为参数插入。
例如：
@RequestMapping(method=RequestMethod.GET, value="/emp/{id}")
public ModelAndView getEmployee(@PathVariable String id) { … }

其他有用的注释
使用 ` @RequestParam ` 将 URL 参数插入方法中。
使用`  @RequestHeader ` 将某一 HTTP 头插入方法中。
使用 ` @RequestBody ` 将 HTTP 请求正文插入方法中。
使用`  @ResponseBody ` 将内容或对象作为 HTTP 响应正文返回。
使用 HttpEntity<T> 将它自动插入方法中，如果将它作为参数提供。
使用 ResponseEntity<T> 返回具有自定义状态或头的 HTTP 响应。
例如：
public`  @ResponseBody Employee ` getEmployeeBy(''@RequestParam("name") 
String name` , ` @RequestHeader("Accept") String accept` , ` @RequestBody String body'') {…} 
public ResponseEntity<String> method(HttpEntity<String> entity) {…}
参见 Spring 文档（参见 参考资料) 获得可插入方法中的支持注释或对象的完整列表。
多具象支持
使用不同 MIME 类型表示同一资源是 RESTful web 服务的一个重要方面。通常，可以使用具有不同 "accept" HTTP 头的同一 URI 提取具有不同表示的资源。还可以使用不同的 URI 或具有不同请求参数的 URI。
“使用 Spring 3 构建 RESTful web 服务”（参见 参考资料）介绍了 ` ContentNegotiatingViewResolver `，可以**挑选不同的视图解析器处理**同一 URI（具有不同的 accept 头）。因此，ContentNegotiatingViewResolver 可用于生成多个具象。
还有另一种方式可生成多具象 — 将`  HttpMessageConverter 和 c@ResponseBody 注释结合起来使用。使用这种方法无需使用视图技术。 `

##  HttpMessageConverter 
HTTP 请求和响应是基于文本的，意味着浏览器和服务器通过交换**原始文本**进行通信。但是，使用 Spring，controller 类中的方法返回纯 'String' 类型和域模型（或其他 Java 内建对象）。
**如何将对象序列化/反序列化为原始文本？这由 HttpMessageConverter 处理。**Spring 具有捆绑实现，可满足常见需求。表 1 显示了一些示例。
表 1. HttpMessageConverter 示例
  * ` StringHttpMessageConverter `	注意：**默认情况下他的编码为ISO-8859-1,会导致返回的中文乱码**。解决方式见:[[spring:springmvc_json乱码]].只要**@ResponseBody返回类型为String都会通过它处理**.从请求和响应读取/编写字符串。默认情况下，它支持媒体类型 text/* 并使用文本/无格式内容类型编写。
  * FormHttpMessageConverter	从请求和响应读取/编写表单数据。默认情况下，它读取媒体类型 application/x-www-form-urlencoded 并将数据写入 MultiValueMap<String,String>。
  * MarshallingHttpMessageConverter	使用 Spring 的 marshaller/un-marshaller 读取/编写 XML 数据。它转换媒体类型为 application/xml 的数据。
  * ` MappingJacksonHttpMessageConverter `	使用 Jackson 的 ObjectMapper 读取/编写 JSON 数据。它转换媒体类型为 application/json 的数据。
  * AtomFeedHttpMessageConverter	使用 ROME 的 Feed API 读取/编写 ATOM 源。它转换媒体类型为 application/atom+xml 的数据。
  * RssChannelHttpMessageConverter	使用 ROME 的 feed API 读取/编写 RSS 源。它转换媒体类型为 application/rss+xml 的数据。

##  配置 HttpMessageConverter 

首先，您必须配置 HttpMessageConverter。要生成多个具象，自定义几个 HttpMessageConverter 实例，以将对象转换为不同的媒体类型。此部分包括 JSON、ATOM 和 XML 媒体类型。
JSON
从最简单的示例开始。JSON 是一个轻量型的数据交换格式，人们可轻松地进行读取和编写。清单 1 显示了配置 JSON converter 的代码。
首先加入` Jackson `依赖：
```

<jackson.version>1.8.9</jackson.version>

<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-core-lgpl</artifactId>
  <version>${jackson.version}</version>
</dependency>
<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-lgpl</artifactId>
  <version>${jackson.version}</version>
</dependency>

```
```

<!-- springmvc传json值时的乱码解决 -->
  <mvc:annotation-driven>
      <mvc:message-converters>
          <bean class="org.springframework.http.converter.StringHttpMessageConverter">
              <property name="supportedMediaTypes">
                  <list>
                      <value>application/json;charset=UTF-8</value>
                  </list>
              </property>
          </bean>

      </mvc:message-converters>
  </mvc:annotation-driven>

```
##  配置` fastJson `来作为Json处理器 

```

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.7</version>
</dependency>

```
```

<!-- springmvc传json值时的乱码解决 -->
  <mvc:annotation-driven>
      <mvc:message-converters>
          <bean class="org.springframework.http.converter.StringHttpMessageConverter">
              <property name="supportedMediaTypes">
                  <list>
                    <value>text/html;charset=UTF-8</value>
                      <value>application/json;charset=UTF-8</value>
                  </list>
              </property>
          </bean>
 <!-- 配置Fastjson支持 -->
            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
              <!-- 避免IE执行AJAX时,返回JSON出现下载文件 -->
                <property name="supportedMediaTypes">
                    <list>
                      <!-- 这里顺序不能反，一定先写text/html,不然ie下出现下载提示 -->
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
              </bean>
      </mvc:message-converters>
  </mvc:annotation-driven>

```
**清单 2. 处理在 EmployeeController 中定义的 JSON 请求**
```

@RequestMapping(method=RequestMethod.GET, value="/emp/{id}", 
		headers="Accept=application/json")
public @ResponseBody Employee getEmp(@PathVariable String id) {
Employee e = employeeDS.get(Long.parseLong(id));
return e;
}
	
@RequestMapping(method=RequestMethod.GET, value="/emps", 
		headers="Accept=application/json")
public @ResponseBody EmployeeListinggetAllEmp() {
List<Employee> employees = employeeDS.getAll();
EmployeeListinglist = new EmployeeList(employees);
return list;
}

```
@ResponseBody 注释用于将返回对象（Employee 或 EmployeeList）变为响应的正文内容，将使用 MappingJacksonHttpMessageConverter 将其映射到 JSON。
使用 HttpMessageConverter 和 @ResponseBody，您可以实现多个具象，而无需包含 Spring 的视图技术 — 这是使用 ContentNegotiatingViewResolver 所不具有的一个优势。

更多请参考这两篇文章：
[使用 Spring 3 来创建 RESTful Web Services](http://www.ibm.com/developerworks/cn/web/wa-spring3webserv/)
[使用 Spring 3 MVC HttpMessageConverter 功能构建 RESTful web 服务](https://www.ibm.com/developerworks/cn/web/wa-restful/)