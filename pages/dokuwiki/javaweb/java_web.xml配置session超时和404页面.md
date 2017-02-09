title: java_web.xml配置session超时和404页面 

#  java web.xml配置session超时和404页面 
```

<!-- session timeout 30分钟-->
<session-config>
	<session-timeout>30</session-timeout>
</session-config>

<!-- error page -->
<error-page>
	<error-code>404</error-code>
	<location>/WEB-INF/errors/404.jsp</location>
</error-page>

```