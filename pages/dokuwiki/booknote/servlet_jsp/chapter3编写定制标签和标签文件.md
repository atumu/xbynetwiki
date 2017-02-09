title: chapter3编写定制标签和标签文件 

#  Chapter3编写定制标签和标签文件 
当我们在 JSP 页面使用一个简单的标签时，**底层实际上由标签处理类提供支持，从而可以使用简单的标签来封装复杂的功能**，从而使团队更好地协作开发（能让美工人员更好地参与 JSP 页面的开发）。
<note tip>标签库是非常重要的技术，通常来说，初学者、普通开发人员自己开发标签库的机会很少，但如果希望成为高级程序员，或者希望开发通用框架，就需要大量开发自定义标签了。所有的 MVC 框架，如 Struts 2、SpringMVC、JSF 等都提供了丰富的自定义标签。</note>
##  编写定制标签 
**javax.servlet.jsp.tagext**包
###  一、基本概念 
1、标签(Tag)
标签是一种XML元素，通过标签可以使JSP网页变得简洁并且易于维护，还可以方便地实现同一个JSP文件支持多种语言版本。由于标签是XML元素，所以它的名称和属性都是大小写敏感的。
2、标签库(Tag library)
由一系列功能相似、逻辑上互相联系的标签构成的集合称为标签库。
3、标签库描述文件(Tag Library Descriptor)
标签库描述文件是一个XML文件，这个文件提供了标签库中类和JSP中对标签引用的映射关系。它是一个配置文件，和web.xml是类似的。
4、标签处理类(Tag Handle Class)
标签处理类是一个Java类，这个类继承了TagSupport或者扩展了SimpleTag接口，通过这个类可以实现自定义JSP标签的具体功能。
首先我们需要大致了解开发自定义标签所涉及到的接口与类的层次结构(其中SimpleTag接口与**SimpleTagSupport**类是JSP2.0中新引入的)。
![](/data/dokuwiki/booknote/servlet_jsp/pasted/20150603-100410.png)
###  创建和使用一个Tag Library的基本步骤 
早期 JSP 自定义标签类开发过程略微复杂一些，**但JSP 2 已经简化了这个过程，它只要自定义标签类都必须继承一个父类**：**javax.servlet.jsp.tagext.SimpleTagSupport**，除此之外，JSP 自定义标签类还有如下要求。
  * 如果标签类包含属性，每个属性都有对应的 **getter 和 setter** 方法。` 注意标签类必须要有一个无参构造器 `
  * 重写 **doTag()** 方法，这个方法**负责生成页面内容**。
JSP 2 规范简化了标签库的开发，在 JSP 2 中开发标签库只需如下几个**步骤**：
  * 开发自定义标签处理类；
  * 建立一个 *.tld 文件，每个 *.tld 文件对应一个标签库，每个标签库对应多个标签；
  * 在 JSP 文件中使用自定义标签。


###  创建标签库描述文件(Tag Library Descriptor) 
1、标签库描述文件，简称TLD，采用XML文件格式，定义了用户的标签库。TLD文件中的元素可以分成3类：
  * A.标签库元素 
  * B.标签元素 
  * C.标签属性元素
2、taglib元素用来设定标签库的相关信息，它的常用属性有：
  * tlib-version：指定该标签库实现的版本，这是一个作为标识的内部版本号，对程序没有太大的作用。
  * A.shortname：指定Tag Library默认的前缀名(prefix)；
  * B.uri：设定Tag Library的惟一访问表示符。
3、tag元素用来定义一个标签，它的常见属性有：
  * A.name：设定Tag的名字；
  * B.tagclass：设定Tag的处理类；
  * C.bodycontent：设定标签的主体(body)内容。
   *  * 1)**empty**：表示标签中没有body； 
  *   * 2)JSP：表示标签的body中可以加入JSP程序代码； 实际上由于 JSP 2 规范不再推荐使用 JSP 脚本，所以 JSP 2 自定义标签的标签体中不能包含 JSP 脚本。**所以实际上 body-content 元素的值不可以是 JSP。**
    * (3)**scriptless**：指定该标签的标签体可以是静态 HTML 元素，表达式语言，但不允许出现 JSP 脚本。
  *   * 4)tagdependent：表示标签中的内容由标签自己去处理。

4、标签属性元素用来定义标签的属性，它的常见属性有：
  * A.name：属性名称；
  * B.required：属性是否必需的，默认为false； 
  * C.fragment：设置该属性是否支持 JSP 脚本、表达式等动态内容，子元素的值是 true 或 false。
定义了上面的标签库定义文件后，将标签库文件放在 Web 应用的** WEB-INF 路径**，或任意子路径下，Java Web 规范会自动加载该文件，则该文件定义的标签库也将生效。
###  在Web应用中使用标签 
在 JSP 页面中确定指定标签需要 2 点：
  * 标签库 URI：确定使用哪个标签库。
  * 标签名：确定使用哪个标签。
` <%@ taglib uri="tagliburi" prefix="tagPrefix" %> `
如:` <%@ taglib uri="/WEB-INF/my.tld" prefix="my" %> `
` <%@ taglib uri="http://wiki.xby1993.net/my" prefix="my" %> `
###  带属性的标签 
实际上还有如下两种常用的标签：
  * 带属性的标签。
  * 带标签体的标签。

正如前面介绍的，**带属性标签必须为每个属性提供对应的 setter 和 getter 方法**。带属性标签的配置方法与简单标签也略有差别，下面介绍一个带属性标签的示例：
```

<!-- 定义第二个标签 -->
<tag>
    <!-- 定义标签名 -->
    <name>query</name>
    <!-- 定义标签处理类 -->
    <tag-class>lee.QueryTag</tag-class>
    <!-- 定义标签体为空 -->
    <body-content>empty</body-content>
    <!-- 配置标签属性:driver -->
    <attribute>
        <name>driver</name> 
        <required>true</required>
        <fragment>true</fragment>
    </attribute>
</tag>

```
###  带标签体的标签 

带标签体的标签，可以在标签内嵌入其他内容（包括静态的 HTML 内容和动态的 JSP 内容），通常用于完成一些逻辑运算，例如判断和循环等。下面以一个迭代器标签为示例，介绍带标签体标签的开发过程。
```

//标签的处理方法，简单标签处理类只需要重写doTag方法
    public void doTag() throws JspException, IOException
    {
        //从page scope中获取属性名为collection的集合
        Collection itemList = (Collection)getJspContext().
            getAttribute(collection);
        //遍历集合
        for (Object s : itemList)
        {
            //将集合的元素设置到page 范围
            getJspContext().setAttribute(item, s );
            //输出标签体
            getJspBody().invoke(null);
        }
    }

```
每次遍历都调用了** getJspBody()** 方法，如程序中粗体字代码所示，该方法返回该标签所包含的标签体**：JspFragment** 对象，执行该对象的** invoke() 方法，即可输出标签体内容**。
因为该标签的标签体不为空，配置该标签时**指定 body-content 为 scriptless**，该标签的配置代码片段如下代码所示：
```

<!-- 定义第三个标签 -->
<tag>
    <!-- 定义标签名 -->
    <name>iterator</name>
    <!-- 定义标签处理类 -->
    <tag-class>lee.IteratorTag</tag-class>
    <!-- 定义标签体支持JSP脚本 -->
    <body-content>scriptless</body-content>
 
  </tag>

```
```

 <!-- 使用迭代器标签，对a集合进行迭代 -->
        <mytag:iterator collection="a" item="item">
            <tr>
                <td>${pageScope.item}</td>
            <tr>
        </mytag:iterator>

```

###  自定义EL函数 
一、利用EL表达式调用普通Java类中的静态方法
1、编写一个java类，并编写一个静态方法，如下所示：
```

public class ElDemo {
//静态方法:将小写转换为大写
public static String convert(String str){
return str.toUpperCase();
}
}

```
在tld文件中添加
```

<function><!--定义一个函数-->
	<name>convert</name><!--函数名称，以便在jsp中调用-->
	<function-class>cn.itcast.el.ElDemo</function-class><!--指定是哪个类中的函数，类名用全名-->
	<function-signature>
		java.lang.String convert(java.lang.String)<!--指定方法的签名。返回值类型和参数类型要用全名-->
	</function-signature>
</function>

```
在web.xml中注册该tld文件
```

<jsp-config>
  <taglib>
  <taglib-uri>http://www.itcast.cn/tld</taglib-uri><!--与tld中uri一致就可-->
  <taglib-location>/WEB-INF/tld/mytld.tld</taglib-location><!--tld所在的位置-->
  </taglib>
  </jsp-config>

```
4、在JSP中使用该函数
先利用taglib指令引入该tld
<%@ taglib uri="http://www.itcast.cn/tld" prefix="mm"%>
用一下代码来调用
${mm:convert("abcdefg")}输出ABCDEFG
###  HelloWorld标签实例 
下面开发一个最简单的自定义标签，该标签负责在页面上输出 HelloWorld。
```

// 标签处理类，继承 SimpleTagSupport 父类
public class HelloWorldTag extends SimpleTagSupport 
{ 
    // 重写 doTag 方法，该方法在标签结束生成页面内容
    public void doTag()throws JspException, 
        IOException 
    { 
        // 获取页面输出流，并输出字符串
        getJspContext().getOut().write("Hello World"); 
    } 
}
上面这个标签处理类非常简单，它继承了 SimpleTagSupport 父类，并重写 doTag() 方法，而 doTag() 方法则负责输出页面内容。该标签没有属性，因此无须提供 setter 和 getter 方法。
###  建立 TLD 文件 
标签库定义文件的根元素是 taglib，它可以包含多个 tag 子元素，每个 tag 子元素都定义一个标签。通常我们可以到 Web 容器下复制一个标签库定义文件，并在此基础上进行修改即可。例如 Tomcat6.0，在 webapps\examples\WEB-INF\jsp2 路径下包含了一个 jsp2-example-taglib.tld 文件，这就是示范用的标签库定义文件。
将该文件复制到 Web 应用的 WEB-INF/ 路径，或 WEB-INF 的任意子路径下，并对该文件进行简单修改，修改后的 mytaglib.tld 文件代码如下：
<code xml>
<?xml version="1.0" encoding="GBK"?>
<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee web-jsptaglibrary_2_0.xsd"
    version="2.0">
    <tlib-version>1.0</tlib-version>
    <short-name>mytaglib</short-name>
    <!-- 定义该标签库的URI -->
    <uri>http://www.crazyit.org/mytaglib</uri>
    <!-- 定义第一个标签 -->
    <tag>
        <!-- 定义标签名 -->
        <name>helloWorld</name>
        <!-- 定义标签处理类 -->
        <tag-class>lee.HelloWorldTag</tag-class>
        <!-- 定义标签体为空 -->
        <body-content>empty</body-content>
    </tag>
</taglib>

```
实际上 JSTL 标签库提供了一套功能非常强大标签，例如普通的输出标签，像我们刚刚介绍的迭代器标签，还有用于分支判断的标签等等，**JSTL（JSP 标准标签库）**都有非常完善的实现。除此之外，**Apache 下还有一套 DisplayTags 的标签库实现，做得也非常不错。**


##  编写标签文件 
###  1.  标签文件介绍 
标签文件是JSP2.0新增的功能，标签文件的本质在转译成一个Servlet之后，是一个实现了SimpleTag接口的类，目标是让开发人员可以直接使用JSP语法来制作标签。
标签文件是以**.tag或者.tagx**作为扩展名，如果标签文件中包含了的其他完整或部分片段的标签文件，那么应该以**.tagf**为扩展名的。标签文件必须存放在WEB-INF目录下，最好是在**WEB-INF**目录下再新建一个**tags目录**，然后将所有的标签文件都存放在这里。**标签文件可以使用所有的JSP元素，但是不能使用page指令**，因为标签文件不一个页面，标签文件多个一个tag指令
###  2.  标签文件的隐藏属性 

标签文件可以使用隐藏属性：**request、response、jspContext、session、application、out、config**，其中jspContext隐含对象的类型为javax.servlet.jsp.JspContext，它用来管理所有范围的变量，就相当于JSP页面的pageContext隐藏对象一样
###  3.  标签文件的指令 

标签文件中可以使用的主要指令：**taglib、include、attribute、variable**。

` <%@ tag display-name="" body-content="" dynamic-attributes="" small-icon="" large-icon="" description="" example="" language="" import="" pageEncoding="" isELIgnored=""> ` 
**body-content表示可能的值有三种，分别是empty、scriptless、tagdependent、empty**。默认值为scriptless；
当dynamic-attributes设定时,将会产生一个Map类型的集合对象,用来存放属性的名称和值；

` <%@ attribute name="" required="" fragment="" rtexprvalue="" type="" description=""%> `
` <%@ variable name-given="" name-from-attribute="" alias="" variable-class="" declare="" scope="" desription=""> `
scope表示此变量的范围，范围是：AT_BEGIN、AT_END和NESTED，默认值为NESTED；
` <%@ include file="include.tagf" %> ` include.tagf不是一个完整的tag文件
` <%@ include file="include.html" %> `
###  4.  标签文件的使用 

在引用标签文件的JSP页面必须使用taglib指令**<%@ taglib tagdir="/WEB-INF/tags" prefix="r" %>**，其中tagdir属性指定标签文件的地址。使用的自定义标签文件应该遵循<prefix:TagFileName/>模式。如果标签具有主体，则应该跟随具有</prefix:TagFileName>格式的结束标签。也就是说如果在WEB-INF/tags目录下有一个row.tag的标签文件，那么就可以这样来指定<%@ tagdir="/WEB-INF/tags" prefix="r" %>, 调用为<r:row/>
###  5.  标签文件的示例 
设置了属性fragment=true,就需要通过jsp:invoke来调用属性主体
''<%@ attribute name="even" fragment="true" required="true"%>
<jsp:invoke fragment="even" />
<!-- 调用标签主体-->
<jsp:dobody/>''
**注意jsp:invoke与jsp:dobody的异同**
```

/*# = 标签文件row.tag==*/
<%@ attribute name="items" rtexprvalue="true" required="true"%>
<%@ attribute name="even" fragment="true" required="true"%>
<%@ attribute name="odd" fragment="true" required="true"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

<c:forEach items="${items}" varStatus="status">
 <c:choose>
  <c:when test="${status.count % 2 == 0}">
    <!-- 设置了属性fragment=true,就需要通过jsp:invoke来调用属性主体-->
   <jsp:invoke fragment="even" />
  </c:when>
  <c:otherwise>
   <jsp:invoke fragment="odd" />
  </c:otherwise>
 </c:choose>
</c:forEach>
<!-- 调用标签主体-->
<jsp:dobody/>
  
  
  <!--JSP片段-->
  <table>
 <h:row items="1,2,3,4,5,6,7,8,9,10">
  <jsp:attribute name="even">
   <c:set var="counter" value="${counter + 1}" />
   <tr bgcolor="pink">
    <td>${counter}: 偶数行</td>
   </tr> 
  </jsp:attribute>
  <jsp:attribute name="odd">
   <c:set var="counter" value="${counter + 1}" />
   <tr bgcolor="orange">
    <td>${counter}: 奇数行</td>
   </tr> 
  </jsp:attribute>
   haha,我是标签主体，通过dobody调用
 </h:row>
</table>

```
  
参考：
http://www.ibm.com/developerworks/cn/java/j-lo-jsp2tag/
http://www.cnblogs.com/zhaoyang/archive/2011/12/25/2301108.html
http://www.51cto.com/specbook/11/57761.htm
http://blog.csdn.net/afgasdg/article/details/6990527
http://blog.csdn.net/csuliky/article/details/2481667