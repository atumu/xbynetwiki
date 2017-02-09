title: spring-security处理session超时 

#  Spring-Security处理Session超时 
```

	<http>  
		....

		<!-- invalid-session-url指定处理Session超时的URL  -->
		<!--会话固定攻击防护，有几种选择：changeSessionId(只支持Servlet3.1以上),migrateSession(默认选项，将创建新的会话并复制所有现有特性),newSession,none -->
		<session-management session-fixation-protection="migrateSession" invalid-session-url="${host}/sessiontimeout"> 
			<concurrency-control max-sessions="1" expired-url="/login?maxSessions=1" /> <!--限制用户会话数量，为了启用该特性，记得配置前面所讲的特殊的监听器 -->
		</session-management>

```
```

@RequestMapping("/sessiontimeout")
	public void sessiontimeout(HttpServletRequest request,HttpServletResponse response){
		request.getSession().setAttribute("loginId", UUID.randomUUID().toString());
		String requestType = request.getHeader("X-Requested-With");
		// 如果是ajax请求
		if (StringUtils.checkEquals(requestType, "XMLHttpRequest")) {
			// 给前台传一个超时标志
			response.setHeader("sessionstatus", "timeout");
			log.debug("ajax请求超时处理");
			// 返回任意一个json串，防止前台报错
			try {
				response.getOutputStream().print("{\"status\":\"timeout\"}");
				
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
				log.debug("ajax超时处理异常{}",e);
			}
		} else {
			// 如果是普通的浏览器HTTP请求。
			ControllerUtil.redirectURL(response, "/login");
			log.debug("HTTP session超时处理异常");
		}
	}

```
参考：http://zhengjunxiang.iteye.com/blog/1990689