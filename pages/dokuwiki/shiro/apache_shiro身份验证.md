title: apache_shiro身份验证 

#  apache Shiro身份验证 
在shiro中，用户需要提供` principals （身份）和credentials `（证明）给shiro，从而应用能验证用户身份：
另外两个相关的概念是之前提到的` Subject及Realm `，分别是主体及验证主体的数据源。
  * Authentication：身份认证，即用户提供一些信息来证明自己的身份。如用户名和密码，licence等。
  * Principals ：主体的“标识属性”，可以是任意标识，例如用户名，身份证号码，手机号码等。Principals可以有多个，但是必须有一个主要的Principal（Primary Principal），这个标识，必须是唯一的。
  * Credentials：凭据，即只有主体知道或具有的秘密值，例如密码或数字证书，或者某些生物特征，例如指纹，视网膜等。
  * Principals/ Credentials的配对，**他最常见的例子是用户名/密码。**

**环境准备**
```

<dependencies>  

    <dependency>  
        <groupId>org.apache.shiro</groupId>  
        <artifactId>shiro-core</artifactId>  
        <version>1.2.4</version>  
    </dependency>  
</dependencies>  

``` 

##  身份认证（Authenticating Subjects） 
身份认证可概括为3个步骤。
**1.收集用户提供的身份标识和凭据**
```

//使用最常见的用户名密码的方式  
UsernamePasswordToken token = new UsernamePasswordToken(username, password);  
//记住我  
token.setRememberMe(true);

```
在这个例子中，我们使用` UsernamePasswordToken `，支持最常见的**用户名/口令**的认证方式。它是org.apache.shiro.authc.` AuthenticationToken `接口的一个实现。
值得注意的是，**Shiro并不在乎你是如何获得这些信息的**（上面例子中的username,和password）：这两个参数可能是用户通过http请求提交过来的，也可能是其他的方式获得的。

**2.提交用户的身份标识/凭据信息进行身份验证。**
在身份标识和凭据被收集并存入UsernamePasswordToken实例中，我们**需要提交Token给Shiro去尝试执行身份认证。**
```

//获取Subject对象  
Subject currentUser = SecurityUtils.getSubject();  
//传入上一步骤创建的token对象，登录，即进行身份验证操作。  
currentUser.login(token);

```

**3.验证成功或失败。**
我们可以通过` subject.isAuthenticated() `来判断是否验证成功，验证成功返回true,否则返回false。
如果登录失败，例如用户名或者密码错误，或者请求次数过多。我们可以根据Shiro提供的丰富的运行时错误，来来判断是由于什么引起的错误。
```

try {  
currentUser.login(token);  
} catch ( UnknownAccountException uae ) { //用户名未知...  
} catch ( IncorrectCredentialsException ice ) {//凭据不正确，例如密码不正确 ...  
} catch ( LockedAccountException lae ) { //用户被锁定，例如管理员把某个用户禁用...  
} catch ( ExcessiveAttemptsException eae ) {//尝试认证次数多余系统指定次数 ...  
} catch ( AuthenticationException ae ) {  
//其他未指定异常  
}  
//未抛出异常，程序正常向下执行。

```
##  登录/退出 

1、首先准备一些用户身份/凭据（shiro.ini）
[users]  
zhang=123  
wang=123  
此处使用ini配置文件，**通过[users]指定了两个主体**：zhang/123、wang/123。
2、测试用例（com.github.zhangkaitao.shiro.chapter2.LoginLogoutTest） 
```

@Test  
public void testHelloworld() {  
    //1、获取SecurityManager工厂，此处使用Ini配置文件初始化SecurityManager  
    Factory<org.apache.shiro.mgt.SecurityManager> factory =  
            new IniSecurityManagerFactory("classpath:shiro.ini");  
    //2、得到SecurityManager实例 并绑定给SecurityUtils  
    org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();  
    SecurityUtils.setSecurityManager(securityManager);  
    //3、得到Subject及创建用户名/密码身份验证Token（即用户身份/凭证）  
    Subject subject = SecurityUtils.getSubject();  
    UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");  
  
    try {  
        //4、登录，即身份验证  
        subject.login(token);  
    } catch (AuthenticationException e) {  
        //5、身份验证失败  
    }  
  
    Assert.assertEquals(true, subject.isAuthenticated()); //断言用户已经登录  
  
    //6、退出  
    subject.logout();  
}  

```
2.1、首先通过new IniSecurityManagerFactory并指定一个ini配置文件来创建一个SecurityManager工厂；
2.2、接着获取SecurityManager并绑定到SecurityUtils，**这是一个全局设置，设置一次即可；**

2.3、通过SecurityUtils` 得到Subject，其会自动绑定到当前线程 `；如果在web环境在请求结束时需要解除绑定；然后获取身份验证的Token，如用户名/密码；
2.4、调用subject.login方法进行登录，其会自动委托给SecurityManager.login方法进行登录；
2.5、**如果身份验证失败请捕获AuthenticationException或其子类**，常见的如： DisabledAccountException（禁用的帐号）、LockedAccountException（锁定的帐号）、UnknownAccountException（错误的帐号）、ExcessiveAttemptsException（登录失败次数过多）、IncorrectCredentialsException （错误的凭证）、ExpiredCredentialsException（过期的凭证）等，具体请查看其继承关系；对于页面的错误消息展示，最好使用如“用户名/密码错误”而不是“用户名错误”/“密码错误”，防止一些恶意用户非法扫描帐号库；
2.6、最后可以调用` subject.logout `退出，其会自动委托给SecurityManager.logout方法退出。
注：调用了注销的方法，用户的session将会被作废，同时，在web系统中，rememberMe的cookie将会被删除。
因为在Web程序中记住身份信息往往使用Cookies，**` 而Cookies只能在Response提交时才能被删除，所以强烈要求在为最终用户调用subject.logout()之后立即将用户引导到一个新页面，确保任何与安全相关的Cookies如期删除，这是Http本身Cookies功能的限制而不是Shiro的限制。 `**
**从如上代码可总结出身份验证的步骤：**
1、收集用户身份/凭证，即如用户名/密码；
2、调用Subject.login进行登录，如果失败将得到相应的AuthenticationException异常，根据异常提示用户错误信息；否则登录成功；
3、最后调用Subject.logout进行退出操作。
 
如上测试的几个问题：
1、用户名/密码硬编码在ini配置文件，以后需要改成如数据库存储，且密码需要加密存储；
2、用户身份Token可能不仅仅是用户名/密码，也可能还有其他的，如登录时允许用户名/邮箱/手机号同时登录

##  身份认证流程 

![](/data/dokuwiki/shiro/pasted/20151027-142524.png)
流程如下：
1、首先调用Subject.login(token)进行登录，其会自动委托给Security Manager，调用之前必须通过SecurityUtils. setSecurityManager()设置；
2、SecurityManager负责真正的身份验证逻辑；**它会委托给Authenticator进行身份验证**；
3、**Authenticator才是真正的身份验证者**，Shiro API中核心的身份认证入口点，此处可以自定义插入自己的实现；
4、Authenticator可能会委托给相应的**AuthenticationStrategy进行多Realm身份验证**，默认ModularRealmAuthenticator会调用AuthenticationStrategy进行多Realm身份验证；
5、Authenticator会把相应的token传入Realm，从Realm获取身份验证信息，如果没有返回/抛出异常表示身份验证失败了。此处**可以配置多个Realm**，将按照相应的顺序及策略进行访问。

##  已记住和已认证（Remembered vs. Authenticated） 
如上例所示，shiro支持在登录过程中执行”remember me”，在此值得指出，**一个已记住的Subject（remembered Subject）和一个正常通过认证的Subject（authenticated Subject）在shiro是完全不同的。**
**Remembered（已记住）**
一个 remembered 主体，不是匿名的，有一个已知的ID(identity)（subject.getPrincipals()不为空）。而这个记住的过程，发生在之前的身份认证过程中。可以通过` subject.isRemembered( `)来判断是已记住状态，已记住会返true。
**Authenticated（已认证）**
一个已认证的Subject是指在当前会话（session）中被成功地验证过了。即在认证过程没有抛出异常，如果是已认证的主体，` subject.isAuthenticated() `返回true。
    
**互斥（Mutually Exclusive）**
以` 记住和已认证是互斥的 `，也就对于他们的判断（subject.isRemembered()和subject.isAuthenticated()），**一个返回true，则另一个返回false。**
##  Realm 
**Realm：域，Shiro从从Realm获取安全数据（如用户、角色、权限**），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。**如我们之前的ini配置方式将使用org.apache.shiro.realm.text.IniRealm。**
 
` org.apache.shiro.realm.Realm接口 `如下： 
```

String getName(); //返回一个唯一的Realm名字  
boolean supports(AuthenticationToken token); //判断此Realm是否支持此Token  
AuthenticationInfo getAuthenticationInfo(AuthenticationToken token)  
 throws AuthenticationException;  //根据Token获取认证信息  

``` 
###  单Realm配置 
1、**自定义Realm实现**（com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1）：  
```

public class MyRealm1 implements Realm {  
    @Override  
    public String getName() {  
        return "myrealm1";  
    }  
    @Override  
    public boolean supports(AuthenticationToken token) {  
        //仅支持UsernamePasswordToken类型的Token  
        return token instanceof UsernamePasswordToken;   
    }  
    @Override  
    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {  
        String username = (String)token.getPrincipal();  //得到用户名  
        String password = new String((char[])token.getCredentials()); //得到密码  
        if(!"zhang".equals(username)) {  
            throw new UnknownAccountException(); //如果用户名错误  
        }  
        if(!"123".equals(password)) {  
            throw new IncorrectCredentialsException(); //如果密码错误  
        }  
        //如果身份认证验证成功，返回一个AuthenticationInfo实现；  
        return new SimpleAuthenticationInfo(username, password, getName());  
    }  
}  

``` 
 
**2、ini配置文件指定自定义Realm实现(shiro-realm.ini) ** 
```

#声明一个realm  
myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1  
#指定securityManager的realms实现  
securityManager.realms=$myRealm1 

```  
通过$name来引入之前的realm定义
 
3、测试用例请参考com.github.zhangkaitao.shiro.chapter2.LoginLogoutTest的testCustomRealm测试方法，只需要把之前的shiro.ini配置文件改成shiro-realm.ini即可。

###  多Realm配置 
1、ini配置文件（shiro-multi-realm.ini）  
```

#声明一个realm  
myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1  
myRealm2=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm2  
#指定securityManager的realms实现  
securityManager.realms=$myRealm1,$myRealm2 

```  
**securityManager会按照realms指定的顺序进行身份认证。**此处我们使用显示指定顺序的方式指定了Realm的顺序，如果删除“securityManager.realms=$myRealm1,$myRealm2”，那么securityManager会按照realm声明的顺序进行使用（即无需设置realms属性，其会自动发现），当我们显示指定realm后，其他没有指定realm将被忽略，如“securityManager.realms=$myRealm1”，那么myRealm2不会被自动设置进去。
 
2、测试用例请参考com.github.zhangkaitao.shiro.chapter2.LoginLogoutTest的testCustomMultiRealm测试方法。

###  Shiro默认提供的Realm 
![](/data/dokuwiki/shiro/pasted/20151027-143142.png)

以后一般继承` AuthorizingRealm `（授权）即可；其继承了AuthenticatingRealm（即身份验证），而且也间接继承了CachingRealm（带有缓存实现）。其中主要默认实现如下：
  * org.apache.shiro.realm.text.` IniRealm `：**[users]部分指定用户名/密码及其角色；[roles]部分指定角色即权限信息；**
  * org.apache.shiro.realm.text.` PropertiesRealm `： user.username=password,role1,role2指定用户名/密码及其角色；role.role1=permission1,permission2指定角色及权限信息；
  * org.apache.shiro.realm.jdbc.` JdbcRealm `：通过sql查询相应的信息，如“select password from users where username = ?”获取用户密码，“select password, password_salt from users where username = ?”获取用户密码及盐；“select role_name from user_roles where username = ?”获取用户角色；“select permission from roles_permissions where role_name = ?”获取角色对应的权限信息；也可以调用相应的api进行自定义sql；
##  JDBC Realm使用 
1、数据库及依赖
```

<dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
    <version>5.1.25</version>  
</dependency>  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>druid</artifactId>  
    <version>0.2.23</version>  
</dependency> 

```  
本文将使用mysql数据库及druid连接池； 
 
2、到数据库shiro下建三张表：` users（用户名/密码）、user_roles（用户/角色）、roles_permissions（角色/权限） `，具体请参照shiro-example-chapter2/sql/shiro.sql；并添加一个用户记录，用户名/密码为zhang/123；
 
**3、ini配置（shiro-jdbc-realm.ini）** 
```

jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm  
dataSource=com.alibaba.druid.pool.DruidDataSource  
dataSource.driverClassName=com.mysql.jdbc.Driver  
dataSource.url=jdbc:mysql://localhost:3306/shiro  
dataSource.username=root  
#dataSource.password=  
jdbcRealm.dataSource=$dataSource  
securityManager.realms=$jdbcRealm 

```
1、变量名=全限定类名会自动创建一个类实例
2、变量名.属性=值 自动调用相应的setter方法进行赋值
3、$变量名 引用之前的一个对象实例 
4、测试代码请参照com.github.zhangkaitao.shiro.chapter2.LoginLogoutTest的testJDBCRealm方法，和之前的没什么区别。

##  Authenticator及AuthenticationStrategy 
Authenticator的职责是验证用户帐号，是Shiro API中身份验证核心的入口点： 
```

public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)  
            throws AuthenticationException;  

``` 
如果验证成功，将返回AuthenticationInfo验证信息；此信息中包含了身份及凭证；如果验证失败将抛出相应的AuthenticationException实现。
 
` SecurityManager接口继承了Authenticator `，另外还有一个` ModularRealmAuthenticator实现 `，其委托给多个Realm进行验证，验证规则通过` AuthenticationStrategy `接口指定，默认提供的实现：
  * FirstSuccessfulStrategy：只要有一个Realm验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他的忽略；
  * AtLeastOneSuccessfulStrategy：只要有一个Realm验证成功即可，和FirstSuccessfulStrategy不同，返回所有Realm身份验证成功的认证信息；
  * AllSuccessfulStrategy：所有Realm验证成功才算成功，且返回所有Realm身份验证成功的认证信息，如果有一个失败就失败了。
 
**ModularRealmAuthenticator默认使用AtLeastOneSuccessfulStrategy策略。**
假设我们有三个realm：
myRealm1： 用户名/密码为zhang/123时成功，且返回身份/凭据为zhang/123；
myRealm2： 用户名/密码为wang/123时成功，且返回身份/凭据为wang/123；
myRealm3： 用户名/密码为zhang/123时成功，且返回身份/凭据为zhang@163.com/123，**和myRealm1不同的是返回时的身份变了；**
 
1、ini配置文件(shiro-authenticator-all-success.ini) 
```

#指定securityManager的authenticator实现  
authenticator=org.apache.shiro.authc.pam.ModularRealmAuthenticator  
securityManager.authenticator=$authenticator  
  
#指定securityManager.authenticator的authenticationStrategy  
allSuccessfulStrategy=org.apache.shiro.authc.pam.AllSuccessfulStrategy  
securityManager.authenticator.authenticationStrategy=$allSuccessfulStrategy 

myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1  
myRealm2=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm2  
myRealm3=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm3  
securityManager.realms=$myRealm1,$myRealm3  

``` 

2、测试代码（com.github.zhangkaitao.shiro.chapter2.AuthenticatorTest）
2.1、首先通用化登录逻辑 
```

private void login(String configFile) {  
    //1、获取SecurityManager工厂，此处使用Ini配置文件初始化SecurityManager  
    Factory<org.apache.shiro.mgt.SecurityManager> factory =  
            new IniSecurityManagerFactory(configFile);  
  
    //2、得到SecurityManager实例 并绑定给SecurityUtils  
    org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();  
    SecurityUtils.setSecurityManager(securityManager);  
  
    //3、得到Subject及创建用户名/密码身份验证Token（即用户身份/凭证）  
    Subject subject = SecurityUtils.getSubject();  
    UsernamePasswordToken token = new UsernamePasswordToken("zhang", "123");  
  
    subject.login(token);  
} 

``` 
 
2.2、测试AllSuccessfulStrategy成功：    
```

@Test  
public void testAllSuccessfulStrategyWithSuccess() {  
    login("classpath:shiro-authenticator-all-success.ini");  
    Subject subject = SecurityUtils.getSubject();  
  
    //得到一个身份集合，其包含了Realm验证成功的身份信息  
    PrincipalCollection principalCollection = subject.getPrincipals();  
    Assert.assertEquals(2, principalCollection.asList().size());  
}

```   
即PrincipalCollection包含了zhang和zhang@163.com身份信息。
 
2.3、测试AllSuccessfulStrategy失败：
```

    @Test(expected = UnknownAccountException.class)  
    public void testAllSuccessfulStrategyWithFail() {  
        login("classpath:shiro-authenticator-all-fail.ini");  
        Subject subject = SecurityUtils.getSubject();  
}

```   
shiro-authenticator-all-fail.ini与shiro-authenticator-all-success.ini不同的配置是使用了securityManager.realms=$myRealm1,$myRealm2；即myRealm验证失败。
 
对于AtLeastOneSuccessfulStrategy和FirstSuccessfulStrategy的区别，请参照testAtLeastOneSuccessfulStrategyWithSuccess和testFirstOneSuccessfulStrategyWithSuccess测试方法。唯一不同点一个是返回所有验证成功的Realm的认证信息；另一个是只返回第一个验证成功的Realm的认证信息。
 
**自定义AuthenticationStrategy实现**，首先看其API：
```

//在所有Realm验证之前调用  
AuthenticationInfo beforeAllAttempts(  
Collection<? extends Realm> realms, AuthenticationToken token)   
throws AuthenticationException;  
//在每个Realm之前调用  
AuthenticationInfo beforeAttempt(  
Realm realm, AuthenticationToken token, AuthenticationInfo aggregate)   
throws AuthenticationException;  
//在每个Realm之后调用  
AuthenticationInfo afterAttempt(  
Realm realm, AuthenticationToken token,   
AuthenticationInfo singleRealmInfo, AuthenticationInfo aggregateInfo, Throwable t)  
throws AuthenticationException;  
//在所有Realm之后调用  
AuthenticationInfo afterAllAttempts(  
AuthenticationToken token, AuthenticationInfo aggregate)   
throws AuthenticationException;

```   
因为每个AuthenticationStrategy实例都是无状态的，所有每次都通过接口将相应的认证信息传入下一次流程；通过如上接口可以进行如合并/返回第一个验证成功的认证信息。
 
**自定义实现时一般继承org.apache.shiro.authc.pam.AbstractAuthenticationStrategy即可**，具体可以参考代码com.github.zhangkaitao.shiro.chapter2.authenticator.strategy包下OnlyOneAuthenticatorStrategy 和AtLeastTwoAuthenticatorStrategy。

到此基本的身份验证就搞定了，对于AuthenticationToken 、AuthenticationInfo和Realm的详细使用后续章节再陆续介绍。


其余参考：http://blog.csdn.net/lishehe/article/details/45219023
http://meri-stuff.blogspot.com/2011/04/apache-shiro-part-2-realms-database-and.html
[源码分析shiro认证授权流程](http://www.cnblogs.com/davidwang456/p/4428421.html)