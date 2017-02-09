title: spring-security权限控制注解 

#  Spring-Security权限控制注解详解 
前面我们介绍过
首先开启注解支持:
```

<!-- <global-method-security secured-annotations=“enabled” /> 针对Spring Security注解@Secured ，它标注的角色值必须以ROLE_开头，而@RolesAllowed则不存在此限制-->
<global-method-security jsr250-annotations=“enabled” /> 针对JSR-250标准注解@RolesAllowed ,推荐使用 位于javax.annotation.security包中。还有@PermitAll等注解

```
使用` @Secured `注解保护方法调用
```

@RolesAllowed("ROLE_ADMIN")或者@RolesAllowed({"ROLE_ADMIN","ROLE_SPITTER"}) 只需具备其中之一
public void add(){}

``` 
**如果想使用SpEL实现调用前后的安全性**。即注解` @PreAuthorize,@PostAuthorize `
启用这一功能：
```

<global-method-security pre-post-annotations=“enabled” />

```
如果没有相关权限。就会抛出AccessDeniedException，Spring Security过滤器捕获该异常并返回一个403 Forbidden错误给用户

@PreAuthorize注解可以访问方法参数，通过"#参数名"的形式。

##  基于方法的权限控制 
之前介绍的都是**基于URL的权限控制**，Spring Security同样**支持对于方法的权限控制**。
  * 可以通过` intercept-methods `对某个bean下面的方法进行权限控制，
  * 也可以通过` pointcut `对整个Service层的方法进行统一的权限控制，
  * 还可以通过` 注解定义 `对单独的某一个方法进行权限控制。

###  intercept-methods定义方法权限控制 
intercept-methods是需要定义在bean元素下的，通过它可以定义对当前的bean的某些方法进行权限控制，具体方法是使用其下的子元素protect进行定义的。protect元素需要指定两个属性，access和method，method表示需要拦截的方法名称，可以使用通配符，access表示执行对应的方法需要拥有的权限，多个权限之间可以使用逗号分隔。
```

   <bean id="userService" class="com.xxx.service.impl.UserServiceImpl">
      <security:intercept-methods>
         <security:protect access="ROLE_USER" method="find*"/>
         <security:protect access="ROLE_ADMIN" method="add*"/>
         <security:protect access="ROLE_ADMIN" method="update*"/>
         <security:protect access="ROLE_ADMIN" method="delete*"/>
      </security:intercept-methods>
   </bean>

```
在上面的配置中表示在执行UserServiceImpl的方法名以find开始的方法时需要当前用户拥有ROLE_USER的权限，在执行方法名以add、update或delete开始的方法时需要拥有ROLE_ADMIN的权限。当访问被拒绝时还是交由ExceptionTranslationFilter处理，这也就意味着如果用户未登录则会引导用户进行登录，否则默认将返回403错误码到客户端。
 
###  使用pointcut定义方法权限控制 
基于pointcut的方法权限控制是通过global-method-security下的protect-pointcut来定义的。可以在global-method-security元素下定义多个protect-pointcut以对不同的pointcut使用不同的权限控制。
```

   <security:global-method-security>
      <security:protect-pointcut access="ROLE_READ" expression="execution(* com.elim.*..*Service.find*(..))"/>
      <security:protect-pointcut access="ROLE_WRITE" expression="execution(* com.elim.*..*Service.*(..))"/>
   </security:global-method-security>

```
上面的定义表示我们在执行com.elim包或其子包下任意以Service结尾的类，其方法名以find开始的所有方法时都需要用户拥有ROLE_READ的权限，对于com.elim包或其子包下任意以Service结尾的类的其它方法在执行时都需要ROLE_WRITE的权限。需要注意的是对应的类需要是定义在ApplicationContext中的bean才行。此外同对于URL的权限控制一样，当定义多个protect-pointcut时更具有特性的应当先定义，**因为在pointcut匹配的时候是按照声明顺序进行匹配的，一旦匹配上了后续的将不再进行匹配了。**

###  使用注解定义方法权限控制 
了解使用注解的优势：
<note important>授权规则是一个真正的契约，这一接口的契约作用吻合。授权并不是实现细节。因此它们不应该属于实现，而是应该属于接口。` 通过使用这些注解在接口中声明授权规则 `，从而使它们成为契约的一部分，并应用到所有的实现中。</note>
**基于注解的方法权限控制也是需要通过global-method-security元素定义来进行启用的**。Spring Security在方法的权限控制上支持三种类型的注解，**JSR-250注解、@Secured注解和支持表达式的注解。**这三种注解默认都是没有启用的，需要单独通过global-method-security元素的对应属性进行启用。
**1.3.1   JSR-250注解**
要使用JSR-250注解，首先我们需要通过设置global-method-security元素的jsr250-annotation=”enabled”来启用基于JSR-250注解的支持，默认为disabled。
```

<security:global-method-security jsr250-annotations="enabled"/>

```
此外，还需要确保添加了jsr250-api到我们的类路径下。之后就可以在我们的Service方法上使用JSR-250注解进行权限控制了。
```

@Service
@RolesAllowed("ROLE_ADMIN")
public class UserServiceImpl implements UserService {
 
   @RolesAllowed({"ROLE_USER", "ROLE_ADMIN"})
   public User find(int id) {
      System.out.println("find user by id............." + id);
      return null;
   }
   @RolesAllowed("ROLE_USER")
   public List<User> findAll() {
      System.out.println("find all user...............");
      return null;
   }
 
}

```
上面的代码表示执行UserServiceImpl里面所有的方法都需要角色ROLE_ADMIN，其中findAll()方法的执行需要ROLE_USER角色，而find()方法的执行对于ROLE_USER或者ROLE_ADMIN角色都可以。
顺便介绍一下JSR-250中对权限支持的注解。
RolesAllowed表示访问对应方法时所应该具有的角色。其可以标注在类上，也可以标注在方法上，当标注在类上时表示其中所有方法的执行都需要对应的角色，当标注在方法上表示执行该方法时所需要的角色，当方法和类上都使用了@RolesAllowed进行标注，` 则方法上的@RolesAllowed将覆盖类上的@RolesAllowed，即方法上的@RolesAllowed将对当前方法起作用。@RolesAllowed的值是由角色名称组成的数组。 `
@PermitAll表示允许所有的角色进行访问，也就是说不进行权限控制。@PermitAll可以标注在方法上也可以标注在类上，当标注在方法上时则只对对应方法不进行权限控制，而标注在类上时表示对类里面所有的方法都不进行权限控制。
（1）当@PermitAll标注在类上，而@RolesAllowed标注在方法上时则按照@RolesAllowed将覆盖@PermitAll，即需要@RolesAllowed对应的角色才能访问。
（2）当@RolesAllowed标注在类上，而@PermitAll标注在方法上时则对应的方法也是不进行权限控制的。
（3）当在方法上同时使用了@PermitAll和@RolesAllowed时**先定义的将发生作用**，而都定义在类上时则是反过来的，即后定义的将发生作用（这个没多大的实际意义，实际应用中不会有这样的定义）。
@DenyAll是和PermitAll相反的，表示无论什么角色都不能访问。@DenyAll只能定义在方法上。你可能会有疑问使用@DenyAll标注的方法无论拥有什么权限都不能访问，那还定义它干啥呢？使用@DenyAll定义的方法只是在我们的权限控制中不能访问，脱离了权限控制还是可以访问的。
 
**1.3.2  @Secured注解**
@Secured是由Spring Security定义的用来支持方法权限控制的注解。它的使用也是需要启用对应的支持才会生效的。通过设置global-method-security元素的secured-annotations=”enabled”可以启用Spring Security对使用@Secured注解标注的方法进行权限控制的支持，其值默认为disabled。
```

   <security:global-method-security secured-annotations="enabled"/>

```
```

@Service
public class UserServiceImpl implements UserService {
 
   @Secured("ROLE_ADMIN")
   public void addUser(User user) {
      System.out.println("addUser................" + user);
   }
 
   @Secured("ROLE_USER")
   public List<User> findAll() {
      System.out.println("find all user...............");
      return null;
   }
 
}

```
在上面的代码中我们使用@Secured定义了只有拥有ROLE_ADMIN角色的用户才能调用方法addUser()，只有拥有ROLE_USER角色的用户才能调用方法findAll()。
 
**1.3.3   支持表达式的注解**
Spring Security中定义了四个支持使用表达式的注解，分别是` @PreAuthorize、@PostAuthorize、@PreFilter和@PostFilter `。其中前两者可以用来在方法调用前或者调用后进行权限检查，后两者可以用来对集合类型的参数或者返回值进行过滤。要使它们的定义能够对我们的方法的调用产生影响我们需要设置global-method-security元素的pre-post-annotations=”enabled”，默认为disabled。
```

   <security:global-method-security pre-post-annotations="disabled"/>

```
**使用@PreAuthorize和@PostAuthorize进行访问控制**
@PreAuthorize可以用来控制一个方法是否能够被调用。
```

@Service
public class UserServiceImpl implements UserService {
 
   @PreAuthorize("hasRole('ROLE_ADMIN') and #user.id==0")
   public void addUser(User user) {
      System.out.println("addUser................" + user);
   }
 
   @PreAuthorize("hasRole('ROLE_USER') or hasRole('ROLE_ADMIN')")
   public User find(int id) {
      System.out.println("find user by id............." + id);
      return null;
   }
 
}

```
在上面的代码中我们定义了只有拥有角色ROLE_ADMIN的用户才能访问adduser()方法，而访问find()方法需要有ROLE_USER角色或ROLE_ADMIN角色。` 使用表达式时我们还可以在表达式中使用方法参数。 `
不过在接口中可能会遇到问题。解决方案是使用` @org.springframework.security.access.method.P `("user") User user注解形式指定参数。
```

public class UserServiceImpl implements UserService {
 
   /**
    * 限制只能查询Id小于10的用户
    */
   @PreAuthorize("#id<10")
   public User find(@P("id") int id) { //@P显示指定
      System.out.println("find user by id........." + id);
      return null;
   }
  
   /**
    * 限制只能查询自己的信息
    */
   @PreAuthorize("principal.username.equals(#username)")
   public User find(String username) {
      System.out.println("find user by username......" + username);
      return null;
   }
 
   /**
    * 限制只能新增用户名称为abc的用户
    */
   @PreAuthorize("#user.name.equals('abc')")
   public void add(User user) {
      System.out.println("addUser............" + user);
   }
 
}

```
在上面代码中我们定义了调用find(int id)方法时，只允许参数id小于10的调用；调用find(String username)时只允许username为当前用户的用户名；定义了调用add()方法时只有当参数user的name为abc时才可以调用。
 有时候可能你会想在方法调用完之后进行权限检查，这种情况比较少，但是如果你有的话，Spring Security也为我们提供了支持，通过@PostAuthorize可以达到这一效果。使用@PostAuthorize时我们可以使用内置的表达式` returnObject表示方法的返回值 `。我们来看下面这一段示例代码。
```

   @PostAuthorize("returnObject.id%2==0")
   public User find(int id) {
      User user = new User();
      user.setId(id);
      return user;
   }

```
上面这一段代码表示将在方法find()调用完成后进行权限检查，如果返回值的id是偶数则表示校验通过，否则表示校验失败，将抛出AccessDeniedException。需要注意的是@PostAuthorize是在方法调用完成后进行权限检查，它不能控制方法是否能被调用，只能在方法调用完成后检查权限决定是否要抛出` AccessDeniedException `。
 
**使用@PreFilter和@PostFilter进行过滤**
使用@PreFilter和@PostFilter可以` 对集合类型的参数或返回值进行过滤 `。` 使用@PreFilter和@PostFilter时，Spring Security将移除使对应表达式的结果为false的元素。 `
但是这个不支持分页，所以一般情况下尽量少用。
 ```

@PostFilter("filterObject.id%2==0")
   public List<User> findAll() {
      List<User> userList = new ArrayList<User>();
      User user;
      for (int i=0; i<10; i++) {
         user = new User();
         user.setId(i);
         userList.add(user);
      }
      return userList;
   }

```
上述代码表示将对返回结果中id不为偶数的user进行移除。` filterObject是使用@PreFilter和@PostFilter时的一个内置表达式，表示集合中的当前对象。 `当@PreFilter标注的方法拥有多个集合类型的参数时，需要通过@PreFilter的filterTarget属性指定当前@PreFilter是针对哪个参数进行过滤的。如下面代码就通过filterTarget指定了当前@PreFilter是用来过滤参数ids的。
```

   @PreFilter(filterTarget="ids", value="filterObject%2==0")
   public void delete(List<Integer> ids, List<String> usernames) {
      ...
   }

```

参考：http://haohaoxuexi.iteye.com/blog/2262363
http://haohaoxuexi.iteye.com/blog/2247073