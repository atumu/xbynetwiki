createAt:2017-02-08 16:12:34
author:xbynet
modifyAt:2017-02-08 16:35:18
location:dokuwiki/javaee/WebService学习之CXF系列1入门
title:WebService学习之CXF系列1入门

 CXF官方网址：http://cxf.apache.org/
 # CXF 入门示例
 参考：http://www.cnblogs.com/hoojo/archive/2011/03/29/1998909.html
 1、 HelloWorldService服务器端代码
```
import javax.jws.WebParam;
import javax.jws.WebService;
import javax.jws.soap.SOAPBinding;
import javax.jws.soap.SOAPBinding.Style;
 
/**
 * <b>function:</b>CXF WebService 服务器端helloWorld示例
 */
@WebService
@SOAPBinding(style = Style.RPC)
public class HelloWorldService {
    
    public String sayHello(@WebParam(name = "name") String name) {
        return name + " say: Hello World ";
    }
}
```
发布HelloWorldService，代码如下：
```
import javax.xml.ws.Endpoint;
import com.hoo.service.HelloWorldService;
/**
 * <b>function:</b> 发布CXF WebService
 */
public class DeployHelloWorldService {
	
	/**
	 * <b>function:</b>发布WebService
	 */
	public static void deployService() {
		System.out.println("Server start ……");
		HelloWorldService service = new HelloWorldService();
		String address = "http://localhost:9000/helloWorld";
		Endpoint.publish(address, service);
	}
	
	public static void main(String[] args) throws InterruptedException {
		//发布WebService
		deployService();
		System.out.println("server ready ……");
		Thread.sleep(1000 * 60);
		System.out.println("server exiting");
		//休眠60秒后就退出
		System.exit(0);
	}
}
```
那么你在WebBrowser中请求：
http://localhost:9000/helloWorld?wsdl就可以看到xml内容了。

**定制客户端调用WebService的接口，这个接口中的方法签名和参数信息可以从wsdl中的内容看到**，代码如下：
```
import javax.jws.WebParam;
import javax.jws.WebService;
 
/**
 * <b>function:</b> 客户端调用WebService所需要的接口
 */
@WebService
public interface IHelloWorldService {
    public String sayHello(@WebParam(name = "name") String name);
}
```
编写客户端调用WebService代码
```
mport org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import com.hoo.service.IHelloWorldService;
/**
 * <b>function:</b>CXF WebService客户端调用代码
 */
public class HelloWorldServiceClient {
  
    public static void main(String[] args) {
        //调用WebService
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();
        factory.setServiceClass(IHelloWorldService.class);
        factory.setAddress("http://localhost:9000/helloWorld");
        
        IHelloWorldService service = (IHelloWorldService) factory.create();
        System.out.println("[result]" + service.sayHello("hoojo"));
    }
}
```

# CXF对Interceptor拦截器的支持
http://www.cnblogs.com/hoojo/archive/2011/03/30/1999499.html
我们就用上面的HelloWorldService，客户端的调用代码重新写一份，代码如下：
```
import org.apache.cxf.interceptor.LoggingInInterceptor;
import org.apache.cxf.interceptor.LoggingOutInterceptor;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.apache.cxf.phase.Phase;
import com.hoo.interceptor.MessageInterceptor;
import com.hoo.service.IHelloWorldService;
 
/**
 * <b>function:</b>CXF WebService客户端调用代码
 */
public class ServiceMessageInterceperClient {
    
    public static void main(String[] args) {
        //调用WebService
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();
        factory.setServiceClass(IHelloWorldService.class);
        factory.setAddress("http://localhost:9000/helloWorld");
        factory.getInInterceptors().add(new LoggingInInterceptor());
        factory.getOutInterceptors().add(new LoggingOutInterceptor());
        
        IHelloWorldService service = (IHelloWorldService) factory.create();
        System.out.println("[result]" + service.sayHello("hoojo"));
    }
}
```
上面的CXF的拦截器是添加在客户端，同样在服务器端也是可以添加拦截器Interceptor的。

刚才是客户端添加Interceptor，现在我们自己编写一个Interceptor，这个Interceptor需要继承AbstractPhaseInterceptor，实现handleMessage和一个带参数的构造函数。然后在服务器端添加这个Interceptor。
```
package com.hoo.interceptor;
 
import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.message.Message;
import org.apache.cxf.phase.AbstractPhaseInterceptor;
 
/**
 * <b>function:</b> 自定义消息拦截器
 */
public class MessageInterceptor extends AbstractPhaseInterceptor<Message> {
    
    //至少要一个带参的构造函数
    public MessageInterceptor(String phase) {
        super(phase);
    }
 
    public void handleMessage(Message message) throws Fault {
        System.out.println("############handleMessage##########");
        System.out.println(message);
        if (message.getDestination() != null) {
            System.out.println(message.getId() + "#" + message.getDestination().getMessageObserver());
        }
        if (message.getExchange() != null) {
            System.out.println(message.getExchange().getInMessage() + "#" + message.getExchange().getInFaultMessage());
            System.out.println(message.getExchange().getOutMessage() + "#" + message.getExchange().getOutFaultMessage());
        }
    }
}
```
下面看看发布服务和添加自定义拦截器的代码：
```
package com.hoo.service.deploy;
 
import org.apache.cxf.jaxws.JaxWsServerFactoryBean;
import org.apache.cxf.phase.Phase;
import com.hoo.interceptor.MessageInterceptor;
import com.hoo.service.HelloWorldService;
 
/**
 * <b>function:</b>在服务器发布自定义的Interceptor
 */
public class DeployInterceptorService {
 
    public static void main(String[] args) throws InterruptedException {
        //发布WebService
        JaxWsServerFactoryBean factory = new JaxWsServerFactoryBean();
        //设置Service Class
        factory.setServiceClass(HelloWorldService.class);
        factory.setAddress("http://localhost:9000/helloWorld");
        //设置ServiceBean对象
         factory.setServiceBean(new HelloWorldService());
        
        //添加请求和响应的拦截器，Phase.RECEIVE只对In有效，Phase.SEND只对Out有效
         factory.getInInterceptors().add(new MessageInterceptor(Phase.RECEIVE));
        factory.getOutInterceptors().add(new MessageInterceptor(Phase.SEND));
        
        factory.create();
        
        System.out.println("Server start ......");
        Thread.sleep(1000 * 60);
        System.exit(0);
        System.out.println("Server exit ");
    }
}
```
**值得说的是，以前发布WebService是用Endpoint的push方法。这里用的是JaxWsServerFactoryBean和客户端调用的代码JaxWsProxyFactoryBean有点不同。**

# CXF WebService中传递复杂类型对象
前面介绍的都是传递简单的字符串，现在开始介绍传递复杂类型的对象。如JavaBean、Array、List、Map等。
1、 首先看看服务器端的代码所需要的JavaBean对象
```
public class User implements Serializable {
    private static final long serialVersionUID = 677484458789332877L;
    private int id;
    private String name;
    private String email;
    private String address;
    
    //getter、setter
    
    @Override
    public String toString() {
        return this.id + "#" + this.name + "#" + this.email + "#" + this.address;
    }
}
```
下面的是集合传递Users，CXF直接传递集合对象会出现异常，用一个对象包装下就Ok了，不知道是什么原因。
```
public class Users {
    private List<User> users;
    private User[] userArr;
    private HashMap<String, User> maps;
        
    //getter、setter方法
}
```
2、 下面看看复杂对象传递的服务器端代码
```
import javax.jws.WebParam;
import javax.jws.WebService;
import javax.jws.soap.SOAPBinding;
import javax.jws.soap.SOAPBinding.Style;

@WebService
@SOAPBinding(style = Style.RPC)
@SuppressWarnings("deprecation")
public class ComplexUserService {
    
    public User getUserByName(@WebParam(name = "name") String name) {
        User user = new User();
        user.setId(new Date().getSeconds());
        user.setName(name);
        user.setAddress("china");
        user.setEmail(name + "@hoo.com");
        return user;
    }
    
    public void setUser(User user) {
        System.out.println("############Server setUser###########");
        System.out.println("setUser:" + user);
    }
    
    public Users getUsers(int i) {
        List<User> users = new ArrayList<User>();
        for (int j = 0; j <= i; j++) {
            User user = new User();
            user.setId(new Date().getSeconds());
            user.setName("jack#" + j);
            user.setAddress("china");
            user.setEmail("jack" + j + "@hoo.com");
            users.add(user);
        }
        Users u = new Users();
        u.setUsers(users);
        return u;
    }
    
    public void setUsers(Users users) {
        System.out.println("############Server setUsers###########");
        for (User u : users.getUsers()) {
            System.out.println("setUsers:" + u);
        }
    }
    
    public Users getUserArray(int i) {
        User[] users = new User[i];
        for (int j = 0; j < i; j++) {
            User user = new User();
            user.setId(new Date().getSeconds());
            user.setName("jack#" + j);
            user.setAddress("china");
            user.setEmail("jack" + j + "@hoo.com");
            users[j] = user;
        }
        Users u = new Users();
        u.setUserArr(users);
        return u;
    }
    
    public void setUserArray(Users users) {
        System.out.println("############Server setUserArray###########");
        for (User u : users.getUserArr()) {
            System.out.println("setUserArray:" + u);
        }
    }
    
    public void setUserMap(Users maps) {
        System.out.println("############Server setUserMap###########");
        System.out.println("setUserMap:" + maps.getMaps());
    }
    
    public Users getUserMap() {
        HashMap<String, User> users = new HashMap<String, User>();
        User user = new User();
        user.setId(new Date().getSeconds());
        user.setName("jack#");
        user.setAddress("china#");
        user.setEmail("jack@hoo.com");
        users.put("A", user);
        
        user = new User();
        user.setId(new Date().getSeconds());
        user.setName("tom");
        user.setAddress("china$$");
        user.setEmail("tom@hoo.com");
        users.put("B", user);
        Users u = new Users();
        u.setMaps(users);
        
        return u;
    }
}
```
3、 发布WebService的代码
```
public class DeployComplexUserService {
 
    public static void main(String[] args) throws InterruptedException {
        String address = "http://localhost:9000/complexUser";
        DeployUtils.deployService(address, new ComplexUserService());
        Thread.sleep(1000 * 60 * 20);
        System.exit(0);
        System.out.println("Server exit ");
    }
}
```
DeployUtils.java
```
public final class DeployUtils {
    public static void deployService(String address, Object service) {
        System.out.println("Server start ……");
        Endpoint.publish(address, service);
    }
}
```
4、 通过发布后的地址的http://localhost:9000/complexUser?wsdl; wsdl中的内容定制你的客户端调用WebService的接口，当然你也可以让服务器端实现一个接口。
5、 客户端调用代码，和以前没什么变化。
```
public class ComplexUserServiceClient {
 
    public static void main(String[] args) {
        //调用WebService
        JaxWsProxyFactoryBean factory = new JaxWsProxyFactoryBean();
        factory.setServiceClass(IComplexUserService.class);
        factory.setAddress("http://localhost:9000/complexUser");
        
        IComplexUserService service = (IComplexUserService) factory.create();
        
        System.out.println("#############Client getUserByName##############");
        User user = service.getUserByName("hoojo");
        System.out.println(user);
        
        user.setAddress("China-Guangzhou");
        service.setUser(user);
        
        System.out.println("#############Client getUsers##############");
        Users users = service.getUsers(4);
        System.out.println(users);
        List<User> tempUsers = new ArrayList<User>();
        for (User u : users.getUsers()) {
            System.out.println(u);
            u.setName("hoojo" + new Random().nextInt(100));
            u.setAddress("Chian-GuangZhou#" + new Random().nextInt(100));
            tempUsers.add(u);
        }
        users.getUsers().clear();
        users.getUsers().addAll(tempUsers);
        service.setUsers(users);
        
        System.out.println("#############Client getUserArray##############");
        users = service.getUserArray(4);
        User[] userArr = new User[4];
        int i = 0;
        for (User u : users.getUserArr()) {
            System.out.println(u);
            u.setName("hoojo" + new Random().nextInt(100));
            u.setAddress("Chian-ShenZhen#" + new Random().nextInt(100));
            userArr[i] = u;
            i ++;
        }
        
        users.setUserArr(userArr);
        service.setUserArray(users);
        
        System.out.println("##################Client getUserMap###############");
        users = service.getUserMap();
        System.out.println(users.getMaps());
        users.getMaps().put("ABA", userArr[0]);
        service.setUserMap(users);
    }
}
```
# CXF WebService整合Spring
http://www.cnblogs.com/hoojo/archive/2011/03/30/1999563.html
http://www.cnblogs.com/hoojo/archive/2012/07/13/2590593.html
http://cxf.apache.org/docs/writing-a-service-with-spring.html
首先在web.xml中添加如下配置：
```
<servlet>
    <servlet-name>CXFService</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
</servlet>
 
<servlet-mapping>
    <servlet-name>CXFService</servlet-name>
    <url-pattern>/*</url-pattern>
</servlet-mapping>
```
新建一个applicationContext-server.xml文件，文件内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jaxws="http://cxf.apache.org/jaxws"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans >
    http://cxf.apache.org/jaxws 
    http://cxf.apache.org/schemas/jaxws.xsd"
```
添加完这个文件后，还需要在这个文件中导入这么几个文件。文件内容如下：
```
<import resource="classpath:META-INF/cxf/cxf.xml"/>
<import resource="classpath:META-INF/cxf/cxf-extension-soap.xml"/>
<import resource="classpath:META-INF/cxf/cxf-servlet.xml"/>
```
下面开始写服务器端代码，首先定制服务器端的接口，代码如下：
```
@WebService
public interface IComplexUserService {
    
    public User getUserByName(@WebParam(name = "name") String name);
    
    public void setUser(User user);
}
```
下面编写WebService的实现类。
```
@WebService(endpointInterface="com.demo.IComplexUserService")
public class ComplexUserService implements IComplexUserService {
    
    public User getUserByName(@WebParam(name = "name") String name) {
        User user = new User();
        user.setId(new Date().getSeconds());
        user.setName(name);
        user.setAddress("china");
        user.setEmail(name + "@hoo.com");
        return user;
    }
    
    public void setUser(User user) {
        System.out.println("############Server setUser###########");
        System.out.println("setUser:" + user);
    }
}
```
注意的是和Spring集成，这里一定要完成接口实现，如果没有接口的话会有错误的。
下面要在applicationContext-server.xml文件中添加如下配置：
```
<bean id="userServiceBean" class="com.hoo.service.ComplexUserService"/>
 
<bean id="inMessageInterceptor" class="com.hoo.interceptor.MessageInterceptor">
    <constructor-arg  value="receive"/>
</bean>
 
<bean id="outLoggingInterceptor" class="org.apache.cxf.interceptor.LoggingOutInterceptor"/>
<!-- 注意下面的address，这里的address的名称就是访问的WebService的name；#userServiceBean是直接引用Ioc容器中的Bean对象 -->
<jaxws:server id="userService" serviceBean="#userServiceBean" address="/Users">
    <jaxws:inInterceptors>
        <ref bean="inMessageInterceptor"/>
    </jaxws:inInterceptors>
    <jaxws:outInterceptors>
        <ref bean="outLoggingInterceptor"/>
    </jaxws:outInterceptors>
</jaxws:server>
<!-- 或者这种方式，在老版本中这个是不能引用Ioc容器中的对象，但在2.x中可以直接用#id或#name的方式发布服务 -->
<jaxws:endpoint id="userService2" implementor="#userServiceBean" address="/Users">
    <jaxws:inInterceptors>
        <ref bean="inMessageInterceptor"/>
    </jaxws:inInterceptors>
    <jaxws:outInterceptors>
        <ref bean="outLoggingInterceptor"/>
    </jaxws:outInterceptors>
</jaxws:endpoint>
```
下面启动tomcat服务器后，在WebBrowser中请求：
http://localhost:8080/CXFWebService/Users?wsdl
如果你能看到wsdl的xml文件的内容，就说明你成功了，注意的是上面地址的Users就是上面xml配置中的address的名称，是一一对应的。

客户端请求的代码照前面所说的写就可以了。

这个server端是通过Spring整合配置的，**下面我们将Client端也通过Spring配置完成整合。**
首先增加applicationContext-client.xml配置文件
```
<jaxws:client id="userWsClient" serviceClass="com.hoo.service.IComplexUserService" 
        address="http://localhost:8080/CXFWebService/Users"/>
```
客户端请求代码如下：
```
public class SpringUsersWsClient {
 
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext-client.xml");
        IComplexUserService service = ctx.getBean("userWsClient", IComplexUserService.class);
        
        System.out.println("#############Client getUserByName##############");
        User user = service.getUserByName("hoojo");
        System.out.println(user);
        
        user.setAddress("China-Guangzhou");
        service.setUser(user);
    }
}
```
