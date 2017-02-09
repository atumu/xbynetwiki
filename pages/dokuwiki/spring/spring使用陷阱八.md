title: spring使用陷阱八 

#  Spring陷阱之拦截器反射获取被拦截方法信息 
```

public class GlobalInterceptor extends HandlerInterceptorAdapter{
	private static Logger log=LoggerFactory.getLogger(LoginInterceptor.class);
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		HttpSession s=request.getSession();
		s.setAttribute("host", PathUtil.getHost());
		s.setAttribute("siteName", GeneUtil.SITE_NAME);
		if(handler instanceof HandlerMethod){
			HandlerMethod hm=(HandlerMethod)handler;
			Object target=hm.getBean();
			Class<?> clazz=hm.getBeanType();
			Method m=hm.getMethod();
		}
		return true;
	}


}


```