title: spring-security项目配置实战1 

#  Spring-Security项目配置实战一 
##  web.xml 

```

<?xml version="1.0" encoding="utf-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
>

  <display-name>Archetype Created Web Application</display-name>
  <error-page>
  	<error-code>404</error-code>
  	<location>/404.jsp</location>
  </error-page>
  <error-page>  
    <error-code>500</error-code>  
    <location>/500.jsp</location>  
</error-page>
  <error-page>  
    <error-code>401</error-code>  
    <location>/401.jsp</location>  
</error-page>  

	<!-- 通过配置ContextLoaderListener与全局的contextConfigLocation参数用于启用一个全局的Spring应用上下文，
	作为所有的DispatcherServlet应用上下文的父亲，可以共享这个父亲的所有bean与配置 -->
  	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
         classpath:spring-hibernate.xml, classpath:spring-security.xml
        </param-value>
      </context-param>

  	<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
 <listener>
     <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class> <!--为了启用登录并发控制，即同一个用户的登录人数限制。-->
   </listener>
 <filter>
        <filter-name>springSecurityFilterChain</filter-name> <!-- 这个Fileter name：springSecurityFilterChain是有意思的用于查找自动创建的过滤器bean。 -->
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>springSecurityFilterChain</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!--  解决SpringMVC post中文乱码 -->
    <filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
		<init-param>  
            <param-name>forceEncoding</param-name>  
            <param-value>true</param-value>  
        </init-param>  
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
     <!--    注意，这个Servlet名很重要，它决定了Spring应用上下文的加载文件名为spitter-servlet.xml-->
     <!-- 该servlet应用上下文的父亲为前面的全局应用上下文，它可以共享全局应用上下文里的配置与bean等-->
     <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-servlet.xml</param-value>
		</init-param>
        <load-on-startup>1</load-on-startup>
       
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern> <!-- 注意不要配置为/*,不然JSP机制不能直接处理JSP请求等，这不是我们期望的结果-->
    </servlet-mapping>
    
    	<!--静态资源配置到default servlet这比使用SpringMVC的<mvc:resources mapping="/static/**" location="/static/" />要有效，要更好
	更具体的URL模式总是会覆盖/，所以静态资源URL映射可以覆盖前面的配置不走DispatcherServlet，而走由Web服务器容器初始化的default servlet。
	-->
     <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/resource/*</url-pattern>
        <url-pattern>/static/*</url-pattern>
        <url-pattern>/ueditor/*</url-pattern>
    </servlet-mapping>
 <!--  用于普通类中获取session和request -->   
 <listener>
        <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
 </listener>
 
<listener>
    <listener-class>org.apache.tiles.extras.complete.CompleteAutoloadTilesListener</listener-class>
</listener>
<session-config>
  		<session-timeout>30</session-timeout>
</session-config>
</web-app>

```
##  spring-security.xml 
```

<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd
           http://www.springframework.org/schema/security
           http://www.springframework.org/schema/security/spring-security.xsd">
           
     <context:property-placeholder location="classpath:common.properties"/>	
     <beans:bean id="defaultUserAuthenticationService" class="net.xby1993.springmvc.security.service.DefaultUserAuthenticationService"/>
	<global-method-security pre-post-annotations="enabled" jsr250-annotations="enabled" access-decision-manager-ref="methodAccessDecisionManager" >
	 <expression-handler ref="methodSecurityExpressionHandler"/>
	</global-method-security>
	<authentication-manager alias="authenticationManager"> <!-- AuthenticationManager管理AuthenticationProvider管理UserService管理User认证信息获取 -->
		<authentication-provider
        				 user-service-ref="defaultUserAuthenticationService">
			<password-encoder hash="bcrypt" />
		</authentication-provider>
	</authentication-manager>
	<http security="none" pattern="/resource/**" />

	<http  use-expressions="true" access-decision-manager-ref="webAccessDecisionManager">   <!-- use-expressions="true"开启SpEL -->
		<expression-handler ref="webSecurityExpressionHandler"/>
		<!-- invalid-session-url指定处理Session超时的URL  -->
		<!--会话固定攻击防护，有几种选择：changeSessionId(只支持Servlet3.1以上),migrateSession(默认选项，将创建新的会话并复制所有现有特性),newSession,none -->
		<session-management session-fixation-protection="migrateSession" invalid-session-url="${host}/sessiontimeout"> 
			<concurrency-control max-sessions="1" expired-url="${host}/login?maxSessions=1" /> <!--限制用户会话数量，为了启用该特性，记得配置前面所讲的特殊的监听器 -->
		</session-management>
		<!-- <remember-me key="myAppName" token-validity-seconds="2419200" />提供记住我功能，提供一个字段名为_spring_security_remember_me的复选框，如果使用Java配置时字段名为remember-me -->
		<!-- login-page登录页面 ，
			login-processing-url并不也不能是实际地址，只是对于Spring Security一个标识，如果前台提交表单地址是这个就处理登录认证
			authentication-failure-url登录认证失败地址
			default-target-url如果是用户直接输入登录地址进行登录，那么登录成功后将进入该页面，否则就进入原先的页面。
			
			logout-url也是一个标记地址，不能是实际地址。用于Spring-Security处理退出。
			logout-success-url退出成功后进入的页面。
			-->
			
		<form-login login-page="${host}/login?page=1" 
			login-processing-url="/login11"  
			authentication-failure-url="${host}/login?error=1"
			default-target-url="${host}/home/"  
			username-parameter="username"
			password-parameter="password"/>
			<logout logout-url="/logout" logout-success-url="${host}/index"
				invalidate-session="true" delete-cookies="JSESSIONID" />
			<intercept-url pattern="/index" access="permitAll" />
			<intercept-url pattern="/login" access="permitAll" />
			<intercept-url pattern="/logout" access="permitAll" />
			<intercept-url pattern="/captcha/**" access="permitAll" />
			<intercept-url pattern="/home" access="hasAuthority('ROLE_USER')" />
			<intercept-url pattern="/admin/**" access="hasAuthority('ROLE_ADMIN')" />
			<intercept-url pattern="/upload/**" access="hasAuthority('ROLE_ADMIN')" />

	</http>
	 <!-- 角色继承 -->   
   <beans:bean id="roleHierarchy"
      class="org.springframework.security.access.hierarchicalroles.RoleHierarchyImpl">
      <beans:property name="hierarchy"><!-- 角色继承关系 -->
         <beans:value>
            ROLE_ADMIN > ROLE_USER
         </beans:value>
      </beans:property>
   </beans:bean>
     <beans:bean id="roleHierarchyVoter"
      class="org.springframework.security.access.vote.RoleHierarchyVoter">
      <beans:constructor-arg ref="roleHierarchy" />
   </beans:bean>
 <!-- 用于web的ExpressionHandler -->
    <beans:bean id="webSecurityExpressionHandler" name="webSecurityExpressionHandler"
                class="org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler">
        <beans:property name="roleHierarchy" ref="roleHierarchy"/>
    </beans:bean>
 
    <!-- 用于method的ExpressionHandler -->
    <beans:bean id="methodSecurityExpressionHandler" class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler">
        <beans:property name="roleHierarchy" ref="roleHierarchy"/>
    </beans:bean>
   <!-- 用于web的ExpressionHandler -->   
 <beans:bean id="webAccessDecisionManager"  
    class="org.springframework.security.access.vote.AffirmativeBased">  
    <beans:property name="decisionVoters">  
        <beans:list>  
            <beans:bean class="org.springframework.security.web.access.expression.WebExpressionVoter">
                    <beans:property name="expressionHandler" ref="webSecurityExpressionHandler"/>
                </beans:bean>
                <beans:ref bean="roleHierarchyVoter" />  
                <beans:bean class="org.springframework.security.access.vote.AuthenticatedVoter"/>
        </beans:list>  
    </beans:property>  
</beans:bean> 
 <!-- 用于method的AccessDecisionManager -->
    <beans:bean id="methodAccessDecisionManager" class="org.springframework.security.access.vote.AffirmativeBased">
        <beans:constructor-arg>
            <beans:list>
                <beans:bean class="org.springframework.security.access.vote.RoleHierarchyVoter">
                    <beans:constructor-arg ref="roleHierarchy"/>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.vote.AuthenticatedVoter"/>
                <beans:bean class="org.springframework.security.access.prepost.PreInvocationAuthorizationAdviceVoter">
                    <beans:constructor-arg name="pre">
                        <beans:bean class="org.springframework.security.access.expression.method.ExpressionBasedPreInvocationAdvice">
                            <beans:property name="expressionHandler" ref="methodSecurityExpressionHandler"/>
                        </beans:bean>
                    </beans:constructor-arg>
                </beans:bean>
                <beans:bean class="org.springframework.security.access.annotation.Jsr250Voter"/>
            </beans:list>
        </beans:constructor-arg>
    </beans:bean>
</beans:beans>

```      
##  登录表单： 

```

<!--<c:out value=''/>login-->
				<form method="POST" action="/login11" id="loginform">
					<input type="hidden" name="isLoginForm" id="isLoginForm"
						value="true" /> <input type="hidden" id="loginId" name="loginId"
						value="<c:out value='${sessionScope.loginId}'/>" />
					<div class="form-group">
						<span class="error">${error}</span>
					</div>

					<div class="form-group">


						<input type="text" class="form-control" id="name" name="username"
							placeholder="用户名" > <%-- value="${user.name }" --%>

					</div>
					<div class="form-group">

						<input type="password" class="form-control " name="password"
							id="password" placeholder="密码 ">

					</div>
					<div class="row">
						<div class="col-md-7 col-sm-7">
							<input name="captchaCode" class="form-control" type="text"
								id="captchaCode" maxlength="4" placeholder="验证码" />
						</div>
						<div class="col-md-5 col-sm-5">
							<img src="captcha" id="kaptchaImage" />
						</div>
					</div>
					<div style="margin-top: 10px">
						<button type="submit" class="btn btn-primary btn-block">登录</button>
					</div>
				</form>

```
##  LoginController   
```

@Controller
public class LoginController {
	private static final Logger log=LoggerFactory.getLogger(LoginController.class);
	@Autowired
	private UserService uService;
	@Autowired
	private GlobalConfigService gcService;
  @RequestMapping("/login")
	public String login2(HttpServletRequest request, HttpServletResponse resp, LoginDto dto) {
		HttpSession session = request.getSession();
		// 如果早就登录了，那么直接跳转
		if (SecurityContextHolder.getContext().getAuthentication().getPrincipal() instanceof UserDetails) {
			ControllerUtil.redirectURL(resp, "/home");
			log.debug("已登录，直接跳转");
			return null;
		}
		session.setAttribute("loginId", UUID.randomUUID().toString());
		if(dto.getError()!=null){
			request.setAttribute("error", "用户名或密码错误");
			log.debug("用户名或密码错误");
		}else if(dto.getMaxSessions()!=null){
			request.setAttribute("error", "您的帐户在另一处登录，已强制退出");
			log.debug("用户数达到最大会话数");
		}else if(dto.getPage()!=null){
			
		}
		return "login";

	}

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
}

```
##  AdminController 
```

@RequestMapping("/home")
	public ModelAndView success(HttpServletRequest request,HttpServletResponse resp) {
		ModelAndView mv=new ModelAndView();
		HttpSession session = request.getSession();
			
			mv.getModel().put("userName", session.getAttribute(GeneUtil.getCurrentUserName()));
			List<Menu> menuList=menuService.getAllMenuForRole(GeneUtil.getCurRole(request));
			mv.getModel().put("menus", menuList);
			mv.setViewName("admin.index");
		return mv;
	}

```
```

public static String getCurrentUserName(){
		String name=SecurityContextHolder.getContext().getAuthentication().getName();
		return name;
	}
	public static String[] getCurRole(HttpServletRequest request){
//		HttpSession session=request.getSession();
//		Object o=session.getAttribute(GeneUtil.USERTYPE_KEY);
//		return o==null?"":o.toString();
		Collection<? extends GrantedAuthority> colle= SecurityContextHolder.getContext().getAuthentication().getAuthorities();
//		String[] strs=(String[]) colle.toArray();
		List<String> list=new ArrayList<>();
		for(GrantedAuthority auth:colle){
			list.add(auth.getAuthority());
		}
		return list.toArray(new String[list.size()]);
	}

```
##  UserAuthenticationService接口 
```

public interface UserAuthenticationService extends UserDetailsService {
	@Override
	UserPrincipal loadUserByUsername(String username);
	void saveUser(UserPrincipal principal, String newPassword);
}

```
##  DefaultUserAuthenticationService实现 
```

//@Service //自动注册bean
public class DefaultUserAuthenticationService implements UserAuthenticationService //间接实现UserDetailsService接口
{
    private static final Logger log = LoggerFactory.getLogger(DefaultUserAuthenticationService.class);
    private static final SecureRandom RANDOM;
    private static final int HASHING_ROUNDS = 10;

    static
    {
        try
        {
            RANDOM = SecureRandom.getInstance("SHA1PRNG");
        }
        catch(NoSuchAlgorithmException e)
        {
            throw new IllegalStateException(e);
        }
    }

    /**
    *这个UserRepository就是一个JPA仓库
    public interface UserRepository extends CrudRepository<UserPrincipal, Long>
{
    UserPrincipal getByUsername(String username);
}
    */
    @Autowired UserPrincipalDao userRepository; 

    @Override
    @Transactional 
    public UserPrincipal loadUserByUsername(String username) //返回一个UserPrincipal对象
    {
        UserPrincipal principal = this.userRepository.findOneByUsername(username); //从数据库中获取信息
       // make sure the authorities and password are loaded
        principal.getAuthorities().size();
        principal.getPassword();
        return principal;
    }

    @Override
    @Transactional
    public void saveUser(UserPrincipal principal, String newPassword) //保存新密码
    {
        if(newPassword != null && newPassword.length() > 0)
        {
            String salt = BCrypt.gensalt(HASHING_ROUNDS, RANDOM);
            principal.setPassword(BCrypt.hashpw(newPassword, salt));
        }

        this.userRepository.save(principal);
    }
}

```
##  UserPrincipal实体 
```

@Entity
@Table(uniqueConstraints = {
        @UniqueConstraint(name = "UserPrincipal_Username", columnNames = "user_name") //唯一性约束
})
public class UserPrincipal implements UserDetails, CredentialsContainer, Cloneable
{
    private static final long serialVersionUID = 1L;
    private String id;
    private String username;
    private String password;
    private Set<UserAuthority> authorities = new HashSet<>();
    private boolean accountNonExpired;
    private boolean accountNonLocked;
    private boolean credentialsNonExpired;
    private boolean enabled;
    @Id
    @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
    @Column(name = "user_id")
    public String getId()
    {
        return this.id;
    }
    public void setId(String id)
    {
        this.id = id;
    }
    @Override
    @Column(name = "user_name")
    public String getUsername()
    {
        return this.username;
    }

    public void setUsername(String username)
    {
        this.username = username;
    }

    @Override
    public void eraseCredentials()
    {
        this.setPassword(null);
    }

    @Override
//    @ElementCollection(fetch = FetchType.EAGER)
//    @CollectionTable(name = "user_authority", joinColumns = {
//            @JoinColumn(name = "user_id", referencedColumnName = "user_id")
//    })
    @ManyToMany(fetch=FetchType.EAGER,cascade={CascadeType.PERSIST,CascadeType.MERGE,CascadeType.REFRESH}) 
    public Set<UserAuthority> getAuthorities()
    {
        return this.authorities;
    }

    public void setAuthorities(Set<UserAuthority> authorities)
    {
        this.authorities = authorities;
    }

    @Override
    public boolean isAccountNonExpired()
    {
        return this.accountNonExpired;
    }
    public void setAccountNonExpired(boolean accountNonExpired)
    {
        this.accountNonExpired = accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked()
    {
        return this.accountNonLocked;
    }
    public void setAccountNonLocked(boolean accountNonLocked)
    {
        this.accountNonLocked = accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired()
    {
        return this.credentialsNonExpired;
    }
    public void setCredentialsNonExpired(boolean credentialsNonExpired)
    {
        this.credentialsNonExpired = credentialsNonExpired;
    }

    @Override
    public boolean isEnabled()
    {
        return this.enabled;
    }
    public void setEnabled(boolean enabled)
    {
        this.enabled = enabled;
    }

    @Override
    public int hashCode()
    {
        return this.username.hashCode();
    }
    @Override
    public boolean equals(Object other)
    {
        return other instanceof UserPrincipal &&
                ((UserPrincipal)other).id == this.id;
    }

    @Override
    @SuppressWarnings("CloneDoesntDeclareCloneNotSupportedException")
    protected UserPrincipal clone()
    {
        try {
            return (UserPrincipal)super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e); // not possible
        }
    }

    @Override
    public String toString()
    {
        return this.username;
    }
    @Override
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
}


```
##  UserAuthority实体 
```

//@Embeddable
@Entity
public class UserAuthority implements GrantedAuthority
{
	@Id
    @GeneratedValue(generator = "system-uuid")
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
	@Column(name="role_id")
	private String roleId;
    private String authority;
    public UserAuthority() { }
    public UserAuthority(String authority)
    {
        this.authority = authority;
    }
    @Override
    public String getAuthority()
    {
        return this.authority;
    }
    public void setAuthority(String authority)
    {
        this.authority = authority;
    }
	public String getRoleId() {
		return roleId;
	}
	public void setRoleId(String roleId) {
		this.roleId = roleId;
	}
}


```