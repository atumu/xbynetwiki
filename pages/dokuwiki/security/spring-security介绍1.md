title: spring-security介绍1 

#  Spring-Security核心类与认证简介 
##  了解Spring-Security基础类 
Spring Security提供了认证和授权服务。专门设计为结合Spring使用的用于Web应用程序。这一点与同是安全框架的Shiro不同。
Spring Security的一些接口：
  * org.springframework.security.core.**Authentication**包含用户认证相关信息。如：
getPrincipal()(如用户名)、getCredentials()（如密码）、getAuthorities()、isAuthenticated()、getDetails()、setAuthenticated(boolean isAuthenticated)

  * org.springframework.security.authentication.**AuthenticationProvider**认证服务的提供者：
Authentication	authenticate(Authentication authentication)该方法接受一个未认证的Authentication(其中包含了用于身份证明的凭证)。然后将该Authentication标记为已认证并返回相同一个代表已认证、完全不同的Authentication，或者抛出一个AuthenticationException异常（如果认证失败了的话）Spring Security为JDBC、LDAP、CAS、JAAS、OpenID等都提供了AuthenticationProvider内建实现。此外，我们还可以自定义实现。
boolean	supports(Class<?> authentication) 支持的Authentication实现类型。

org.springframework.security.core.context.` SecurityContext `接口表示的是当前应用的安全上下文。通过此接口可以获取和设置当前的认证对象。
org.springframework.security.core.` Authentication `接口用来表示此**认证对象**。通过认证对象的方法可以判断当前用户是否已经通过认证，以及获取当前认证用户的相关信息，包括用户名、密码和权限等。要使用此认证对象，首先需要获取到 SecurityContext 对象。通过 org.springframework.security.core.context.` SecurityContextHolder `(默认情况下，SecurityContextHolder使用 ThreadLocal来保存 SecurityContext对象。) 类提供的静态方法 getContext() 就可以获取。再通过 SecurityContext对象的 getAuthentication()就可以得到认证对象。` 通过认证对象的 getPrincipal() 方法就可以获得当前的认证主体，通常是 UserDetails 接口的实现。 `
典型的认证过程就是当用户输入了用户名和密码之后，UserDetailsService通过用户名找到对应的 UserDetails 对象，接着比较密码是否匹配。如果不匹配，则返回出错信息；如果匹配的话，说明用户认证成功。

###  Authentication 
Authentication是一个接口，**用来表示用户认证信息的**，在用户登录认证之前相关信息会封装为一个Authentication具体实现类的对象，在登录认证成功之后又会生成一个信息更全面，包含用户权限等信息的Authentication对象，然后把它保存在SecurityContextHolder所持有的SecurityContext中，供后续的程序进行调用，如访问权限的鉴定等。

###  SecurityContextHolder 
SecurityContextHolder是用来保存SecurityContext的。SecurityContext中含有当前正在访问系统的用户的详细信息。默认情况下，SecurityContextHolder将使用ThreadLocal来保存SecurityContext，这也就意味着在处于同一线程中的方法中我们可以从ThreadLocal中获取到当前的SecurityContext。
SecurityContextHolder中定义了一系列的静态方法，而这些静态方法内部逻辑基本上都是通过SecurityContextHolder持有的SecurityContextHolderStrategy来实现的，如getContext()、setContext()、clearContext()等。而默认使用的strategy就是基于ThreadLocal的ThreadLocalSecurityContextHolderStrategy。另外，Spring Security还提供了两种类型的strategy实现，GlobalSecurityContextHolderStrategy和InheritableThreadLocalSecurityContextHolderStrategy，前者表示全局使用同一个SecurityContext，如C/S结构的客户端；后者使用InheritableThreadLocal来存放SecurityContext，即子线程可以使用父线程中存放的变量。

在程序的任何地方，通过如下方式我们可以获取到当前用户的用户名。
```

 public String getCurrentUsername() {
      Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
      if (principal instanceof UserDetails) {
         return ((UserDetails) principal).getUsername();
      }
      if (principal instanceof Principal) {
         return ((Principal) principal).getName();
      }
      return String.valueOf(principal);
   }

```
通过Authentication.getPrincipal()可以获取到代表当前用户的信息，这个对象通常是` UserDetails `的实例。获取当前用户的用户名是一种比较常见的需求，关于上述代码其实Spring Security在Authentication中的实现类中已经为我们做了相关实现，所以获取当前用户的用户名最简单的方式应当如下。
```

 public String getCurrentUsername() {
      return SecurityContextHolder.getContext().getAuthentication().getName();
   }

```
 此外，调用SecurityContextHolder.getContext()获取SecurityContext时，如果对应的SecurityContext不存在，则Spring Security将为我们建立一个空的SecurityContext并进行返回。

###  AuthenticationManager和AuthenticationProvider 
AuthenticationManager是一个用来处理认证（Authentication）请求的接口。在其中只定义了一个方法authenticate()，该方法只接收一个代表认证请求的Authentication对象作为参数，如果认证成功，则会返回一个封装了当前用户权限等信息的Authentication对象进行返回。
Authentication authenticate(Authentication authentication) throws AuthenticationException;
在Spring Security中，AuthenticationManager的**默认实现是ProviderManager，而且它不直接自己处理认证请求，而是委托给其所配置的AuthenticationProvider列表**，然后会依次使用每一个AuthenticationProvider进行认证，如果有一个AuthenticationProvider认证后的结果不为null，则表示该AuthenticationProvider已经认证成功，之后的AuthenticationProvider将不再继续认证。然后直接以该AuthenticationProvider的认证结果作为ProviderManager的认证结果。如果所有的AuthenticationProvider的认证结果都为null，则表示认证失败，将抛出一个ProviderNotFoundException。**` 校验认证请求最常用的方法是根据请求的用户名加载对应的UserDetails，然后比对UserDetails的密码与认证请求的密码是否一致，一致则表示认证通过。 `** **Spring Security内部的` DaoAuthenticationProvider `就是使用的这种方式。其内部使用` UserDetailsService来负责加载UserDetails， `**UserDetailsService将在下节讲解。**在认证成功以后会使用加载的UserDetails来封装要返回的Authentication对象，加载的UserDetails对象是包含用户权限等信息的。认证成功返回的Authentication对象将会保存在当前的SecurityContext中。**

###  UserDetailsService 
` 通过Authentication.getPrincipal()的返回类型是Object，但很多情况下其返回的其实是一个UserDetails的实例 `。**UserDetails是Spring Security中一个核心的接口**。其中定义了一些可以获取用户名、密码、权限等与认证相关的信息的方法。Spring Security内部使用的UserDetails实现类大都是内置的User类，我们如果要使用UserDetails时也可以直接使用该类。在Spring Security内部很多地方需要使用用户信息的时候基本上都是使用的UserDetails，比如在登录认证的时候。登录认证的时候Spring Security会通过` UserDetailsService `的loadUserByUsername()方法获取对应的UserDetails进行认证，**认证通过后会将该UserDetails赋给认证通过的Authentication的principal，然后再把该Authentication存入到SecurityContext中。**之后如果需要使用用户信息的时候就是通过SecurityContextHolder获取存放在SecurityContext中的Authentication的principal。
通常我们需要在应用中获取当前用户的其它信息，如Email、电话等。这时存放在Authentication的principal中只包含有认证相关信息的UserDetails对象可能就不能满足我们的要求了。这时我们可以实现自己的UserDetails，在该实现类中我们可以定义一些获取用户其它信息的方法，这样将来我们就可以直接从当前SecurityContext的Authentication的principal中获取这些信息了。上文已经提到了UserDetails是通过UserDetailsService的loadUserByUsername()方法进行加载的。UserDetailsService也是一个接口，我们也需要实现自己的UserDetailsService来加载我们自定义的UserDetails信息。然后把它指定给AuthenticationProvider即可。如下是一个配置UserDetailsService的示例。
```

   <!-- 用于认证的AuthenticationManager -->
   <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider
         user-service-ref="userDetailsService" />
   </security:authentication-manager>
 
   <bean id="userDetailsService"
      class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
      <property name="dataSource" ref="dataSource" />
   </bean>

```
上述代码中我们使用的JdbcDaoImpl是Spring Security为我们提供的UserDetailsService的实现，另外Spring Security还为我们提供了UserDetailsService另外一个实现，InMemoryDaoImpl。

###  JdbcDaoImpl 
JdbcDaoImpl允许我们从数据库来加载UserDetails，其底层使用的是Spring的JdbcTemplate进行操作，所以我们需要给其指定一个数据源。此外，我们需要通过usersByUsernameQuery属性指定通过username查询用户信息的SQL语句；通过authoritiesByUsernameQuery属性指定通过username查询用户所拥有的权限的SQL语句；如果我们通过设置JdbcDaoImpl的enableGroups为true启用了用户组权限的支持，则我们还需要通过groupAuthoritiesByUsernameQuery属性指定根据username查询用户组权限的SQL语句。当这些信息都没有指定时，将使用默认的SQL语句，默认的SQL语句如下所示。
```

select username, password, enabled from users where username=? --根据username查询用户信息
select username, authority from authorities where username=? --根据username查询用户权限信息
select g.id, g.group_name, ga.authority from groups g, groups_members gm, groups_authorities ga where gm.username=? and g.id=ga.group_id and g.id=gm.group_id --根据username查询用户组权限

```
使用默认的SQL语句进行查询时意味着我们对应的数据库中应该有对应的表和表结构，Spring Security为我们提供的默认表的创建脚本如下。
```

create table users(
      username varchar_ignorecase(50) not null primary key,
      password varchar_ignorecase(50) not null,
      enabled boolean not null);
 
create table authorities (
      username varchar_ignorecase(50) not null,
      authority varchar_ignorecase(50) not null,
      constraint fk_authorities_users foreign key(username) references users(username));
      create unique index ix_auth_username on authorities (username,authority);
     
create table groups (
  id bigint generated by default as identity(start with 0) primary key,
  group_name varchar_ignorecase(50) notnull);
 
create table group_authorities (
  group_id bigint notnull,
  authority varchar(50) notnull,
  constraint fk_group_authorities_group foreign key(group_id) references groups(id));
 
create table group_members (
  id bigint generated by default as identity(start with 0) primary key,
  username varchar(50) notnull,
  group_id bigint notnull,
  constraint fk_group_members_group foreign key(group_id) references groups(id));

```
此外，使用jdbc-user-service元素时在底层Spring Security默认使用的就是JdbcDaoImpl。
```

   <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider>
         <!-- 基于Jdbc的UserDetailsService实现，JdbcDaoImpl -->
         <security:jdbc-user-service data-source-ref="dataSource"/>
      </security:authentication-provider>
   </security:authentication-manager>

```

###  GrantedAuthority 
**Authentication的getAuthorities()可以返回当前Authentication对象拥有的权限，即当前用户拥有的权限。**其返回值是一个GrantedAuthority类型的数组，` 每一个GrantedAuthority对象代表赋予给当前用户的一种权限。 `GrantedAuthority是一个接口，其通常是通过UserDetailsService进行加载，然后赋予给UserDetails的。
**GrantedAuthority中只定义了一个getAuthority()方法，该方法返回一个字符串，表示对应权限的字符串表示，**如果对应权限不能用字符串表示，则应当返回null。
Spring Security针对GrantedAuthority有一个简单实现` SimpleGrantedAuthority `。该类只是简单的接收一个表示权限的字符串。Spring Security内部的所有AuthenticationProvider都是使用SimpleGrantedAuthority来封装Authentication对象。

##  认证过程 
1、用户使用用户名和密码进行登录。
2、Spring Security将获取到的用户名和密码封装成一个` 实现了Authentication接口的UsernamePasswordAuthenticationToken。 `
3、将上述产生的` token对象传递给AuthenticationManager进行登录认证 `。
4、AuthenticationManager认证成功后将会返回一个封装了用户权限等信息的` Authentication对象。 `
5、通过调用SecurityContextHolder.getContext().setAuthentication(...)将AuthenticationManager返回的Authentication对象赋予给当前的SecurityContext。
上述介绍的就是Spring Security的认证过程。在认证成功后，用户就可以继续操作去访问其它受保护的资源了，但是在访问的时候将会使用保存在SecurityContext中的Authentication对象进行相关的权限鉴定。
 
1.2     Web应用的认证过程

如果用户直接访问登录页面，那么认证过程跟上节描述的基本一致，只是在认证完成后将跳转到指定的成功页面，默认是应用的根路径。如果用户直接访问一个受保护的资源，那么认证过程将如下：
1、引导用户进行登录，通常是重定向到一个基于form表单进行登录的页面，具体视配置而定。
2、用户输入用户名和密码后请求认证，后台还是会像上节描述的那样获取用户名和密码封装成一个UsernamePasswordAuthenticationToken对象，然后把它传递给AuthenticationManager进行认证。
3、如果认证失败将继续执行步骤1，` 如果认证成功则会保存返回的Authentication到SecurityContext，然后默认会将用户重定向到之前访问的页面。 `
4、用户登录认证成功后再次访问之前受保护的资源时就会对用户进行权限鉴定，如不存在对应的访问权限，则会返回403错误码。
 
在上述步骤中将有很多不同的类参与，但其中主要的参与者是ExceptionTranslationFilter。
 
1.2.1   ExceptionTranslationFilter
ExceptionTranslationFilter是用来处理来自AbstractSecurityInterceptor抛出的AuthenticationException和AccessDeniedException的。**AbstractSecurityInterceptor是Spring Security用于拦截请求进行权限鉴定的**，其拥有两个具体的子类，` 拦截方法调用的MethodSecurityInterceptor和拦截URL请求的FilterSecurityInterceptor。 `当ExceptionTranslationFilter捕获到的是AuthenticationException时将调用AuthenticationEntryPoint引导用户进行登录；如果捕获的是AccessDeniedException，但是用户还没有通过认证，则调用AuthenticationEntryPoint引导用户进行登录认证，否则将返回一个表示不存在对应权限的403错误码。
 
**1.2.2  在request之间共享SecurityContext**
可能你早就有这么一个疑问了，既然SecurityContext是存放在ThreadLocal中的，而且在每次权限鉴定的时候都是从ThreadLocal中获取SecurityContext中对应的Authentication所拥有的权限，并且不同的request是不同的线程，为什么每次都可以从ThreadLocal中获取到当前用户对应的SecurityContext呢？在Web应用中这是通过SecurityContextPersistentFilter实现的，默认情况下其会在每次请求开始的时候从session中获取SecurityContext，然后把它设置给SecurityContextHolder，在请求结束后又会将SecurityContextHolder所持有的SecurityContext保存在session中，并且清除SecurityContextHolder所持有的SecurityContext。**这样当我们第一次访问系统的时候，SecurityContextHolder所持有的SecurityContext肯定是空的，待我们登录成功后，SecurityContextHolder所持有的SecurityContext就不是空的了，且包含有认证成功的Authentication对象，待请求结束后我们就会将SecurityContext存在session中，等到下次请求的时候就可以从session中获取到该SecurityContext并把它赋予给SecurityContextHolder了，由于SecurityContextHolder已经持有认证过的Authentication对象了，所以下次访问的时候也就不再需要进行登录认证了。**

参考：http://haohaoxuexi.iteye.com/blog/2156765