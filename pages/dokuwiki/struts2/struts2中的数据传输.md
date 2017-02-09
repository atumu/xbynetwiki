title: struts2中的数据传输 

#  struts2中的数据传输 
参考：http://blog.csdn.net/li_tengfei/article/details/6098126
如何将数据从Action传输到JSP？可通过多种方式传输
**1、通过Action的属性传输**
直接给action的属性赋值，(底部其实是被作为ValueStack的root对象保存。)在转向之后的JSP中，直接用标签<s:property value=”OGNL表达式”/>取出即可。
比如：
```

public class UserAction {
    private String username;
    private Integer age;
    private boolean valid;
   
    //查看用户的详细信息
    public String detail(){
      
       username = "张三";
       age = 18;
       valid = true;
      
       return "detail";
    }

```
在detail.jsp中，引入struts2的taglib，用这些taglib来呈现数据：
```

<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <body>
username:<s:property value="username"/> <br/>
valid:<s:property value="valid"/> <br/>
age:<s:property value="age"/> <br/>
 
  </body>
</html>

```

**2、通过ActionContext传输**
可通过` ActionContext.getContext().put() `方法来传值.底部传入ValueStack的Context Map中，通过OGNL表达式 ` #属性名 ` 结合struts2标签获取
在Action的方法中：
```

    public String detail(){    
       ActionContext.getContext().put("name", "王五");     
       username = "张三";   
       ActionContext.getContext().put("username", "李四");
       return "detail";
    }

```
在JSP中：
```

  <body>
    <!-- 从ActionContext中取name的值 -->
    name: <s:property value="#name"/> <br/>
    <!-- 先看Action中有没有name属性，如果没有，则到ActionContext中找name的值 -->
    name: <s:property value="name"/> <br/> 
    <!-- 从ActionContext中取username的值 -->
    username : <s:property value="#username"/> <br/>
    <!-- 从Action对象中取username属性 -->
    username : <s:property value="username"/> <br/>
  </body>

```

**3、通过request/session等传输**
可通过` ServletActionContext.getRequest()/getSession() `等方法来获得request/session对象，然后调用其中的` setAttribute() `方法来传值。
（部传入ValueStack的request,session对象。）
演示各种数据的传输、展现技巧！
在Action中通过request/session来传值：
```

    public String detail(){
       //通过request
       ServletActionContext.getRequest().setAttribute("sex", "男"); 
       //通过session
       ServletActionContext.getRequest().getSession().setAttribute("address", "北京");
       //通过session
       ActionContext.getContext().getSession().put("postcode", "1234567");
       return "detail";
    }

```
在JSP中取值：
```

  <body>
    <!-- 从request中取sex值 -->
    request.sex = <s:property value="#request.sex"/> <br/>
    request.sex = <s:property value="#request['sex']"/> <br/>
   
    <!-- 从session中取值 -->
    session.address = <s:property value="#session.address"/> <br/>
    session.postcode = <s:property value="#session.postcode"/> <br/>
   
    <!-- 依次搜索page/request/session/application scope取值 -->
    attr.postcode=<s:property value="#attr.postcode"/> <br/>
  </body>

```
 
**4、传递复杂对象及集合对象**
如果在Action中传递一个复杂的对象到JSP，在JSP中，通过OGNL表达式，可以用句点“.”来访问对象中的属性。
如果传递一个集合对象到JSP，在JSP中可以通过<s:iterator>标签来访问集合中的数据。
Action中的代码：
 ```

      //传递复杂对象
       User u = new User();
       u.setUsername("admin");
       Group g = new Group();
       g.setName("管理员组");
       u.setGroup(g);
       ActionContext.getContext().put("user", u);
      
       //列表数据
       List list = new ArrayList();
       for(int i=0; i<10; i++){
           User user = new User();
           user.setUsername("User"+i);
           user.setAge(10+i);
           list.add(user);
       }
       ActionContext.getContext().put("users", list);

```
 
JSP中的代码：
```

    <!-- 通过句点访问对象的属性值 -->
    user.username=<s:property value="#user.username"/> <br/>
    user.group.name=<s:property value="#user.group.name"/> <br/>
   
    users: <br/>
    <s:iterator value="#users">
       <!-- 这个访问的是当前循环的user对象中的username属性 -->
       username:<s:property value="username"/>,<s:property value="age"/> <br/>
    </s:iterator>
   
    <!-- 这个访问的是Action对象中的username属性 -->
    username:<s:property value="username"/> <br/>

```
 
**5、利用OGNL表达式访问静态方法、普通的实例方法及Action对象中的方法**
假设有一个工具类，如下所示：
```

public class Utils {
    public static String toUpperCase(String str){
       return str.toUpperCase();
    }
   
    public String toLowerCase(String str){
       return str.toLowerCase();
    }
}

```
Action类的定义如下：
```

public class UserAction {
    private String username;
    //查看用户的详细信息
    public String detail(){
       ……………………
       return "detail";
    }
   
    //这个方法可以在JSP中用OGNL表达式直接调用!
    public Utils getUtils(){
       return new Utils();
    }

```
则在JSP中可以直接通过OGNL表达式来访问这些方法：
```

<!-- 调用静态方法 -->
<s:property value="@cn.com.leadfar.utils.Utils@toUpperCase(username)"/>
<!-- 利用OGNL表达式创建Utils对象，并调用它的实例方法 -->
<s:property value="new cn.com.leadfar.utils.Utils().toLowerCase(username)"/>
<!-- 调用Action对象的getUtils().toLowerCase()方法 -->
<s:property value="utils.toLowerCase(username)"/>

```
**【注意，在最新的struts2版本中，要想在JSP中通过OGNL表达式访问静态方法，则必须配置如下constant：
<constant name="struts.ognl.allowStaticMethodAccess" value="true"></constant>
】**
如何在iterator循环体内访问外部的同名属性？
在JSP中通过<s:iterator>访问列表数据：
```

    <s:iterator value="#users">
       <!-- 这个访问的是当前循环的user对象中的username属性 -->
<s:property value="username"/>
       <s:property value="#root[1].username"/> <br/>
    </s:iterator>

```
上面这个例子中，<s:property value=”username”>访问的是当前循环中的user对象的username属性，而<s:property value=”#root[1].username”/>访问的是UserAction对象中的username属性！