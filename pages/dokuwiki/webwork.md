title: webwork 

#  webwork 
WebWork是由OpenSymphony组织开发的，致力于组件化和代码重用的J2EE Web框架。现在的WebWork2.x前身是Rickard Oberg开发的WebWork，但现在WebWork已经被拆分成了Xwork1和WebWork2两个项目
Xwork提供了很多核心功能：前端拦截机（interceptor），运行时表单属性验证，类型转换，强大的表达式语言（OGNL – the Object Graph Notation Language），IoC（Inversion of Control依赖倒转控制）容器等。

WebWork2建立在Xwork之上，处理HTTP的请求和响应。所有的请求都会被它的前端控制器（ServletDispatcher，最新版本是FilterDispatcher）截获。
前端控制器对请求的数据进行包装，初始化上下文数据，根据配置文件查找请求URL对应的Action类，执行Action，将执行结果转发到相应的展现页面。
WebWork2支持多视图表示，视图部分可以使用JSP, Velocity, FreeMarker, JasperReports，XML等。

中软 MVC 组件（ CSS_Webwork）基于 WebWork2.1.7，对 WebWork进行删 减，保留其核心的MVC 部分，去掉了模板、 校验器部分，UI Tag UI TagUI TagUI TagUI Tag等。
![](/data/dokuwiki/pasted/20150809-055515.png)
**依赖：**
![](/data/dokuwiki/pasted/20150809-055845.png)
**部署：**
1. 将 web.xml及 webwork.tld放到工程部署目 录 WEB-INFO目录下
2. 将上述开发包放到工程部署目录 WEB-INFO/lib目
##  配置 
**WEB-INF/web.xml配置**
```

<servlet>
<servlet-name>webwork</servlet-name>
<servlet-class>com.opensymphony.webwork.dispatcher.ServletDispatcher</servlet-class>
</servlet>
<!--ww tag library-->
<taglib>
	<taglib-uri>webwork</taglib-uri> 
	<taglib-location>/WEB-INF/webwork.tld</taglib-location> 
</taglib>
	
<servlet-mapping>
	<servlet-name>webwork</servlet-name>
	<url-pattern>*.action</url-pattern>
</servlet-mapping>

```
**classpath: webwork-default.xml配置**
定义了三个常用拦截器：
modelStack、defaultStack、uploadStack：
**classpath:webwork.properties配置：**
```

#指定字符集
webwork.i18n.encoding =  UTF-8
webwork.i18n.cosEncoding = UTF-8
#文件上传处理类
webwork.multipart.parser=com.opensymphony.webwork.dispatcher.multipart.CosMultiPartRequest
#文件上传临时文件保存路径，支持相对路径或绝对路径
webwork.multipart.saveDir=\temp
#上传文件最大限制单位字节（Byte）
webwork.multipart.maxSize=99912345
#为方便开发系统每次都去检测xwork.xml是否有修改过配置为true，生产系统应为false
webwork.configuration.xml.reload=true

```
##  应用开发 

**classpath:xwork.xml主文件**
```

<xwork>
		<include file="webwork-default.xml"/>
		<include file="common.xml"/>
		<include file="user.xml"/>

</xwork>


```
webwork子文件：user.xml:
```

<xwork>
	<package name="user" extends="webwork-default" namespace="/user"> 继承自webwork-default，指定命名空间
		<default-interceptor-ref name="defaultStack"/>  定义默认拦截器
	
		<action name="getUserInfo" class="com.css.user.info.GetUserInfo">
			<result name="success" type="dispatcher">
				<param name="location">userinfo.jsp</param>
			</result>
			<result name="error" type="redirect">
				<param name="location">../doError.action</param>
			</result>
			<interceptor-ref name="defaultStack"/>
		</action>
  
		<action name="editUserInfo" class="com.css.user.info.EditUserInfo">
			<result name="success" type="dispatcher">
				<param name="location">../resultjson.jsp</param>
			</result>
			<result name="error" type="dispatcher">
				<param name="location">../resultjson.jsp</param>
			</result>
			<interceptor-ref name="modelStack"/>            
		</action>  
  
  	 <action name="uploadUserImg" class="com.css.user.info.UploadUserImg">
            <result name="success" type="dispatcher">
                <param name="location">../resultxml.jsp</param>
            </result>
            <result name="error" type="dispatcher">
                <param name="location">../resultxml.jsp</param>
            </result>       
        	<interceptor-ref name="uploadStack" />  
		</action>
		
	</package>
</xwork>

```
**result分类：**
` success、error、input、none `
**ResultType分类：**
dispatcher:转jsp
redirect：浏览器重定向到action或jsp，**会导致action执行完成后的数据丢失。**
chain:action chain.新的action将可以使用上一个ActionContext和数据
<result name="success" type="chain">
<param name="actionName">bar</param>
</result>
##  interceptor 
实际开发中最 常用拦截器 主要封装了对表单参数提交时的处理， 如象化等主要封装了对表单参数提交时的处理， 如象化等包括如下三类：
（1） 默认拦截器 <interceptor-ref name="defaultStack"/>
（2） 支持模型驱动拦截器 <interceptor-ref name="modelStack"/>
（3） 支持文件上传拦截器 <interceptor-ref name="uploadStack"/>
###  defaultStack示例 
```

<form name="form1" id="form1" action="login.action" method="post">
<input type="hidden" name="sysId" id="sysId"/><br/>
用户名：<input name="gUser.email" id="gUser.email"/><br/>
密 码：<input name="gUser.password" id="gUser.password" type="password" /><br/>
<input name="login" type="submit" value="登录"/></p>
</form>

```
```

public class LoginAction extends ActionSupport{
private Integer sysId = null;
public GUser gUser = new GUser();
public LoginAction() {
}
public String execute() {
System.out.println(gUser.getEmail());
System.out.println(sysId);
//业务处理......
return Action.SUCCESS;
}

```
###  modelStack示例 
```

<form name="form1" id="form1" action="editUserInfo.action" method="post">
<input type="hidden" name="id" id="id"/>
<div>邮箱：<input readonly name="email" id="email"/></div>
<div>姓名：<input name="realName" id="realName" /></div>
<div>电话：<input name="phone" id="phone"/></div>
<div><input type="submit" value="保存"/></div>
</form>

```
```

public class EditUserInfo extends ActionSupport implements ModelDriven{ 实现ModelDriven接口
private GUser info = new GUser(); 
public Object getModel() { ModelDriven接口的方法。
return info;
}
public EditUserInfo() {
}
protected String execute() {
System.out.println(info.getEmail());
System.out.println(info.getPhone());
//业务处理......
return Action.SUCCESS;
}

```
###  uploadStack  
```

<form name="form1" action="uploadUserImg.action" method="post" ENCTYPE="multipart/form-data" > ENCTYPE="multipart/form-data"
<input type="hidden" name="userId" id="userId" value="1" />
上传附件:
<input type="file" name="uploadFile" id="uploadFile"> name="uploadFile"
<input type="submit" value="上传">
</form>

```
```

public class UploadUserImg extends ActionSupport{
private Integer userId = null;
private File uploadFile;

```
##  标签库 
前面已经在web.xml中配置过webwork标签库了。所以这里直接引用
` <%@ taglib prefix="ww" uri="webwork"%> `
###  <ww:property /> 
说明： 获取结果的属性值 ，如果值未指定 ，将返回栈顶值 。
A：对于多层迭代循环中有相同属性值如name，可以使用[0].name，[1].name来区分
<ww:iterator value="groups">
<ww:iterator value="users">
组另名：<ww:property value="[1].name"/>
用户名：<ww:property value="[0].name"/>
</ww:iterator>
</ww:iterator>
B：从当前作用域中的某个变量名取值（#变量名）
''<ww:set name="days" value="3" scope="page" />
<ww:property value="#days"/>''
C：从session中取值（` # `session）
` <ww:property value="#session.gUser.realName"/> `

###  <ww:set /> 
说明： 将值栈中的某个 对象设置到一scope(page,application,session,stack):
<ww:set name="days" value="3" scope="page" />
<ww:property value="#days"/>
###  <ww:bean /> 
说明： 创建一个 JavaBean，初始化它的属性并放入 ` ActionContext `以便后续使用。
<ww:bean name="'com.css.core.dict.DictMan'" id="dictId" />
<ww:property value="#dictId.getDictType('d_sex',sexId).getDescription()"/>
###  <ww:if /><ww:elseif /><ww:else /> 
<ww:if test="表达式"/>
```

<ww:set name="days" value="3" scope="page" />
<ww:if test="#days==3">
<ww:property value=" @com.css.db.query.QueryCache@get('com.css.demo.model.User',1)"/>
</ww:if>
<ww:elseif test="#days>3">
<ww:property value=" @com.css.db.query.QueryCache@get('com.css.demo.model.User',2)"/>
</ww:elseif>
<ww:else>
<ww:property value="new java.text.SimpleDateFormat('yy-MM-dd HH:mm').format(loginTime)"/>
</ww:else>

```
###  <ww:iterator /> 
说明： 循环遍历任何集合对象,iterator内置的` status `对象包含几个状态：` count,first,last,even,odd `
```

<table>
<ww:iterator value="users" status="rowstatus"> status="rowstatus"
<ww:if test="#rowstatus.first"> test="#rowstatus.first"
<tr bgcolor="blue">
<td><ww:property value="realName"/></td>
<td><ww:property value="email"/></td>
</tr>
</ww:if>
<ww:elseif test="#rowstatus.last">
<tr bgcolor="yellow">
<td><ww:property value="realName"/></td>
<td><ww:property value="email"/></td>
</tr>
</ww:elseif>
  ...

```
###  jsp:include 
说明： 包含另一个页面或活动 。非完整页面
<jsp:include page="dirusermini.jsp"></jsp:include>
###  <ww:select /> 
说明： 绘制一个选项列表元素 。
```

<ww:select
name="sexId"
id="sexId"
value="2"  哪个值选中
list="#dictId.getDictType('d_sex')"
listKey="id"
listValue="description" />
转换结果：
<select name="sexId" id="sexId">
<option value="1">男</option>
<option value="2" selected>女</option>
</select>

```



参考：http://www.blogjava.net/moxie/archive/2006/10/20/76375.html
http://blog.sina.com.cn/s/blog_6f728cd40100pqso.html