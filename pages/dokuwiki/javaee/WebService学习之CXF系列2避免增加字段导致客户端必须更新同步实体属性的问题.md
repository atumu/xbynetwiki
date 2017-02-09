createAt:2017-02-08 16:14:58
author:xbynet
modifyAt:2017-02-08 16:14:58
location:dokuwiki/javaee/WebService学习之CXF系列2避免增加字段导致客户端必须更新同步实体属性的问题
title:WebService学习之CXF系列2避免增加字段导致客户端必须更新、同步实体属性的问题

在使用cxf实现webservice时,经常碰到的问题就是如果在服务端,修改了一个接口的签名实现，如增加一个字段,或者删除一个字段。在这种情况下，在默认的配置中，就会报以下的错误信息：
org.apache.cxf.interceptor.Fault: Unmarshalling Error: unexpected element . Expected elements are 
这种错误即客户端使用的传输对象与服务端接收的参数的字段不匹配。但如果，每次修改服务端的实现，都需要更新客户端时，就会出现一些问题，如在某些情况下，客户端的更新是不可能的事（如不在自己掌握之内，或者服务不能随便更新，或者其它计划时）。

如果避免这种问题，其实也很简单，就是禁用cxf中的字段信息验证，如果禁用掉此验证，就不再会对相应的字段信息进行验证，同时没有的字段也会自动的忽略。整个解决只需要增加以下的一行配置即可，在cxf.xml(spring集成文件)中增加以下配置项：
```
<cxf:properties> 
   <entry key="set-jaxb-validation-event-handler" value="false"/> 
</cxf:properties> 
```
这样，即会禁用掉所有cxf的数据验证，在大多数情况下，这可以满足我们的要求(除非你有其它和cxf集成的数据验证要求)。


参考：http://www.cnblogs.com/hoojo/p/cxf_webservice_Unmarshalling_unexpected_element.html