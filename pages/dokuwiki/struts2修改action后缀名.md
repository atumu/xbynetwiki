title: struts2修改action后缀名 

#  struts2修改action后缀名 
struts2 修改action的后缀

struts2 的默认后缀是 .action 虽然很直观，但是很烦琐。很多人喜欢将请求的后缀改为 .do

在struts2中修改action后缀有两种比较简单的办法：

一、在 struts.properties 中修改。

如你想把后缀改为 .do 则 加上一行： struts.action.extension=do

至于加在第几行，应该没有关系，我加在第一行和最后一样都正常。

二、在struts.xml 中修改。

在 struts.xml 中加入一constant 节点 ：

<constant name="struts.action.extension" value="do" />

在struts2中，所有的action类都有一个默认的后缀xx.action。例如： 
```

<struts>  
  <package name="default" namespace="/" extends="struts-default">  
    <action name="SayStruts2">  
        <result>pages/printStruts2.jsp</result>  
    </action>  
  </package>  
</struts> 

``` 

如果要访问 "SayStrute2" action类，使用如下的URL: 
Java代码  收藏代码
Action URL : http://localhost:8080/Struts2Example/SayStruts2.action  
struts2t是允许配置默认后缀的 
1 html后缀 
```

<struts>   
    <constant name="struts.action.extension" value="html"/>    
    <package name="default" namespace="/" extends="struts-default">  
    <action name="SayStruts2">  
    <result> pages/printStruts2.jsp</result>  
</action>  
    </package>   
</struts> 

``` 


此时访问"SayStruts2"action类可以通过： 
Java代码  收藏代码
Action URL : http://localhost:8080/Struts2Example/SayStruts2.html  


2  无后缀 

```

<struts>   
   <constant name="struts.action.extension" value=""/>    
   <package name="default" namespace="/" extends="struts-default">  
        <action name="SayStruts2">  
           <result> pages/printStruts2.jsp</result>  
        </action>  
   </package>  
</struts> 

``` 

此时访问"SayStruts2"action类可以通过： 
Java代码  收藏代码
Action URL : http://localhost:8080/Struts2Example/SayStruts2  