title: springmvc之国际化 

#  springmvc之国际化 

更多信息可以参考：http://www.cnblogs.com/liukemng/p/3750117.html

国际化需要懂得：i18n
Locale locale=new Locale("zh","CN");
Locale locale=Locale.getDefault();
文本隔离成属性文件。中文转换为Unicode码
文件命名格式messgae_zh_CN.properties
选择和读取正确的属性文件：
ResourceBundle rb=ResourceBundle.getBundle("message",Locale.CN);
rb.getString(key);
当然SpringMVC不会让你去使用这些java原生API。
SpringMVC代码方式提取国际化文本方式为:
```

import org.springframework.web.servlet.support.RequestContext;
RequestContext requestContext = new RequestContext(httpRequest);
requestContext.getMessage("money");

```
...
##  第一步，配置bean让spring直到属性文件所在 
```

<bean id="messageSource"
		class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
		<property name="basenames" >
			<list>
				<value>/WEB-INF/resource/messages</value>
				<value>/WEB-INF/resource/labels</value>
                                <value>classpath:message2</value>
			</list>
		</property>
	</bean>

```
##  第二步，配置语言区域解析器 
此处采用最简单的AcceptHeaderLocaleResolver
```

    <bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver">
    </bean>

```
##  第三步，使用spring的message标签 
```

<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
<spring:message code="form.name" text="默认值"/>

```
还可以参数化资源文件：如下：
message_zh_CN.properties
hello=hello1\u4F60\u597D{0}
 
message_en_US.properties
hello=english{0}

<spring:message code="hello" ` arguments `="111,222" ` argumentSeparator `=",">
**备注：arguments是用来给资源文件添加参数的，argumentSeparator是用来分割多个参数的标记** 
页面显示内容。hello1你好111