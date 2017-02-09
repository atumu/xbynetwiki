title: servlet3.0_3.1的web.xml 

#  servlet3.0_3.1的web.xml的DTD 
参考：http://stackoverflow.com/questions/19661135/dynamic-web-module-3-0-3-1
  * Servlet3.0属性JavaEE6规范。tomcat7支持
  * Servlet3.1属性JavaEE7规范。tomcat8支持
Servlet3.x之后的web.xml schema与之前是不同的。记录如下
Servlet3.0：
```

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>

```
Servlet3.1:
```

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_1.xsd"
    version="3.1">

```
```

<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee" 
         xmlns:web="http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1" xmlns="http://xmlns.jcp.org/xml/ns/javaee">

</web-app>

```