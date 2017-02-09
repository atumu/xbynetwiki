createAt:2017-01-06 10:17:05
author:xbynet
modifyAt:2017-01-06 10:17:15
location:javaee/weblogic下ContextPath获取问题
title:weblogic下ContextPath获取问题

方式一、在weblogic.xml中加入
```
<container-descriptor>
         <show-archived-real-path-enabled>true</show-archived-real-path-enabled>
</container-descriptor>
```

方式二、
```
String webRoot = request.getSession().getServletContext().getRealPath("/");
if(webRoot == null){
    webRoot = this.getClass().getClassLoader().getResource("/").getPath();
    webRoot = webRoot.substring(0,webRoot.indexOf("WEB-INF"));
}
```
参考：
http://stackoverflow.com/questions/536228/why-does-getrealpath-return-null-when-deployed-with-a-war-file
http://ju.outofmemory.cn/entry/255336
https://mritd.me/2016/04/22/WebLogic-request-getContextPath-%E4%B8%BA%E7%A9%BA%E9%97%AE%E9%A2%98/