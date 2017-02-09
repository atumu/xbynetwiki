title: springmvc页面跳转 

#  SpringMVC之页面跳转 
SpringMVC页面跳转3种方式:
1、return "forward:/home"  浏览器端URL不会有变化。
2、return "redirect:/home" 浏览器端URL会变化。**同时/home前会加上域名+contextName**.如http://localhost/myapp/home
3、resp.setStatus(302);  浏览器端URL会变化。 采用设置header Location属性的方式。必须显示设置302状态码，否则不跳转。
resp.setHeader("Location", "http://localhost/home"); setHeader必须在resp.flush之前调用，否则失效。

主要谈谈2和3的区别：
遇到的场景：
web服务器采用nginx,应用服务器采用tomcat.多个子域名对应一台主机的一个IP。每个子域名通过nginx反向代理访问tomcat下的对应应用。
如tomcat下有两个应用：app, test.
tomcat访问：www.abc.com:8080/app , www.abc.com:8080/test
nginx反向代理访问：app.abc.com, test.abc.com
注意这两种形式的URL区别。如果采用return "redirect:/home"方式。那么通过tomcat访问没问题。www.abc.com:8080/app/home. 但是通过nginx反向代理访问的时候变成了。app.abc.com/app/home.
而app.abc.com对应的是www.abc.com:8080/app 那么这样一来。实际URI变成了。www.abc.com:8080/app/app/home .显示不是我们希望的。
这个时候就需要通过设置header的location属性来解决了。所以才有了我们的第3种方式。
```

resp.setStatus(302);
resp.setHeader("Location", "http://app.abc.com/home");

```
此时通过浏览器访问的时候是http://app.abc.com/home符合我们希望。
问题解决。