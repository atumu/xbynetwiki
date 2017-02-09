title: struts2配置 

#  struts2InAction之Struts2配置 
##  基本讲解 

```

<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN" "http://struts.apache.org/dtds/struts-2.0.dtd" >
<struts>

    <!-- include节点是struts2中组件化的方式 可以将每个功能模块独立到一个xml配置文件中 然后用include节点引用 -->
    <include file="struts-default.xml"></include>
    
    
    <!-- package提供了将多个Action组织为一个模块的方式
        package的名字必须是唯一的 package可以扩展 当一个package扩展自
        另一个package时该package会在本身配置的基础上加入扩展的package
        的配置 父package必须在子package前配置 
        name：package名称
        extends:继承的父package名称
        abstract:设置package的属性为抽象的 抽象的package不能定义action 值true:false
        namespace:定义package命名空间 该命名空间影响到url的地址，例如此命名空间为/test那么访问是的地址为http://localhost:8080/struts2/test/XX.action
     -->
    <package name="com.kay.struts2" extends="struts-default" namespace="/test">
        <interceptors>
            <!-- 定义拦截器 
                name:拦截器名称
                class:拦截器类路径
             -->
            <interceptor name="timer" class="com.kay.timer"></interceptor>
            <interceptor name="logger" class="com.kay.logger"></interceptor>
            <!-- 定义拦截器栈 -->
            <interceptor-stack name="mystack">
                <interceptor-ref name="timer"></interceptor-ref>
                <interceptor-ref name="logger"></interceptor-ref>
            </interceptor-stack>
        </interceptors>
        
        <!-- 定义默认的拦截器 每个Action都会自动引用
         如果Action中引用了其它的拦截器 默认的拦截器将无效 -->
        <default-interceptor-ref name="mystack"></default-interceptor-ref>
        
        
        <!-- 全局results配置 -->
        <global-results>
            <result name="input">/error.jsp</result>
        </global-results>
        
        <!-- Action配置 一个Action可以被多次映射(只要action配置中的name不同)
             name：action名称
             class: 对应的类的路径
             method: 调用Action中的方法名
        -->
        <action name="hello" class="com.kay.struts2.Action.LoginAction">
            <!-- 引用拦截器
                name:拦截器名称或拦截器栈名称
             -->
            <interceptor-ref name="timer"></interceptor-ref>
        
            <!-- 节点配置
                name : result名称 和Action中返回的值相同
                type : result类型 不写则选用superpackage的type struts-default.xml中的默认为dispatcher
             -->
         <result name="success" type="dispatcher">/talk.jsp</result>
         <!-- 参数设置 
             name：对应Action中的get/set方法 
         -->
         <param name="url">http://www.sina.com</param>
        </action>
    </package>
</struts>

```


##  常见常量配置介绍 
```

<constant name="struts.i18n.encoding" value="UTF-8"/>
指定Web应用的默认编码集，相当于调用HttpServletRequest的setCharacterEncoding方法

<constant name="struts.action.extension" value="do"/>
该属性指定需要Struts 2处理的请求后缀，该属性的默认值是action，即所有匹配*.action的请求都由Struts 2处理。    如果用户需要指定多个请求后缀，则多个后缀之间以英文逗号（，）隔开。   

<constant name="struts.serve.static.browserCache " value="false"/>
设置浏览器是否缓存静态内容，默认值为true，开发阶段最好false

<constant name="struts.configuration.xml.reload" value="true"/>
当struts的配置文件修改后，系统是否自动重新加载该文件，默认值为false，开发阶段最好true

<constant name="struts.devMode" value="true"/>
开发模式下设为true，这样可以打印出更详细的错误信息

<constant name="struts.enable.DynamicMethodInvocation" value="false"/>
动态方法调用,可以解决多个请求对应一个Servlet的问题,后面详细讲解,默认为true,关闭则设为false.

这里只是列举了一些常用的开关,当然还有许多其他的开关,后面的学习中会逐渐介绍,大家在这里先了解一下.

以下是从网上摘得的,比较全的一个资料
struts.serve.static.browserCache 该属性设置浏览器是否缓存静态内容。当应用处于开发阶段时，我们希望每次请求都获得服务器的最新响应，则可设置该属性为false。

struts.enable.DynamicMethodInvocation 该属性设置Struts 2是否支持动态方法调用，该属性的默认值是true。如果需要关闭动态方法调用，则可设置该属性为false。

struts.enable.SlashesInActionNames 该属性设置Struts 2是否允许在Action名中使用斜线，该属性的默认值是false。如果开发者希望允许在Action名中使用斜线，则可设置该属性为true。

struts.tag.altSyntax 该属性指定是否允许在Struts 2标签中使用表达式语法，因为通常都需要在标签中使用表达式语法，故此属性应该设置为true，该属性的默认值是true。

struts.devMode该属性设置Struts 2应用是否使用开发模式。如果设置该属性为true，则可以在应用出错时显示更多、更友好的出错提示。该属性只接受true和flase两个值，该属性的默认值是false。通常，应用在开发阶段，将该属性设置为true，当进入产品发布阶段后，则该属性设置为false。

struts.i18n.reload该属性设置是否每次HTTP请求到达时，系统都重新加载资源文件。该属性默认值是false。在开发阶段将该属性设置为true会更有利于开发，但在产品发布阶段应将该属性设置为false。

提示 开发阶段将该属性设置了true，将可以在每次请求时都重新加载国际化资源文件，从而可以让开发者看到实时开发效果；产品发布阶段应该将该属性设置为false，是为了提供响应性能，每次请求都需要重新加载资源文件会大大降低应用的性能。

struts.ui.theme该属性指定视图标签默认的视图主题，该属性的默认值是xhtml。

struts.ui.templateDir该属性指定视图主题所需要模板文件的位置，该属性的默认值是template，即默认加载template路径下的模板文件。

struts.ui.templateSuffix该属性指定模板文件的后缀，该属性的默认属性值是ftl。该属性还允许使用ftl、vm或jsp，分别对应FreeMarker、Velocity和JSP模板。

struts.configuration.xml.reload该属性设置当struts.xml文件改变后，系统是否自动重新加载该文件。该属性的默认值是false。

struts.velocity.configfile该属性指定Velocity框架所需的velocity.properties文件的位置。该属性的默认值为velocity.properties。

struts.velocity.contexts该属性指定Velocity框架的Context位置，如果该框架有多个Context，则多个Context之间以英文逗号（,）隔开。

struts.velocity.toolboxlocation该属性指定Velocity框架的toolbox的位置。

struts.url.http.port该属性指定Web应用所在的监听端口。该属性通常没有太大的用户，只是当Struts 2需要生成URL时（例如Url标签），该属性才提供Web应用的默认端口。

struts.url.https.port该属性类似于struts.url.http.port属性的作用，区别是该属性指定的是Web应用的加密服务端口。

struts.url.includeParams该属性指定Struts 2生成URL时是否包含请求参数。该属性接受none、get和all三个属性值，分别对应于不包含、仅包含GET类型请求参数和包含全部请求参数。


struts.custom.i18n.resources该属性指定Struts 2应用所需要的国际化资源文件，如果有多份国际化资源文件，则多个资源文件的文件名以英文逗号（,）隔开。


struts.dispatcher.parametersWorkaround 对于某些Java EE服务器，不支持HttpServlet Request调用getParameterMap()方法，此时可以设置该属性值为true来解决该问题。该属性的默认值是false。对于WebLogic、Orion和OC4J服务器，通常应该设置该属性为true。

struts.freemarker.manager.classname 该属性指定Struts 2使用的FreeMarker管理器。该属性的默认值是org.apache.struts2.views.freemarker.FreemarkerManager，这是Struts 2内建的FreeMarker管理器。

struts.freemarker.wrapper.altMap该属性只支持true和false两个属性值，默认值是true。通常无需修改该属性值。

struts.xslt.nocache 该属性指定XSLT Result是否使用样式表缓存。当应用处于开发阶段时，该属性通常被设置为true；当应用处于产品使用阶段时，该属性通常被设置为false。

struts.configuration.files 该属性指定Struts 2框架默认加载的配置文件，如果需要指定默认加载多个配置文件，则多个配置文件的文件名之间以英文逗号（,）隔开。该属性的默认值为struts-default.xml,struts-plugin.xml,struts.xml，看到该属性值，读者应该明白为什么Struts 2框架默认加载struts.xml文件了。 


在请求时,路径后的后缀action可要可不要,即下面的两种请求都是可以的
http://localhost:8080/Struts2/chapter1/HelloWorld
http://localhost:8080/Struts2/chapter1/HelloWorld.action

```
##  Action配置中的各项默认值 
```

<action name="Login">
<result>/WEB-INF/JspPage/chapter1/Login.jsp</result>
</action> 
我们发现,当我们请求的路径为http://localhost:8080/Struts2/chapter1/Login时,同样可以实现页面的跳转,这是怎么回事呢?

如果没有为action指定class,默认是ActionSupport类
<action name="Login"> 
相当于 
<action name="Login" class="com.opensymphony.xwork2.ActionSupport">

如果没有为action指定method,默认执行action中的execute()方法
<action name="Login">
相当于
<action name="Login" class="com.opensymphony.xwork2.ActionSupport"
method="execute">

如果没有指定result的name属性,默认值为success.
<result>
相当于
<result name="success">

```
![](/data/dokuwiki/pasted/20150723-053846.png)
 ![](/data/dokuwiki/pasted/20150723-053329.png)
![](/data/dokuwiki/pasted/20150723-053354.png)
![](/data/dokuwiki/pasted/20150723-053907.png)
![](/data/dokuwiki/pasted/20150723-053920.png)
**<default-action-ref>**
如果在请求一个没有定义过的Action资源时，系统就会抛出404错误。这种错误不可避免，但这样的页面并不友好。我们可以使用<default-action-ref>来指定一个默认的Action，如果系统没有找到指定的Action，就会指定来调用这个默认的Action。
```

<struts>
    <packagename="wwfy"extends="struts-default">
          
        <default-action-refname="acctionError"></default-action-ref>
        <actionname="acctionError">
            <result>/jsp/actionError.jsp</result>
        </action>
    </package>
</struts>

```
**<default-interceptor-ref>**
该标签用来设置整个包范围内所有Action所要应用的默认拦截器信息。事实上我们的包继承了struts-default包以后，使用的是Struts的默认设置。我们可以在struts-default.xml中找到相关配置：
<default-interceptor-ref name="defaultStack"/>
在实际开发过程中，如果我们有特殊的需求是可以改变默认拦截器配置的。当时一旦更改这个配置，“defaultStack”将不再被引用，需要手动最加。

**<exception-mapping>与<global-exception-mapping>**
这两个标签都是用来配置发生异常时对应的视图信息的,**只不过一个是Action范围的,一个是包范围的**,当同一类型异常在两个范围都被配置时,Action范围的优先级要高于包范围的优先级.这两个标签包含的属性也是一样的:
name	否	用来表示该异常配置信息
result	是	指定发生异常时显示的视图信息,这里要配置为逻辑视图
exception	是	指定异常类型
```

<struts>
    <packagename="default"extends="struts-default">
        <global-exception-mappings>
            <exception-mappingresult="逻辑视图"exception="异常类型"/>
        </global-exception-mappings>
        <actionname="Action名称">
            <exception-mappingresult="逻辑视图"exception="异常类型"/>
        </action>
    </package>
</struts>

```

##  结果类型 
![](/data/dokuwiki/pasted/20150723-053302.png)

##  struts.properties 
![](/data/dokuwiki/pasted/20150723-070903.png)
![](/data/dokuwiki/pasted/20150723-071103.png)
![](/data/dokuwiki/pasted/20150723-071151.png)
还可以参考：http://blog.csdn.net/baple/article/details/9420877