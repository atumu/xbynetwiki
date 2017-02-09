title: springmvc返回json 

#  SpringMVC返回JSON 
参考：http://blog.icoolxue.com/springmvc-3%E4%BD%BF%E7%94%A8fastjson%E4%BB%A3%E6%9B%BFjackson/
http://my.oschina.net/haopeng/blog/324934
由于SpringMVC**默认采用Jackson解析Json**,所以首先添加Jackson相关依赖。
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

然后进行如下步骤：
1、配置`  <mvc:annotation-driven/> `启用
2、controller 配置` @ResponseBody ` . @RequestBody注解的命令对象的转换，Spring会根据相应的HttpMessageConverter进行模型数据（处理方法的返回值）到JSON响应内容的转换。跳过View解析直接输出到客户端。
<note tip>注意：使用@ResponseBody时不要忘记在@ReuqestMapping的属性中添加` headers = "Accept=application/json" `
注意：使用@RequestBody不要忘记在@ReuqestMapping的属性中添加` headers = "Content-Type=application/json" `</note>
原因：**当使用@RequestBody和@ResponseBody注解时，RequestMappingHandlerAdapter就使用HttpMessageConverter来进行读取或者写入相应格式的数据。**
**HttpMessageConverter匹配过程：**
` @RequestBody注解时 `： 根据Request对象header部分的` Content-Type `类型，**逐一匹配合适的HttpMessageConverter来读取数据**；
` @ResponseBody注解时 `： 根据Request对象header部分的` Accept属性 `（逗号分隔），**逐一按accept中的类型，去遍历找到能处理的**

```

@Controller  
public class LoginController {  
    @RequestMapping(value="/validataUser.json" headers = "Accept=application/json")  
    @ResponseBody  
    public Map<String,Object> validataUser(@RequestParam String userName){  
        logger.info(" validata user : {}",userName);  
        Map<String,Object> map = new HashMap<String,Object>();  
        map.put("code", true);  
        return map;  
    }  
}  

```

##  使用FastJson替换Jackson 
首先别忘了添加Fastjson的包，如果使用Maven，可使用如下设置
```

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.7</version>
</dependency>

```
```

    <!-- 启用默认配置 -->
    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="true">
            <!-- 配置Fastjson支持 -->
            <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
              <!-- 避免IE执行AJAX时,返回JSON出现下载文件 -->
                <property name="supportedMediaTypes">
                    <list>
                      <!-- 这里顺序不能反，一定先写text/html,不然ie下出现下载提示 -->
                        <value>text/html;charset=UTF-8</value>
                        <value>application/json</value>
                    </list>
                </property>
             <!-- 
                <property name="features">
                    <list>
                        <value>WriteMapNullValue</value>
                        <value>QuoteFieldNames</value>
                    </list>
                </property>
		-->
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

```

##  @RequestBody, @ResponseBody详解 
详细介绍下@RequestBody、@ResponseBody的具体用法和使用时机；
**@RequestBody**
作用： 
i) 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；
ii) 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。
使用时机：
A) GET、POST方式提时， 根据**request header Content-Type**的值来判断:
<note tip>注意：使用@RequestBody不要忘记在@ReuqestMapping的属性中添加` headers = "Content-Type=application/json" `</note>
```

 @RequestMapping(value = "/{username}", method = RequestMethod.PUT, 
                  headers = "Content-Type=application/json")   // headers = "Content-Type=application/json"
  @ResponseStatus(HttpStatus.NO_CONTENT)
  public void updateSpitter(@PathVariable String username, 
                        @RequestBody Spitter spitter) {  // @RequestBody
    spitterService.saveSpitter(spitter);
  }

```
  * application/x-www-form-urlencoded， **可选（即非必须，因为这种情况的数据` @RequestParam, @ModelAttribute `也可以处理**，当然@RequestBody也能处理）；
  * multipart/form-data, **不能处理**（即使用@RequestBody不能处理这种格式的数据）；
  * 其他格式， 必须（其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理）；
B) PUT方式提交时， 根据request header Content-Type的值来判断:
application/x-www-form-urlencoded， 必须；
multipart/form-data, 不能处理；
其他格式， 必须；
说明：request的body部分的数据编码格式由header部分的Content-Type指定；

**@ResponseBody**
作用： 该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
使用时机： 返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；
<note tip>注意：使用@ResponseBody时不要忘记在@ReuqestMapping的属性中添加` headers = "Accept=application/json" `</note>
```

 @RequestMapping(value = "/{username}/spittles", 
                  method = RequestMethod.GET, 
                  headers = "Accept=application/json")//headers = "Accept=application/json"
  public @ResponseBody   List<Spittle> getSpittlesForSpitter(@PathVariable String username) {  //@ResponseBody   List<Spittle>
    return spitterService.getSpittlesForSpitter(username);
  }

```

##  HttpMessageConverter 
该接口定义了四个方法，分别是读取数据时的 canRead(), read() 和 写入数据时的canWrite(), write()方法。
在使用`  <mvc:annotation-driven /> `标签配置时：
  * 1、默认配置了` RequestMappingHandlerAdapter `（注意SpringMVC3.1之后是RequestMappingHandlerAdapter不是AnnotationMethodHandlerAdapter,详情查看Spring 3.1 document “16.14 Configuring Spring MVC”章节），
  * 2、并为他配置了一下默认的` HttpMessageConverter `：

ByteArrayHttpMessageConverter: 负责读取二进制格式的数据和写出二进制格式的数据；
` StringHttpMessageConverter `：   负责读取字符串格式的数据和写出二进制格式的数据；

ResourceHttpMessageConverter：负责读取资源文件和写出资源文件数据； 
` FormHttpMessageConverter `：       负责读取form提交的数据（能读取的数据格式为 application/x-www-form-urlencoded，不能读取multipart/form-data格式数据）；负责写入application/x-www-from-urlencoded和multipart/form-data格式的数据；

` MappingJacksonHttpMessageConverter `:  负责读取和写入json格式的数据；

SouceHttpMessageConverter：                   负责读取和写入 xml 中javax.xml.transform.Source定义的数据；
Jaxb2RootElementHttpMessageConverter:  负责读取和写入xml 标签格式的数据；

AtomFeedHttpMessageConverter:              负责读取和写入Atom格式的数据；
RssChannelHttpMessageConverter:           负责读取和写入RSS格式的数据；

**当使用@RequestBody和@ResponseBody注解时，RequestMappingHandlerAdapter就使用它们来进行读取或者写入相应格式的数据。**

**HttpMessageConverter匹配过程：**
` @RequestBody注解时 `： 根据Request对象header部分的` Content-Type `类型，**逐一匹配合适的HttpMessageConverter来读取数据**；
` @ResponseBody注解时 `： 根据Request对象header部分的` Accept属性 `（逗号分隔），**逐一按accept中的类型，去遍历找到能处理的**

##  关于直接返回json格式字符串的注意： 
MappingJacksonHttpMessageConverter 调用了 objectMapper.writeValue(OutputStream stream, Object)方法，使用@ResponseBody注解返回的对象就传入Object参数内。
方式一、**若返回的对象为已经格式化好的json串时，不使用@ResponseBody注解，而应该这样处理：**
''1、response.setContentType("application/json; charset=UTF-8");
2、response.getWriter().print(jsonStr);''
直接输出到body区，**然后的视图为void。**

方式二、不过**也可以使用@ResponseBody**，不过需要进行乱码配置：
直接返回字符串，使用@ResponseBody注解时，后台会调用` StringHttpMessageConverter `，它默认的字符集为ISO-8859-1，这会导致中文乱码。可以参考[[spring:springmvc_json乱码]]进行配置即可。

参考：
http://blog.csdn.net/kobejayandy/article/details/12690555