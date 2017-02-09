title: jsp页面布局框架sitemesh 

#   jsp页面布局框架Sitemesh 3 的使用及配置 
原文:http://www.cnblogs.com/luotaoyeah/p/3776879.html
Sitemesh 是一个网页布局和修饰的框架，基于 Servlet 中的 Filter，类似于 ASP.NET 中的‘母版页’技术。参考：百度百科，相关类似技术：Apache Tiles。
官网：http://wiki.sitemesh.org/wiki/display/sitemesh/Home 
![](/data/dokuwiki/opensourcelearn/pasted/20150603-160445.png)
最新版本：3.0.1
① GitHub 地址：https://github.com/sitemesh/sitemesh3
② maven：
```

<dependency>
  <groupId>org.sitemesh</groupId>
  <artifactId>sitemesh</artifactId>
  <version>3.0.1</version>
</dependency>

```
##   配置 Sitemesh 3 过滤器 
```

<web-app>

  ...

  <filter>
    <filter-name>sitemesh</filter-name>
    <filter-class>org.sitemesh.config.ConfigurableSiteMeshFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>sitemesh</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  
</web-app>

```
##  准备两个页面：demo.html 和 decorator.html 
① demo.html - “被装饰的页面”，实际要呈现的内容页。
```

<!DOCTYPE html>
<html>
<head>
    <title>内容页的标题</title>
</head>
<body>
    内容页的body部分
</body>
</html>

```
② decorator.html - “装饰页面”，所谓的“母版页”。
```

<!DOCTYPE html>
<html>
<head>
<title>
    <sitemesh:write property='title' /> - ltcms
</title>
<sitemesh:write property='head' />
</head>
<body>
    <header>header</header>
    <hr />
    demo.html的title将被填充到这儿：
    <sitemesh:write property='title' /><br />
    demo.html的body将被填充到这儿：
    <sitemesh:write property='body' />
    <hr />
    <footer>footer</footer>
</body>
</html>

```
##  添加 /WEB-INF/sitemesh3.xml 
```

<?xml version="1.0" encoding="UTF-8"?>
<sitemesh>
    <!-- 指明满足“/*”的页面，将被“/WEB-INF/views/decorators/decorator.html”所装饰 -->
    <mapping path="/*" decorator="/WEB-INF/views/decorators/decorator.html" />

    <!-- 指明满足“/exclude.jsp*”的页面，将被排除，不被装饰 -->
    <mapping path="/exclude.jsp*" exclude="true" />
</sitemesh>

```
![](/data/dokuwiki/opensourcelearn/pasted/20150603-160702.png)
##  sitemesh3.xml 配置详解 
```

<sitemesh>
    <!--默认情况下，
        sitemesh 只对 HTTP 响应头中 Content-Type 为 text/html 的类型进行拦截和装饰，
        我们可以添加更多的 mime 类型-->
  <mime-type>text/html</mime-type>
  <mime-type>application/vnd.wap.xhtml+xml</mime-type>
  <mime-type>application/xhtml+xml</mime-type>
  ...
  
  <!-- 默认装饰器，当下面的路径都不匹配时，启用该装饰器进行装饰 -->
  <mapping decorator="/default-decorator.html"/>
  
  <!-- 对不同的路径，启用不同的装饰器 -->
  <mapping path="/admin/*" decorator="/another-decorator.html"/>
  <mapping path="/*.special.jsp" decorator="/special-decorator.html"/>

  <!-- 对同一路径，启用多个装饰器 -->
  <mapping>
    <path>/articles/*</path>
    <decorator>/decorators/article.html</decorator>
    <decorator>/decorators/two-page-layout.html</decorator>
    <decorator>/decorators/common.html</decorator>
  </mapping>

  <!-- 排除，不进行装饰的路径 -->
  <mapping path="/javadoc/*" exclude="true"/>
  <mapping path="/brochures/*" exclude="true"/>
  
  <!-- 自定义 tag 规则 -->
  <content-processor>
    <tag-rule-bundle class="com.something.CssCompressingBundle" />
    <tag-rule-bundle class="com.something.LinkRewritingBundle"/>
  </content-processor>
  ...

</sitemesh>

```
##  自定义 tag 规则 
Sitemesh 3 默认只提供了 body，title，head 等 tag 类型，我们可以通过实现 TagRuleBundle 扩展自定义的 tag 规则：
```

public class MyTagRuleBundle implements TagRuleBundle {
    @Override
    public void install(State defaultState, ContentProperty contentProperty,
            SiteMeshContext siteMeshContext) {
        defaultState.addRule("myHeader", new ExportTagToContentRule(contentProperty.getChild("myHeader"), false));
        
    }
    
    @Override
    public void cleanUp(State defaultState, ContentProperty contentProperty,
            SiteMeshContext siteMeshContext) {
    }
}

```
最后在 sitemesh3.xml 中配置即可：
```

<content-processor>
    <tag-rule-bundle class="com.lt.common.ext.sitemesh3.MyTagRuleBundle" />
 </content-processor>

```