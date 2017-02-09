title: struts2学习笔记csdn 

#  struts2学习笔记CSDN 
http://blog.csdn.net/hudie1234567/article/details/6730481
依赖:
![](/data/dokuwiki/pasted/20150723-030426.png)
##  struts2简介 

struts2是在webwork2基础上发展而来的。和struts1一样，struts2也属于MVC框架。不过有一点需要注意的是：struts2和struts2虽然名字很相似，但是在两者在代码编写风格上几乎是不一样的。那么既然有了struts1，为什么还要推出struts2。主要的原因是struts2有以下优点：
1.在软件设计上struts2没有像struts1那样跟servlet API和struts API有着紧密的耦合，struts2的应用可以不依赖于servlet API和struts API。struts2的这种设计属于无侵入式设计，而struts1却属于侵入式设计。
2.struts2提供了拦截器，利用拦截器可以进行AOP编程，实现如权限拦截等功能。
3.struts2提供了类型转换器，可以把特殊的请求参数转化成需要的类型。在struts1中，如果我们要实现同样的功能，就必须向struts1的底层实现BeanUtil注册类型转换器才行。
4.struts2提供支持多种表现层技术，如：jsp、freemarker、velocity等。
5.struts2的输入校验可以对指定的方法进行校验，解决了struts1长久之痛。
6.提供了全局范围、包范围和Action范围的国际化资源文件实现。
##  struts2开发环境搭建 

搭建struts2（这里使用的是2.1.8版的）的开发环境的时，一般都会按如下的步骤：
1.引入struts2需要的jar文件（一般需要commons-fileupload-1.2.1.jar、commons-logging-1.0.4.jar、freemarker-2.3.15.jar、ognl-2.7.3.jar、struts2-core-2.1.8.1.jar和xwork-core-2.1.6.jar这6个jar文件，可以从struts2自带的示例项目中拷贝，粘贴到WebRoot/WEB-INF/lib下面）
2.编写struts2的配置文件（可以从struts2自带的示例项目中拷贝struts.xml，粘贴到src目录下，然后在这个基础上按照自己的需要来更改）
3.在web.xml中加入struts2框架启动配置，具体的方法是在web.xml中加入如下的代码：
```

<filter>
	<filter-name>struts2</filter-name>
	<filter-class>org.apache.struts2.dispatcher.FilterDispatcher</filter-class>
</filter>
	
<filter-mapping>
	<filter-name>struts2</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>

```
##  struts2配置中的包介绍 
```

<package name="upload" namespace="/upload" extends="struts-default">
	<action name="upload" class="zhchljr.action.HelloWorldAction" method="upload">
		<result name="success">/success.jsp</result>
	</action>
</package>

```
**struts2中使用包来管理action。包的作用和java中类包是非常类似的，它主要用于管理一组业务功能相关的action。在实际应用中，应该把一组业务功能相关的action放在同一个包下面。**
![](/data/dokuwiki/pasted/20150721-073436.png)

包的namespace属性用于定义包的命名空间，命名空间作为该action路径的一部分，如访问上面的例子中的action的路径为：/upload/upload.action。
通常每个包都必须继承struts-default包，因为struts很多核心的功能都是在这个包中定义的拦截器实现的，如：从请求中把请求参数封装到action、文件上传和数据验证等功能搜是通过拦截器实现的。struts-default包中定义了这些拦截器和result类型。换句话说，当包继承了strtus-default包才能使用struts提供的核心功能。struts-default包是在struts2-core-2.x.x.x.jar文件中的struts-default.xml中定义的。struts-default.xml是struts2的默认配置文件，struts2每次都会自动加载**struts-default.xml**文件。
包还可以通过abstract=“true”定义为抽象包，抽象包中不能包含action。
注意，在配置文件struts.xml中没有提示的解决办法：window->preference->xml catalog中添加struts-2.0.dtd文件，key type为URI，key为http://struts.apache.org/dtds/struts-2.0.dtd。

##  全局元素 
constant元素定义常量，修改配置。
![](/data/dokuwiki/pasted/20150721-073806.png)
![](/data/dokuwiki/pasted/20150721-073649.png)
##  struts.properties文件 
![](/data/dokuwiki/pasted/20150721-073900.png)
##  action配置中的默认值 
```

<package name="department" namespace="/department" extends="struts-default">  
    <action name="helloworld" class="zhchljr.action.HelloWorldAction" method="execute">  
        <param name="savepath">department</param>  
        <result name="success">/employeeAdd.jsp?username=${username}</result>  
    </action>  
</package>  

```
1.如果没有为action指定class，默认的class是ActionSupport。
2.如果没有为action指定method，默认执行action中的execute方法。
3.如果没有为result指定name属性，默认值为success。
##  action中result的各种转发类型 

result配置类似于struts1中的forward，但struts2提供了多种结果类型，常用的类型有dispatcher（默认值）、redirect、redirectAction、plainText。
result中还可以使用${属性名}表达式来访问action中的属性，表达式中的属性名为action中的属性名，如下：
```

<result type="redirect">/employeeAdd.jsp?id=4{id}</result>  

```
下面是结果类型为redirectAction的例子：
重定向到同一个包中的action：
<result type="redirectAction">add</result>  
重定向到别的namespace中的action：
```

<result type="redirectAction">  
    <param name="actionName">xxx</param>  
    <param name="namespace">/redirectAction</param>  
</result> 

``` 
plainText：显示原始文件内容，例如：当需要原样显示jsp文件源代码的时候，就可以使用此类型。
```

<action name="source">  
    <result type="plainText">  
        <param name="location">/index.jsp</param>  
        <param name="charSet">UTF-8</param> <!--指定读取文件的编码方式-->   
    </result>  
</action> 

```
##  为action的属性注入值 

struts2为action的属性提供了依赖注入功能，在struts2的配置文件中，可以很方便的为action中的属性值注入值。注意，属性必须有setter方法。
```

<action name="helloworld_*" class="zhchljr.action.HelloWorldAction" method="{1}">  
    <param name="savepath">/upload</param>  
    <result name="success">/employeeAdd.jsp?username=${username}</result>  
</action>

```  
上面就是通过<param>节点为action中的savePath注入"/upload"。
##  常用的常量介绍 

default.properties文件中定义了很多常量，下面就说说通常使用到的几个。
struts.i18n.encoding 指定默认编码集
struts.action.extension struts2处理的默认后缀，如果需要定义多个后缀，则用逗号“,"隔开
struts.serve.static.browserCache 设置浏览器是否缓存静态内容，默认为true（生产环境下使用），开发阶段最好关闭
struts.configuration.xml.reload 当struts的配置文件修改后，系统是否自动重新加载该文件，默认值为false（生产环境下），开发阶段最好打开。
struts.devMode 是否为开发模式，默认为false，（生产环境下），开发阶段最好打开，以便打印出更详细的信息
struts.ui.theme 视图主题，一般用simple
struts.objectFactory  于spring集成时，指定由spring负责action对象的创建
struts.enable.DynamicMethodInvocation 是否支持动态方法调用，action!methodname，默认为true
struts.multipart.maxSize 上传文件的大小

##  struts处理流程 
` struts2对用户的每一个请求都会创建一个action，所以struts2是线程安全的。 `
###  为应用指定多个struts配置文件 

在大部分应用里，随着应用规模的增加，系统中action的数量也会大量增加，导致struts.xml配置文件变的非常臃肿。为避免struts.xml文件过于庞大、臃肿，提高struts.xml文件的可读性，可以将一个struts.xml文件分解成过个配置文件，然后在struts.xml文件中包含其他的配置文件。下面的struts.xml文件通过include元素指定多个配置文件：
```

<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE struts PUBLIC  
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"  
    "http://struts.apache.org/dtds/struts-2.0.dtd">  
<struts>  
    <include file="department.xml"></include>  
    <include file="employee.xml"></include>  
    <include file="upload.xml"></include>  
    <include file="interceptor.xml"></include>  
</struts>  

```
###  动态方法调用 

在配置文件的action中不指定method，而是在访问的时候通过actionname!methodname这种方式来访问，前提是struts.enable.DynamicMethodInvocation要设置为true。
```

<package name="employee" namespace="/employee" extends="struts-default">  
    <action name="helloworld" class="zhchljr.action.HelloWorldAction">  
        <result name="success">/employeeAdd.jsp?username=${username}</result>  
    </action>  
</package>

```
假设zhchljr.action.HelloWorldAction类中有add()和update()两个方法，可以使用"/employee/helloworld!add"来访问add()方法。
###  使用通配符的定义action 

使用通配符*定义action，可以有效的减小配置文件的规模。下面就是一个简单的例子，和上面动态方法调用中的配置文件类似：
```

<package name="employee" namespace="/employee" extends="struts-default">  
    <action name="helloworld_*" class="zhchljr.action.HelloWorldAction" method="{1}">  
        <param name="savepath">employee</param>  
        <result name="success">/employeeAdd.jsp?username=${username}</result>  
    </action>  
</package>  

```
假设zhchljr.action.HelloWorldAction类中有add()和update()两个方法,这样定义后，就可以使用”/employee/helloworld_add“来访问add()方法。
##  接收请求参数 

1.**采用基本类型接收参数(get/post)**
**在action类中定义与请求参数同名的属性，并且该属性有set和get方法，struts2就能自动接收请求参数并赋予同名属性。**
请求路径：http://localhost:8080/struts2/product/view.action?id=1
2.**采用复合类型接收参数**
请求路径：http://localhost:8080/struts2/product/view.action?product.id=1

##  自定义类型转换器 
略
##  访问或添加request/session/application属性 


1.如果只是添加或访问request/session/application中的属性，就可以用ActionContext类来实现。如下代码所示：

```

ActionContext ctx = ActionContext.getContext();  
ctx.getApplication().put("app", "应用范围");//相当于往ServletContext中放入app  
ctx.getSession().put("ses", "session应用范围");//相当于往session中加入ses  
ctx.put("req", "request应用范围");//相当于往request中加入req   
ctx.put("names", Arrays.asList("刘明","茫茫大海"));

```  

2.获取HttpServletRequest/HttpSession/ServletContext/HttpServletResponse对象，可以有两种方法实现：
（1）通过ServletActionContext直接获取：

```

HttpServletRequest request = ServletActionContext.getRequest();  
ServletContext context = ServletActionContext.getServletContext();  
request.setAttribute("req", "请求范围属性");  
request.getSession().setAttribute("ses", "会话范围属性");  
context.setAttribute("app", "应用范围属性");  

```
（2）实现指定接口，由struts2框架运行时注入：

```

public class TestAction implements ServletRequestAware, ServletResponseAware,   ServletContextAware {  
    private HttpServletRequest request;  
    private HttpServletResponse response;  
    private ServletContext servletContext;  
      
    public void setServletRequest(HttpServletRequest request) {  
        this.request = request;  
    }  
  
    public void setServletResponse(HttpServletResponse response) {  
        this.response = response;  
    }  
  
    public void setServletContext(ServletContext context) {  
        this.servletContext = context;  
    }  
  
} 

```
##  文件上传 
文件上传在项目中经常会用到，下面就来说说struts2中怎么上传文件的：
1.引入相应的jar包（commons-fileupload-1.2.1.jar和commons-io-1.3.2.jar）
2.把form的enctype设置为"multipart/form-data"，如下所示：
```

<form action="<%=basePath%>upload/upload.action" method="post" name="form" enctype="multipart/form-data">  
    文件1:<input type="file" name="upload"/><br/>  
    <input type="submit" value="上传" />  
</form> 

``` 
3.在action类中添加如下代码中注释的几个属性。
```

public class HelloWorldAction {  
    private File upload;//得到上传的文件  
    private String uploadContentType;//得到上传文件的扩展名  
    private String uploadFileName;//得到上传文件的名称  
              
    public File getUpload() {  
        return upload;  
    }  
  
    public void setUpload(File upload) {  
        this.upload = upload;  
    }  
  
    public String getUploadContentType() {  
        return uploadContentType;  
    }  
  
    public void setUploadContentType(String uploadContentType) {  
        this.uploadContentType = uploadContentType;  
    }  
  
    public String getUploadFileName() {  
        return uploadFileName;  
    }  
  
    public void setUploadFileName(String uploadFileName) {  
        this.uploadFileName = uploadFileName;  
    }  
      
    public String upload() throws IOException {  
        String realpath = ServletActionContext.getServletContext().getRealPath("/upload");  
        if(upload != null) {  
            File savefile = new File(realpath,uploadFileName);  
            if(!savefile.getParentFile().exists()) {  
                savefile.getParentFile().mkdirs();  
            }  
            FileUtils.copyFile(upload, savefile);  
            ActionContext.getContext().put("msg", "文件上传成功！");  
        }  
        return "success";  
    }  
}  

```
注意，如果在上传的过程中文件的大小超过了struts2默认的文件大小的话，就会上传失败，这时候，可以根据具体的情况设置struts.multipart.maxSize的值来满足上传的需求。
###  多文件上传 
```

<form action="<%=basePath%>upload/upload" method="post" name="form" enctype="multipart/form-data">  
    文件1:<input type="file" name="upload"/><br/>  
    文件2:<input type="file" name="upload"/><br/>  
    文件3:<input type="file" name="upload"/><br/>  
    <input type="submit" value="上传" />  
</form>  

```
3.action中添加的几个属性都是数组形式的。
```

public class HelloWorldAction {  
    private File[] upload;//得到上传的文件  
    private String[] uploadContentType;//得到上传文件的扩展名  
    private String[] uploadFileName;//得到上传文件的名称  
              
    public File[] getUpload() {  
        return upload;  
    }  
  
    public void setUpload(File[] upload) {  
        this.upload = upload;  
    }  
  
    public String[] getUploadContentType() {  
        return uploadContentType;  
    }  
  
    public void setUploadContentType(String[] uploadContentType) {  
        this.uploadContentType = uploadContentType;  
    }  
  
    public String[] getUploadFileName() {  
        return uploadFileName;  
    }  
  
    public void setUploadFileName(String[] uploadFileName) {  
        this.uploadFileName = uploadFileName;  
    }  
      
    public String upload() throws IOException {  
        String realpath = ServletActionContext.getServletContext().getRealPath("/upload");  
        if(upload != null) {  
            for(int i=0; i<upload.length; i++) {  
                File savefile = new File(realpath,uploadFileName[i]);  
                if(!savefile.getParentFile().exists()) {  
                    savefile.getParentFile().mkdirs();  
                }  
                FileUtils.copyFile(upload[i], savefile);  
            }  
            ActionContext.getContext().put("msg", "文件上传成功！");  
        }  
        return "success";  
    }  
}  

```
##  拦截器 
![](/data/dokuwiki/pasted/20150721-073026.png)

自定义拦截器要实现com.opensymphony.xwork2.interceptor.Interceptor接口。下面是一个自定义拦截器的例子：
```

public class PermissionInterceptor implements Interceptor {  
  
    public void destroy() {  
          
    }  
  
    public void init() {  
  
    }  
  
    public String intercept(ActionInvocation invocation) throws Exception {  
        Object user = ActionContext.getContext().getSession().get("user");  
        if(user != null) {  
            return invocation.invoke();  
        } else {  
            ActionContext.getContext().put("message", "你没有执行权限！");  
        }  
        return "success";  
    }  
  
}  

```
接下来，就要在配置文件中注册拦截器，具体的做法是：
```

<interceptors>  
    <interceptor name="permission" class="zhchljr.interceptor.PermissionInterceptor"></interceptor>  
</interceptors> 

```
为action指定拦截器，具体的做法是：
```

<action name="interceptor" class="zhchljr.action.HelloWorldAction" method="interceptor">  
    <interceptor-ref name="permission"></interceptor-ref>  
</action>  

```
但是这样做了以后，就会出现一个问题，` struts2中为一个action指定拦截器后，默认的defaultStack中的拦截器就不起作用了，也就是说struts2的众多核心功能都使用不了了 `（struts2的许多核心功能都是通过拦截器实现的），` 为了解决这个问题，引入拦截器栈 `，先使用系统默认的拦截器，然后再来使用自定义的拦截器，具体的做法是：
```

<interceptors>  
    <interceptor name="permission" class="zhchljr.interceptor.PermissionInterceptor"></interceptor>  
    <interceptor-stack name="permissionStack">  
        <interceptor-ref name="defaultStack"></interceptor-ref>  
        <interceptor-ref name="permission"></interceptor-ref>  
    </interceptor-stack>  
</interceptors>  

```
如果希望包下的所有action都使用自定义的拦截器，可以把拦截器设置为默认拦截器，具体的实现方式是：
<default-interceptor-ref name="permissionStack"></default-interceptor-ref>  
##  输入校验 
struts2中可以实现对action中的所有方法进行校验，也可以实现对指定的方法进行校验。可以用如下两种方式来实现输入校验。
###  1.采用手工编写代码实现： 

通过编写validate()方法实现，validate()会校验action中所有与execute方法签名相同的方法。当某个数据校验失败时，应该采用addFieldError()方法往系统的filedErrors添加校验失败信息（为了使用该方法，action可以继承ActionSupport），如果系统的filedErrors包含失败信息，struts2会将请求发到名为input的result，在input视图中可以通过<s:fielderror/>显示失败信息。具体的参见下面的例子：

```

<s:fielderror></s:fielderror>  
<form method="post" action="<%=basePath%>person/manage_save.action">  
       用户名:<input type="text" name="username"/>不能为空<br/>  
       手机号:<input type="text" name="mobile"/>不能为空，并且要符合手机号的格式，1，3/5/8，后面是9个数字<br/>  
       <input type="submit" value="提交"/>  
</form> 

``` 

页面中使用<s:fielderror></s:fielderror>来显示失败信息。
```

public void validate() {//会对action中的所有方法进行校验  
    if(this.username == null || this.username.trim().equals("")) {  
        this.addFieldError("username", "用户名不能为空！");  
    }  
          
    if(this.mobile == null || this.mobile.trim().equals("")) {  
        this.addFieldError("mobile", "手机号不能为空！");  
    } else {  
        if(!Pattern.compile("^1[358]\\d{9}{1}quot;).matcher(this.mobile).matches()) {  
            this.addFieldError("mobile", "手机号格式不正确！");  
        }  
    }  
}  

```
手工编写代码实现对action中的指定方法进行校验 ：通过编写validateXxx()方法实现，validateXxx()会校验action中方法名为xxx的方法。
```

public void validateUpdate() {//会对action中的update方法进行校验  
    if(this.username == null || this.username.trim().equals("")) {  
        this.addFieldError("username", "用户名不能为空！");  
    }  
          
    if(this.mobile == null || this.mobile.trim().equals("")) {  
        this.addFieldError("mobile", "手机号不能为空！");  
    } else {  
        if(!Pattern.compile("^1[358]\\d{9}{1}quot;).matcher(this.mobile).matches()) {  
            this.addFieldError("mobile", "手机号格式不正确！");  
        }  
    }  
}  

```
###  2.基于xml配置方式来实现： 
基于xml配置方式实现对action中的所有方法进行校验：Action要继承ActionSupport，并且提供校验文件，校验文件和action类在同一个包下，文件的命名规则是：ActionClassName-validation.xml，下面是一个校验文件（PersonAction-validation.xml）的例子：
```

<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE validators PUBLIC "-//OpenSymphony Group//XWork Validator 1.0.3//EN"  
       "http://www.opensymphony.com/xwork/xwork-validator-1.0.3.dtd">  
<validators>  
    <field name="username">  
        <field-validator type="requiredstring">  
            <param name="trim">true</param><!-- 默认为true -->  
            <message>用户名不能为空！</message>  
        </field-validator>  
    </field>  
    <field name="mobile">  
        <field-validator type="requiredstring">  
            <param name="trim">true</param><!-- 默认为true -->  
            <message>手机号不能为空！</message>  
        </field-validator>  
        <field-validator type="regex">  
             <param name="expression"><![CDATA[^1[358]\d{9}$]]></param>  
            <message>手机格式不正确！</message>  
        </field-validator>  
    </field>  
</validators> 

```
说明：<field>指定action中要校验的属性，<field-validator>指定校验器，上面指定的requiredstring校验器是由系统提供的，系统提供了能满足大部分验证需求的校验器，这些校验器可以在文件xwork-core-2.X.X.jar中的com.opensymphony.xwork2.validator.validators下的default.xml中找到。<message>为校验失败后的提示信息，如果需要国际化，可以为message指定key属性，key的值为资源文件中的key。在这个配置文件中，对action中字符串类型的username属性进行验证，首先要求调用trim()去掉空格，然后判断用户名是否为空。对string类型的mobile字段的校验使用了regex这个校验器。

<2>基于xml配置方式实现对action中指定的方法进行校验：当校验文件名为ActionClassName-validation.xml时，会对action中的所有方法进行校验。如果要对action中的某一个方法进行校验，那么校验文件的名称为：ActionClassName-ActionName-validation.xml。其中ActionName为struts.xml中的action的名称。配置文件如下：
假设PersonAction中有save()和update()两个方法。对save方法进行校验的文件名为PersonAction-manage_save-validation.xml，对update方法进行校验的文件名位PersonAction-manage_update-validation.xml。
##  国际化 


1.准备资源文件，资源文件的命名格式是：
baseName_language_country.properties
其中baseName为资源文件的基本名，可以自定义。但language和country必须是java支持的语言和国家。如：
中国大陆：baseName_zh_CN.properties
美国：baseName_en_US.properties
对于中文的属性文件，编写好后，应该使用jdk自带的native2ascii命令把文件转成unicode编码的文件，命令的使用方式如下：
native2ascii src.properties dest.properties

2.配置全局资源文件
准备好资源文件后， 可以在struts.xml中通过struts.custom.i18n.resources把资源文件定义为全局资源文件，如下所示：
<constant name="struts.custom.i18n.resources" value="welcome"></constant>  
3.输出国际化信息
（1）在jsp页面中使用<s:text name=""/>标签输出国际化信息，如：<s:text name="welcome"/>，name为资源文件中的key
（2）在action类中，可以继承ActionSupport，使用getText()方法得到国际化信息，该方法的第一个参数用于指定资源文件中的key
（3）在表单标签中，通过key属性指定资源文件中的key，如：<s:textfield name="realname" key="welcome"/>

*.输出带占位符的国际化信息
资源文件中的内容为：welcome=package:{0}，欢迎来到传智播客{1}

在jsp页面输出带占位符的国际化信息
```

<s:text name="welcome">  
    <s:param>liuming</s:param>  
    <s:param>study</s:param>  
</s:text>

```
在action类中获取带占位符的国际化信息，可以使用getText(String key,String[] args)或getText(String key,List args)。
##  OGNL表达式语言 


OGNL是Object Graphic Navigation Language（对象导航图语言）的缩写，它是一个开源项目。Struts2框架使用OGNL作为默认的表达式语言。

相对于EL表达式，它提供了一些平时需要的功能。
（1）支持对象方法调用，如xxx.save()
（2）支持类静态方法调用和值访问，表达式的格式为@全类名（包括包路径）@方法名(参数)，如：@java.lang.String@format('foo%s','bar')或@java.util.Math@PI
（3）操作集合对象

Ognl有一个上下文（Context）概念，说白了上下文就是一个Map结构，它实现了java.util.Map接口，在struts2中context的实现为ActionContext，下面是它的结构示意图：
![](/data/dokuwiki/pasted/20150721-053218.png)
当struts2接受一个请求时，会迅速创建出ActionContext，ValueStack，action，然后把action存放进ValueStack，所以action的实例变量可以被ognl访问。

访问上下文中的对象需要使用#标注命名空间，如#application，#session等
。ognl有一个根对象（root对象），在struts2中根对象就是ValueStack，如果要访问根对象中对象的属性，则可以省略#命名空间，直接访问该对象的属性即可。
需要注意的一点是，**struts2中ognl表达式要配合struts标签才可以使用。**如：<s:property value="name"/>
由于ValueStack是Struts2中的ognl的根对象，如果用户需要访问ValueStack中的对象，在jsp页面可以通过下面的EL表达式访问ValueStack中对象的属性。
${foo}/ /获得ValueStack中某个对象的foo属性
**OGNL表达式的投影功能：**
除了in和not in外，ognl还允许使用某个规则获得集合对象的子集，常用的有以下三个操作符：
?：获得所有符合逻辑的元素
^：获得符合逻辑的第一个元素
$：获得符合逻辑的最后一个元素
##  struts2中的标签 
struts2中标签分为通用标签和UI标签，通用标签包含控制标签和数据标签，如下图所示：
![](/data/dokuwiki/pasted/20150721-070619.png)
UI标签包含form标签，非form标签和Ajax标签，如下图所示：
![](/data/dokuwiki/pasted/20150721-070639.png)
###  使用<s:token/>标签防止重复提交 
使用<s:token/>标签可以防止重复提交，具体的用户如下：

1.在表单中加入<s:token/>，例如：
```

<s:form action="token" namespace="/test" method="post">  
    姓名：<s:textfield name="name" label="姓名"></s:textfield>  
    <s:token></s:token>  
    <s:submit label="提交"></s:submit>  
</s:form>

```  
2.在action中配置token拦截器，如下所示：
```

<action name="token" class="zhchljr.action.PersonAction">  
    <interceptor-ref name="defaultStack"></interceptor-ref>  
    <interceptor-ref name="token"></interceptor-ref>  
    <result name="success">/WEB-INF/page/message.jsp</result>  
    <result name="invalid.token">/index.jsp</result>  
</action> 

``` 
上面加入了token拦截器和invalid.token result，因为token拦截器在会话状态的token与请求状态的token不一致时，直接返回invalid.token result。

##  使用maven建立struts2项目结构： 
mvn archetype:create  
-DgroupId=com.jpleasure
-DartifactId=login 
-DarchetypeGroupId=org.apache.struts  -DarchetypeArtifactId=struts2-archetype-starter
-DarchetypeVersion=2.0.5-SNAPSHOT -DremoteRepositories=http://people.apache.org/repo/m2-snapshot-repository
目录说明
生成的Struts2开发框架目录满足一般的maven项目，主要由以下目录组成：
src
   ├─main                              源代码目录
   │  ├─java                             java代码
   │  ├─resources                        资源文件等
   │  └─webapp                           Web目录
   │      ├─jsp                              JSP目录
   │      ├─styles                           CSS目录
   │      └─WEB-INF                          WEB-INF目录
   └─test                              测试代码目录
        ├─java                              java代码
        └─resources                         资源文件等
重要文件：
src/main/resources
    applicationContext.xml              Spring配置文件
    log4j.properties                    log4j配置文件
    struts.properties                   struts参数文件
    struts.xml                          struts配置文件
    xwork-conversion.properties         xwork参数文件
 
src/main/webapp/WEB-INF
decorators.xml 
dwr.xml                            DWR配置文件
sitemesh.xml                       SiteMesh配置文件
web.xml                            Web部署描述文件
 
进入工程目录（login目录）使用如下命令建立Eclipse工程文件
mvn eclipse:eclipse
使用如下命令打包工程
mvn package
应用程序打包完成之后可以再login/target目录中看到login.war文件，这个文件就是最终的成果文件
 
使用如下命令运行应用程序
mvn jetty:run
 
也可以将login.war拷贝到tomcat的webapps目录下来运行struts2应用程序。


