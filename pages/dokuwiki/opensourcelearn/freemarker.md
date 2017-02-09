title: freemarker 

#   freemarker学习 


什么是Freemarker:.
Freemaker是一个”模板引擎”,也可以说是一个基于模板技术的生成文本输出的一个通用工具.它是一个JAVA的包,一个JAVA程序员可以使用的类库.本身并不是一个对最终用户的应用程序.但是,程序员可以把它应用到他们的产品中.
FreeMarker是设计为可以生成WEB PAGES.它是基于SERVLET遵循MVC模式的.这个思路是应用MVC模式要降低分离,网页设计人员和程序员的耦合.每个人都可以做他们擅长的工作.网页设计人员可以改变网页的面貌,而并不需要程序员的重新编译.因为业务逻辑和页面的设计已经被分离开了.模板是不能由复杂的程序片断组成的.即便网页设计人员和程序员是一个人. 分离是有必要的.它能使程序更加的灵活和清晰.
虽然Freemarker能编程,但是它并不是一个编程语言.它是为程序显示数据而准备的.(像数据库SQL语句的查询.)以及.Freemarker仅仅是利用模板加上数据生成文本页面.
Freemarker并不是一个WEB应用程序框架.可以说是一个WEB应用框架的一个组件.但是FREEMARKER引擎本身并不了解HTTP或者SERVLETS.它只不过生成文本而已.注意,它是MVC框架的一个组件(如STRUTS),也可以在模板中使用JSP标签.
  Freemarker下载地址为：http://www.freemarker.org/index.html
一般的用途:
l 能用来生成任意格式的文本:HTML,XML,RTF,JAVA源码,等等.
l可以更好的嵌入到你的产品中,轻量级的.并不需要servlet环境.不依赖javax.servlet.classes.
l可插入的模板读取器:你可以从任意的源码读取任意的模板.本地的文件,数据库等等.
l你可以做任意你想生成的文本.存储为本地文件.可以用来发送EMAIL或返回到WEB浏览器中.
强大的模板语言
l完整的指令:include,if/elseif/else,loop.
l 建立和修改模板中的变量.
l 能用复杂的表达式在任意地方指定变量.
n字符串操作:concateration,sub-string,uppercase,capitalize,escaping.等等
n十进制数学计算.
n BOOL
n读取数组和相关的数组元素.
n可以自己添加特殊的计算方法.
l宏指令
l 命名空间用来创建和维护宏指令库或者把大的项目分成许多模块.并不用担心命名冲突
 
Freemarker总结二
 
#  二．环境搭建与配置 

<!—freemarker初始配置-->
 
<servlet>
<servlet-name>freemarker</servlet-name>
<servlet-class>freemarker.ext.servlet.FreemarkerServlet</servlet-class>
<init-param>
<param-name>TemplatePath</param-name>
<param-value>/</param-value>
</init-param>
<init-param>
<param-name>NoCache</param-name>
<param-value>true</param-value>
</init-param>
<init-param>
<param-name>ContentType</param-name>
<param-value>text/html</param-value>
</init-param>
<init-param>
<param-name>template_update_delay</param-name>
<param-value>0</param-value>
</init-param>
<init-param>
<param-name>default_encoding</param-name>
<param-value>GBK</param-value>
</init-param>
<init-param>
<param-name>locale</param-name>
<param-value>zh_CN </param-value>
</init-param>
<init-param>
<param-name>number_format</param-name>
<param-value>0.##########</param-value>
</init-param>
<load-on-startup>2</load-on-startup>
</servlet>
<servlet-mapping>
<servlet-name>action</servlet-name>
<url-pattern>*.do</url-pattern>
</servlet-mapping>
<servlet-mapping>
<servlet-name>freemarker</servlet-name>
<url-pattern>*.ftl</url-pattern>
</servlet-mapping>
 
----------------------------------------------------------------------------
#  三．常用语法 

 EG.一个对象BOOK
  1.输出 ${book.name}
空值判断：${book.name?if_exists },
${book.name?default(‘xxx’)}//默认值xxx
${ book.name!"xxx"}//默认值xxx
日期格式：${book.date?string('yyyy-MM-dd')}
数字格式：${book?string.number}--20
${book?string.currency}--<#-- $20.00 -->
${book?string.percent}—<#-- 20% -->
插入布尔值：
<#assign foo=ture />
${foo?string("yes","no")} <#-- yes -->
 
2．逻辑判断
a:
<#if condition>...
<#elseif condition2>...
<#elseif condition3>......
<#else>...
其中空值判断可以写成<#if book.name?? >
 
</#if>
b:
<#switch value>
  <#case refValue1>
    ...
    <#break>
  <#case refValue2>
    ...
    <#break>
  ...
  <#case refValueN>
    ...
    <#break>
  <#default>
    ...
</#switch>
 
3．循环读取
<#list sequence as item>
...
</#list>
空值判断<#if bookList?size = 0></#list>
e.g.
<#list employees as e>
${e_index}. ${e.name}
</#list>
输出:
1. Readonly
2. Robbin
 
Freemarker总结三
#  4.FreeMarker 3 宏/模板 

宏Macro
宏是在模板中使用macro指令定义
l.1 基本用法
宏是和某个变量关联的模板片断，以便在模板中通过用户定义指令使用该变量，下面是一个例子：
<#macro greet>
  <font size="+2">Hello Joe!</font>
</#macro>
调用宏时，与使用FreeMarker的其他指令类似，只是使用@替代FTL标记中的#。
<@greet></@greet> <#--<@greet/>-->
在macro指令中可以在宏变量之后定义参数，如：
<#macro greet person>
  <font size="+2">Hello ${person}!</font>
</#macro>
可以这样使用这个宏变量：
<@greet person="Fred"/>
但是下面的代码具有不同的意思：
<@greet person=Fred/>
这意味着将Fred变量的值传给person参数，该值不仅是字符串，还可以是其它类型，甚至是复杂的表达式。
宏可以有多参数，下面是一个例子：
<#macro greet person color>
  <font size="+2" color="${color}">Hello ${person}!</font>
</#macro>
可以这样使用该宏变量，其中参数的次序是无关的：
<@greet person="Fred" color="black"/>
可以在定义参数时指定缺省值，否则，在调用宏的时候，必须对所有参数赋值：
<#macro greet person color="black">
  <font size="+2" color="${color}">Hello ${person}!</font>
</#macro>
注意：宏的参数是局部变量，只能在宏定义中有效。
嵌套内容
FreeMarker的宏可以有嵌套内容，<#nested>指令会执行宏调用指令开始和结束标记之间的模板片断，举一个简单的例子：
<#macro border>
  <table border=4 cellspacing=0 cellpadding=4><tr><td>
    <#nested>
  </tr></td></table>
</#macro>
执行宏调用：
<@border>The bordered text</@border>
输出结果：
<table border=4 cellspacing=0 cellpadding=4><tr><td>
    The bordered text
  </tr></td></table>
<#nested>指令可以被多次调用，每次都会执行相同的内容。
<#macro do_thrice>
  <#nested>
  <#nested>
  <#nested>
</#macro>
<@do_thrice>
  Anything.
</@do_thrice>
FMPP 输出结果：
Anything.
Anything.
Anything.
嵌套内容可以是有效的FTL，下面是一个有些复杂的例子，我们将上面三个宏组合起来：
<@border>
  <ul>
  <@do_thrice>
    <li><@greet person="Joe"/>
  </@do_thrice>
  </ul>
</@border>
输出结果：
<table border=4 cellspacing=0 cellpadding=4><tr><td>
  <ul>
    <li><font size="+2">Hello Joe!</font>
    <li><font size="+2">Hello Joe!</font>
    <li><font size="+2">Hello Joe!</font>
  </ul>
  </tr></td></table>
宏定义中的局部变量对嵌套内容是不可见的，例如：
<#macro repeat count>
  <#local y = "test">
  <#list 1..count as x>
    ${y} ${count}/${x}: <#nested>
  </#list>
</#macro>
<@repeat count=3>${y?default("?")} ${x?default("?")} ${count?default("?")}</@repeat>
输出结果：
test 3/1: ? ? ?
test 3/2: ? ? ?
test 3/3: ? ? ?
在宏定义中使用循环变量
nestted指令也可以有循环变量（循环变量的含义见下节），调用宏的时候在宏指令的参数后面依次列出循环变量的名字，格式如下：
<@ macro_name paramter list; loop variable list[,]>
例如：
<#macro repeat count>
  <#list 1..count as x>
    <#nested x, x/2, x==count>
  </#list>
</#macro>
<@repeat count=4 ; c, halfc, last>
  ${c}. ${halfc}<#if last> Last!</#if>
</@repeat>
这里count是宏的参数，c, halfc,last则为循环变量，输出结果：
1. 0.5
  2. 1
  3. 1.5
  4. 2 Last!
循环变量和宏标记指定的不同不会有问题，如果调用时少指定了循环变量，那么多余的值不可见。调用时多指定了循环变量，多余的循环变量不会被创建：
<@repeat count=4 ; c, halfc, last>
  ${c}. ${halfc}<#if last> Last!</#if>
</@repeat>
<@repeat count=4 ; c, halfc>
  ${c}. ${halfc}
</@repeat>
<@repeat count=4>
  Just repeat it...
</@repeat>
在模板中定义变量
在模板中定义的变量有三种类型：
plain变量：可以在模板的任何地方访问，包括使用include指令插入的模板，使用assign指令创建和替换。
局部变量：在宏定义体中有效，使用local指令创建和替换。
循环变量：只能存在于指令的嵌套内容，由指令（如list）自动创建；宏的参数是局部变量，而不是循环变量
局部变量隐藏（而不是覆盖）同名的plain变量；循环变量隐藏同名的局部变量和plain变量，下面是一个例子：
<#assign x = "plain">
 
${x}  <#-- we see the plain var. here -->
<@test/>
6. ${x}  <#-- the value of plain var. was not changed -->
<#list ["loop"] as x>
    7. ${x}  <#-- now the loop var. hides the plain var. -->
    <#assign x = "plain2"> <#-- replace the plain var, hiding does not mater here -->
    8. ${x}  <#-- it still hides the plain var. -->
</#list>
9. ${x}  <#-- the new value of plain var. -->
<#macro test>
  2. ${x}  <#-- we still see the plain var. here -->
  <#local x = "local">
  3. ${x}  <#-- now the local var. hides it -->
  <#list ["loop"] as x>
    4. ${x}  <#-- now the loop var. hides the local var. -->
  </#list>
  5. ${x}  <#-- now we see the local var. again -->
</#macro>
输出结果：
1. plain
  2. plain
  3. local
  4. loop
  5. local
  6. plain
  7. loop
  8. loop
  9. plain2
内部循环变量隐藏同名的外部循环变量，如：
<#list ["loop 1"] as x>
  ${x}
  <#list ["loop 2"] as x>
    ${x}
    <#list ["loop 3"] as x>
      ${x}
    </#list>
    ${x}
  </#list>
  ${x}
</#list>
输出结果：
loop 1
    loop 2
      loop 3
    loop 2
  loop 1
模板中的变量会隐藏（而不是覆盖）数据模型中同名变量，如果需要访问数据模型中的同名变量，使用特殊变量global，下面的例子假设数据模型中的user的值是Big Joe：
<#assign user = "Joe Hider">
${user}          <#-- prints: Joe Hider -->
${.globals.user} <#-- prints: Big Joe -->
名字空间
通常情况，只使用一个名字空间，称为主名字空间，但为了创建可重用的宏、变换器或其它变量的集合（通常称库），必须使用多名字空间，其目的是防止同名冲突
创建库
下面是一个创建库的例子（假设保存在lib/my_test.ftl中）：
<#macro copyright date>
  <p>Copyright (C) ${date} Julia Smith. All rights reserved.
  <br>Email: ${mail}</p>
</#macro> 
<#assign mail = "jsmith@acme.com">
使用import指令导入库到模板中，Freemarker会为导入的库创建新的名字空间，并可以通过import指令中指定的散列变量访问库中的变量：
<#import "/lib/my_test.ftl" as my>
<#assign mail="fred@acme.com">
<@my.copyright date="1999-2002"/>
${my.mail}
${mail}
输出结果：
<p>Copyright (C) 1999-2002 Julia Smith. All rights reserved.
  <br>Email: jsmith@acme.com</p>
jsmith@acme.com
fred@acme.com
可以看到例子中使用的两个同名变量并没有冲突，因为它们位于不同的名字空间。还可以使用assign指令在导入的名字空间中创建或替代变量，下面是一个例子：
<#import "/lib/my_test.ftl" as my>
${my.mail}
<#assign mail="jsmith@other.com" in my>
${my.mail}
输出结果：
jsmith@acme.com
jsmith@other.com
数据模型中的变量任何地方都可见，也包括不同的名字空间，下面是修改的库：
<#macro copyright date>
  <p>Copyright (C) ${date} ${user}. All rights reserved.</p>
</#macro>
<#assign mail = "${user}@acme.com">
假设数据模型中的user变量的值是Fred，则下面的代码：
<#import "/lib/my_test.ftl" as my>
<@my.copyright date="1999-2002"/>
${my.mail}
输出结果：
<p>Copyright (C) 1999-2002 Fred. All rights reserved.</p>Fred@acme.com

Freemarker总结四
< type="text/javascript">< type="text/javascript" src="http://pagead2.googlesyndication.com/pagead/show_ads.js">
#  四．Freemarker与Struts结合 

1.输出文件换成以ftl格式的文件
E.G.
<action name="bookActionForm" parameter="method" path="/bookAction" scope="request" type="example.BookAction" validate="true">
<forward name="list" path="/index.ftl"/>
</action>
2.使用struts，jstl等标签
a.导入à<#global html=JspTaglibs["/WEB-INF/tags/struts-html.tld"]>
或<#assign html=JspTaglibs["/WEB-INF/struts-html.tld"]>
<#assign bean=JspTaglibs["/WEB-INF/struts-bean.tld"]>
<#assign logic=JspTaglibs["/WEB-INF/struts-logic.tld"]>
b.使用à<@bean.page id="request" property="request"/>,
<@html.text property="vo.newsTitle" styleClass="input1"/>
#  五．用Freemarker生成Html页面 

   例子：MakeFileManager.java
package example;
import freemarker.template.Configuration;
import java.text.SimpleDateFormat;
import java.io.File;
import freemarker.template.DefaultObjectWrapper;
import java.util.Map;
import java.util.HashMap;
import java.io.Writer;
import java.io.OutputStreamWriter;
import java.io.FileOutputStream;
import freemarker.template.TemplateException;
import java.io.IOException;
import freemarker.template.Template;
public class MakeFileManager {
    public String make(Book book, BookFtl bookFtl) {
        Configuration cfg = new Configuration();//配制
        String realPath = bookFtl.getRealPath();
        String templatePath = realPath + "/WEB-INF/templates/book";
        String cDateStr = "book/" +
                          new SimpleDateFormat("yyyyMMdd").format(new java.util.
                Date());
        String filePostfix = ".html";
        String path = realPath + "/" + cDateStr;
        String fileTimeName = new SimpleDateFormat("yyyyMMddhhmmss").format(new
                java.util.Date());
        String returnFileName = cDateStr + "/" + fileTimeName + filePostfix;
        String fileName = "";
        File bookDir = new File(path);
        if (bookDir.exists()) {
            fileName = path + "/" + fileTimeName + filePostfix;
        } else {
            bookDir.mkdirs();
            fileName = path + "/" + fileTimeName + filePostfix;
        }
        try {
            //设置freemarker的参数
            cfg.setNumberFormat("0.##########");//生成html文件时web.xml配制无效
            //cfg.setEncoding();
            cfg.setDirectoryForTemplateLoading(new File(templatePath));
            cfg.setObjectWrapper(new DefaultObjectWrapper());
            Template bookTemplate = cfg.getTemplate(bookFtl.getTemplateName());//模板对象
            Map root = new HashMap();
            root.put("book", book);
            root.put("book2",book);
            Writer out = new OutputStreamWriter(new FileOutputStream(fileName));
            try {
                bookTemplate.process(root, out);
            } catch (TemplateException e) {
                e.printStackTrace();
            }
            out.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return returnFileName;
    }
}
< type="text/javascript">< type="text/javascript" src="http://pagead2.googlesyndication.com/pagead/show_ads.js">
原文：http://blog.csdn.net/sz_bdqn/article/details/5308939

#  freemarker函数总结 
一、Sequence的内置函数


1. sequence?first 返回sequence的第一个值。

2. sequence?last 返回sequence的最后一个值。

3. sequence?reverse 将sequence的现有顺序反转，即倒序排序

4. sequence?size 返回sequence的大小

5. sequence?sort 将sequence中的对象转化为字符串后顺序排序

6. sequence?sort_by(value) 按sequence中对象的属性value进行排序


二、Hash的内置函数


1. hash?keys 返回hash里的所有key,返回结果为sequence

2. hash?values 返回hash里的所有value,返回结果为sequence


例如：

<#assign user={"name":"hailang", "sex":"man"}>

<#assign keys=user?keys>

<#list keys as key>

${key}=${user[key]}

</#list>


三、 操作字符串函数


1. substring（start,end）从一个字符串中截取子串

start:截取子串开始的索引，start必须大于等于0，小于等于end

end: 截取子串的长度，end必须大于等于0，小于等于字符串长度，如果省略该参数，默认为字符串长度。


例子：

${"str"?substring(0)}结果为str


${"str"?substring(1)}结果为tr


${"str"?substring(2)}结果为r


${"str"?substring(3)}结果为


${"str"?substring(0,0)}结果为


${"str"?substring(0,1)}结果为s


${"str"?substring(0,2)}结果为st


${"str"?substring(0,3)}结果为str


2. cap_first 将字符串中的第一个单词的首字母变为大写。


${"str"?cap_first}结果为Str


3. uncap_first将字符串中的第一个单词的首字母变为小写。


${"Str"?cap_first}结果为str


4. capitalize将字符串中的所有单词的首字母变为大写


${"str"?capitalize}结果为STR


5. date,time，datetime将字符串转换为日期


例如：


<#assign date1="2009-10-12"?date("yyyy-MM-dd")>


<#assign date2="9:28:20"?time("HH:mm:ss")>


<#assign date3="2009-10-12 9:28:20"?time("HH:mm:ss")>


${date1}结果为2009-10-12


${date2}结果为9:28:20


${date3}结果为2009-10-12 9:28:20


注意：如果指定的字符串格式不正确将引发错误。


6. ends_with 判断某个字符串是否由某个子串结尾，返回布尔值。


${"string"?ends_with("ing")?string} 返回结果为true


注意：布尔值必须转换为字符串才能输出


7. html 用于将字符串中的<、>、&和"替换为对应得


8. index_of（substring,start）在字符串中查找某个子串，返回找到子串的第一个字符的索引，如果没有找到子串，则返回-1。


Start参数用于指定从字符串的那个索引处开始搜索，start为数字值。


如果start大于字符串长度，则start取值等于字符串长度，如果start小于0， 则start取值为0。


${"string"?index_of("in")结果为3


${"string"?index_of("ab")结果为-1


9. length返回字符串的长度 ${"string"?length}结果为6


10. lower_case将字符串转为小写


${"STRING"?lower_case}结果为string


11. upper_case将字符串转为大写


${"string"?upper_case}结果为STRING


12. contains 判断字符中是否包含某个子串。返回布尔值


${"string"?contains("ing")?string}结果为true


注意：布尔值必须转换为字符串才能输出


13. number将字符串转换为数字


${"111.11"?number}结果为111.11


14. replace用于将字符串中的一部分从左到右替换为另外的字符串。


${"strabg"?replace("ab","in")}结果为string


15. split使用指定的分隔符将一个字符串拆分为一组字符串


<#list "This|is|split"?split("|") as s>


${s}


</#list>


结果为:


This


is


split


16. trim 删除字符串首尾空格 ${"String"?trim} 结果为String


四、 操作数字


1. c 用于将数字转换为字符串


${123?c} 结果为123


2. string用于将数字转换为字符串


Freemarker中预订义了三种数字格式：number,currency（货币）和percent(百分比)其中number为默认的数字格式转换


例如： 


<#assign tempNum=20>


${tempNum} 


${tempNum?string.number}或${tempNum?string("number")} 结果为20


${tempNum?string.currency}或${tempNum?string("currency")} 结果为￥20.00


${tempNum?string. percent}或${tempNum?string("percent")} 结果为2,000%


五、 操作布尔值


string 用于将布尔值转换为字符串输出

true转为"true"，false转换为"false"

原文：http://my.oschina.net/u/1466836/blog/210700
