title: el表达式中reuqestscope与pagecontext.request的区别 

#  el表达式中reuqestScope与pageContext.request的区别 
以前没注意它们的区别，现在记录下来：
reuqestScope对应的对象不是HttpServletRequest,故而无法通过它获取项目名称等信息。
pageContext.request对应的对象是HttpServletRequest，可以通过它获取项目名称等信息。
看以下实列：
```

<c:out value="${requestScope.contextPath}"/>没有输出。
<c:out value="${pageContext.request.contextPath}"/>输出项目上下文路径，如：/hibernate-test

```
