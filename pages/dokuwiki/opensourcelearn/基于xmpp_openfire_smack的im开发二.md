title: 基于xmpp_openfire_smack的im开发二 

#  基于xmpp_openfire_smack的im开发二 

#  Android客户端开发 
原文：http://blog.csdn.net/shimiso/article/details/11225873
##  源码结构介绍 
![](/data/dokuwiki/android/pasted/20150520-065634.png)
activity包下存放一些android页面交互相关的控制程序，还有一个些公共帮助类
db包为sqlite的工具类封装，这里做了一些自定义的改造，稍微仿Spring的JdbcTemplate结构，使用起来更加方便一点
manager包留下主要是一些管理组件，包括联系人管理，消息管理，提醒管理，离线消息管理，用户管理，xmpp连接管理
model包中都是一些对象模型，传输介质
service中存放一些android后台的核心服务，主要包括聊天服务，联系人服务，系统消息服务，重连接服务
task包中存放一些耗时的异步操作
util中存放一些常用的工具类
view中一些和android的UI相关的显示控件

其中strings.xml中，保存的缺省配置为gtalk的服务器信息，大家如果有谷歌gtalk的账号可以直接登录，否则需要更改这里的配置才可以使用其他的xmpp服务器
```

<!-- 缺省的服务器配置 -->   
  <integer name="xmpp_port">5222</integer>   
  <string name="xmpp_host">talk.google.com</string>   
  <string name="xmpp_service_name">gmail.com</string>  
  <bool name="is_remember">true</bool>  
  <bool name="is_autologin">false</bool>  
  <bool name="is_novisible">false</bool>  

```
##  权限： 

```

   <!-- 访问Internet -->  
<uses-permission android:name="android.permission.INTERNET" />  
<!--- 访问网络状态 -->  
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />  
    <!-- 往SDCard写入数据权限 -->  
 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>  
<!-- 在SDCard中创建与删除文件权限 -->  
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>  
<!-- 往SDCard写入数据权限 -->  
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/> 

```
##  核心类介绍 
1.ActivitySupport类
大家写android程序会发现，不同的activity之间经常需要调用一些公共的资源，这里的资源不仅包括android自身的，还有我们自己的管理服务类，甚至相互之间传递一些参数，这里我仿照struts2的设计，提炼出一个ActivitySupport类，同时抽取一个接口，让所有的Activity都集成这个类，因为有了接口，我们便可以采用回调模式，非常方便的传递数据和使用公共的资源，这种好处相信大家使用之后都能有深刻的体会，通过接口回调传递参数和相互调用的方式无疑是最优雅的，spring和hibernate源码中曾经大量使用这种结构。
2.SQLiteTemplate类
我们希望在android操作数据库是优雅的一种方式，这里不必关注事务，也不用担心分页，更不用为了封装传递对象烦恼，总之一切就像面向对象那样，简单，模板类的出现正是解决这个问题，虽然它看上去可能不是那么完美有待提高，这里我封装了很多sqlite常用的工具，大家可以借鉴使用。
3.XmppConnectionManager管理类
这个类是xmpp连接的管理类，如果大家使用smack的api对这个应该不会陌生，asmack对xmpp连接的管理，与smack的差别不大，但是部分细微区别也有，我们在使用中如果遇到问题，还要多加注意，我们这里将其设计成单例，毕竟重复创建连接是个非常消耗的过程。
##  3.演示效果 
![](/data/dokuwiki/android/pasted/20150520-070005.png)![](/data/dokuwiki/android/pasted/20150520-070012.png)![](/data/dokuwiki/android/pasted/20150520-070016.png)
很像QQ吧，没错，这是2012年版本qq的安卓界面，只是界面元素一样，实现方式大不相同，下面简单列一下这个客户端实现的功能：
1.聊天
2.离线消息
3.添加，删除好友
4.添加，移动好友分组
5.设置昵称
6.监控好友状态
7.网络断开系统自动重连接
8.收到添加好友请求消息处理
9.收到系统广播消息处理
10.查看历史聊天记录
11.消息弹出提醒，和小气泡
....

#  Android消息推送技术原理分析和实践 

参考：http://blog.csdn.net/shimiso/article/details/8156439
openfire过于庞大繁复，许多对我们来说都是没什么用的，甚至要砍掉改造，能不能有精简的xmpp服务器呢?答案是有的,androidpn,笔者认真比对过openfire和androidpn的源码,最后惊奇的发现,原来它就是从openfire里面庖丁解牛出来的一部分,做这件事的人非常的了不起,为我们省了很大力气,在此感谢他的开源和共享精神,那么androidpn分离出来的是消息推送服务,简言之就是从服务端向android客户端推送消息的服务,因为openfire的源码架构是在jetty基础上建立的,它的启动和部署方式和我们传统的服务器tomcat和weblogic等有点区别,所以androidpn也有jetty的影子,在和我们传统架构组合的时候还要再把它和jetty拆开, androidpn的搭建和使用网上的教程很多,大家可以发现大部分千篇一律,出现一个OK界面就没了,堂而皇之的写上原创,有的只是改了下hello world,如此糊弄,实在难为所用!
![](/data/dokuwiki/android/pasted/20150520-070821.png)
androidpn消息推送采用的是apache的mina框架做的,服务端和客户端两边都有监听,也就是我们所说的socket编程,有人说socket编程有什么难的,就那么回事,其实不然,我们平时写的socket聊天都只是在局域网的,但是要穿透路由和防火墙,让信息安全及时的传送到另一个网关的局域网电脑中,就不是一件简单的活了,其中涉及到在nat上打洞,还有线程,断网重连,安全加密等等,那么androidpn配合mina相当于把这些活都干了,那么我们要的干活就相对比较精细了,第一学习mina的安装配置的规则,第二学习xmpp协议组装和解析的规则,第三学习androidpn推和收消息的核心代码,如此三点我们便能灵活驾驭住androidpn出现再大的问题自己也能动手去调了。
在和spring整合的时候大家要注意不要让mina服务启动2次，笔者整合时候无意发现在linux64位系统，weblogic上启动时候总是报5222已经被占用，反复查看代码发现mina在随web容器启动过一次5222端口后，xmppserver类中的start方法中ClassPathXmlApplicationContext类又加载了一次spring配置，导致端口被重复开启两次，最后终于发现问题所在：
```

<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"  
    xmlns:util="http://www.springframework.org/schema/util"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd  
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd  
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-2.5.xsd">  
    <context:component-scan base-package="org.androidpn.server.*" /><!-- 自动装配 -->    
  
    <!-- # # # # # === -->  
    <!-- Resources                                                       -->  
    <!-- # # # # # === -->  
    <bean id="propertyConfigurer"  
        class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">  
        <property name="locations">  
            <list>  
                <value>classpath:jdbc.properties</value>  
            </list>  
        </property>  
    </bean>  
  
    <!-- # # # # # === -->  
    <!-- Data Source                                                     -->  
    <!-- # # # # # === -->  
  
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  
        destroy-method="close">  
        <property name="driverClassName" value="${jdbcDriverClassName}" />  
        <property name="url" value="${jdbcUrl}" />  
        <property name="username" value="${jdbcUsername}" />  
        <property name="password" value="${jdbcPassword}" />  
        <property name="maxActive" value="${jdbcMaxActive}" />  
        <property name="maxIdle" value="${jdbcMaxIdle}" />  
        <property name="maxWait" value="${jdbcMaxWait}" />  
        <property name="defaultAutoCommit" value="true" />  
    </bean>   
      
    <!-- sessionFactory -->  
    <bean id="sessionFactory"  
        class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">  
        <property name="dataSource" ref="dataSource" />  
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />  
    </bean>  
  
    <!-- 配置事务管理器 -->  
    <bean id="txManager"  
        class="org.springframework.orm.hibernate3.HibernateTransactionManager">  
        <property name="sessionFactory" ref="sessionFactory" />  
        <property name="dataSource" ref="dataSource" />  
    </bean>  
      
    <!-- 采用注解来管理事务-->  
    <tx:annotation-driven transaction-manager="txManager" />   
      
    <!-- spring hibernate工具类模板 -->  
    <bean id="hibernateTemplate"  
        class="org.springframework.orm.hibernate3.HibernateTemplate">  
        <property name="sessionFactory" ref="sessionFactory"></property>  
    </bean>  
    <!-- spring jdbc 工具类模板 -->  
    <bean id="jdbcTemplate"  
        class="org.springframework.jdbc.core.JdbcTemplate">  
        <property name="dataSource">  
            <ref bean="dataSource" />  
        </property>  
    </bean>      
      
    <!-- # # # # # === -->  
    <!-- SSL                                                             -->  
    <!-- # # # # # === -->  
  
    <!--  
    <bean id="tlsContextFactory"  
        class="org.androidpn.server.ssl2.ResourceBasedTLSContextFactory">  
        <constructor-arg value="classpath:bogus_mina_tls.cert" />  
        <property name="password" value="boguspw" />  
        <property name="trustManagerFactory">  
            <bean class="org.androidpn.server.ssl2.BogusTrustManagerFactory" />  
        </property>  
    </bean>  
    -->  
    <!-- MINA  -->   
    <bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">  
        <property name="customEditors">  
            <map>  
                <entry key="java.net.SocketAddress">  
                    <bean class="org.apache.mina.integration.beans.InetSocketAddressEditor" />  
                </entry>  
            </map>  
        </property>  
    </bean>  
  
    <bean id="xmppHandler" class="org.androidpn.server.xmpp.net.XmppIoHandler" />  
  
    <bean id="filterChainBuilder"  
        class="org.apache.mina.core.filterchain.DefaultIoFilterChainBuilder">  
        <property name="filters">  
            <map>  
                <entry key="executor">  
                    <bean class="org.apache.mina.filter.executor.ExecutorFilter" />  
                </entry>  
                <entry key="codec">  
                    <bean class="org.apache.mina.filter.codec.ProtocolCodecFilter">  
                        <constructor-arg>  
                            <bean class="org.androidpn.server.xmpp.codec.XmppCodecFactory" />  
                        </constructor-arg>  
                    </bean>  
                </entry>  
                <!--  
                <entry key="logging">  
                    <bean class="org.apache.mina.filter.logging.LoggingFilter" />  
                </entry>  
                -->  
            </map>  
        </property>  
    </bean>  
  
    <bean id="ioAcceptor" class="org.apache.mina.transport.socket.nio.NioSocketAcceptor"  
        init-method="bind" destroy-method="unbind" scope="singleton">  
        <property name="defaultLocalAddress" value=":5222" />  
        <property name="handler" ref="xmppHandler" />  
        <property name="filterChainBuilder" ref="filterChainBuilder" />  
        <property name="reuseAddress" value="true" />  
    </bean>  
       
      
    <bean id="serviceLocator" class="org.androidpn.server.service.ServiceLocator" scope="singleton" />   
  
      
    <!-- Services-->   
      
    <bean id="userService" class="org.androidpn.server.service.impl.UserServiceImpl"/>  
      
    <bean id="notificationService" class="org.androidpn.server.service.impl.NotificationServiceImpl"/>  
       
</beans>  

```
配置serviceLocator是为了保证spring容器只能由一个上下文，也就是spring容器只被启动一次，我们将BeanFactory交给了serviceLocator，这样一来有什么好处呢？
控制层，服务层，数据库操作层都受spring管理，在他们中去跟spring要资源，一定是要什么有什么想怎么拿就怎么拿，都很方便，但是如果想在没有被spring所管理的类中去拿spring的资源，动作就不那么优雅了，有人建议用ClassPath加载器初始化spring工厂来获取资源，问题就处在这里，这种做法必定会产生2个spring上下文，一个是web容器所启动的，一个是java类加载器所启动的，我们的MINA服务器也就被启动了2次，其实资源被重复多次实例化除了影响性能外，对程序影响可能并不大，但是MINA被启动2次，肯定会出问题的。为保证spring只有一个上下文，我们将容器上下文交给了serviceLocator，脱离spring管控的环境可以面向serviceLocator来调度spring中的资源操作MINA服务器。
```

package org.androidpn.server.service;  
  
import org.springframework.beans.BeansException;  
import org.springframework.beans.factory.BeanFactory;  
import org.springframework.beans.factory.BeanFactoryAware;  
  
   
public class ServiceLocator implements BeanFactoryAware {  
    private static BeanFactory beanFactory = null;  
  
    private static ServiceLocator servlocator = null;  
  
    public static String USER_SERVICE = "userService";  
  
    public static String NOTIFICATION_SERVICE = "notificationService";  
  
    public void setBeanFactory(BeanFactory factory) throws BeansException {  
    this.beanFactory = factory;  
    }  
  
    public BeanFactory getBeanFactory() {  
    return beanFactory;  
    }  
  
    public static ServiceLocator getInstance() {  
    if (servlocator == null)  
        servlocator = (ServiceLocator) beanFactory.getBean("serviceLocator");  
    return servlocator;  
    }  
  
    /** 
     * 根据提供的bean名称得到相应的服务类 
     *  
     * @param servName 
     *            bean名称 
     */  
    public static Object getService(String servName) {  
    return beanFactory.getBean(servName);  
    }  
  
    /** 
     * 根据提供的bean名称得到对应于指定类型的服务类 
     *  
     * @param servName 
     *            bean名称 
     * @param clazz 
     *            返回的bean类型,若类型不匹配,将抛出异常 
     */  
    public static Object getService(String servName, Class clazz) {  
    return beanFactory.getBean(servName, clazz);  
    }  
  
    /** 
     * Obtains the user service. 
     *  
     * @return the user service 
     */  
    public static UserService getUserService() {  
    return (UserService) getService(USER_SERVICE);  
    }  
  
    public static NotificationService getNotificationService() {  
    return (NotificationService) getService(NOTIFICATION_SERVICE);  
    }  
}  

```
在config.properties中还要特别注意xmpp.resourceName必须跟客户端中XmppManager的private static final String XMPP_RESOURCE_NAME = "AndroidpnClient";保持一致，否则连不上服务器，还xmpp.session.maxInactiveInterval=-1表示永不中断，如果设定了时间超过这个时间范围没有任何活动就会自动断开，这里的时间单位全部是毫秒。
```

apiKey=1234567890  
xmpp.ssl.storeType=JKS  
xmpp.ssl.keystore=conf/security/keystore  
xmpp.ssl.keypass=changeit  
xmpp.ssl.truststore=conf/security/truststore  
xmpp.ssl.trustpass=changeit  
xmpp.resourceName=AndroidpnClient  
  
##Added by ken  
username=admin  
password=admin  
  
#资源名称  
resource_name=AndroidpnClient  
  
#校验超时时间间隔  
xmpp.session.checkTimeoutInterval=10000  
  
#Session timeout最大非活动时间间隔  
xmpp.session.maxInactiveInterval=1000000  

```
在androidpn.properties中端口和IP不要写错，有人喜欢写localhost，在手机上是无法识别的，必须写绝对IP地址。
apiey=1234567890
xmppHost=192.168.1.78
xmppPort=5222
 
运行结果如下：
![](/data/dokuwiki/android/pasted/20150520-071031.png)![](/data/dokuwiki/android/pasted/20150520-071039.png)![](/data/dokuwiki/android/pasted/20150520-071043.png)
离线消息也支持，先给离线用户发个消息，效果如下：
![](/data/dokuwiki/android/pasted/20150520-071126.png)
在数据库中我们看到有一条离线消息是发给用户4aa50dde313f4b63907c2430bf00b413，status为0标记为离线
![](/data/dokuwiki/android/pasted/20150520-071142.png)
这时我们再上线，大约等待20秒左右，查看系统控制台打印：
![](/data/dokuwiki/android/pasted/20150520-071418.png)

查看android端看看用户4aa50dde313f4b63907c2430bf00b413上线情况：
![](/data/dokuwiki/android/pasted/20150520-071228.png)
这时候数据库记录发生了变化，status变成了2，表示已经接收，用户点击OK的时候，它又变成了3表示已经查看
![](/data/dokuwiki/android/pasted/20150520-071437.png)
离线消息的原理相对比较简单，当系统给指定用户发送消息时候，会首先判断用户是够在线，如果在线就直接发送，如果没有在线就暂时标记保存，等用户上线时候先查离线消息然后弹出，其实整个项目都是开源的，可能唯一的难点就是对MINA和XMPP协议的不了解，再加上本身对socket和多线程的畏惧，如果这些全部都掌握，驾驭好这套源码还是很有信心的，了解其基本原理以后，我们就可以放心的做更多的扩展。
 网上现在也有不少androidpn版本，五花八门什么都有，里面到底有没问题，改了什么没改什么都不知道，基本上已经追溯不到原创到底是谁了，索性就只能从国外的一个网站上下了一个比较可靠的版本自己动手去量身改造，终于出了一个比较稳定版本。对于消息提醒来说，它仅仅是个notification，许多人非要把业务数据也做进去，更有夸张好几兆的xml数据就这么硬塞提醒过去，这种做法本身就背离了设计的初衷，非要把跑车当牛车使能不出问题吗？其实业务数据还是用http拉比较好，xmpp及时的前提是用资源消耗作为代价的，我们能适度就适度用，用好用稳就行!

搭建步骤：
1.android端找到res/raw/androidpn.properties文件修改服务器ip地址,不要写localhost,写绝对ip地址
2.服务端找到resources/jdbc.properties 在mysql中新建一个数据库apn,并将连接指向该库,设置用户名和密码,库表会随服务启动的时候自动创建
3.先启动服务,再打开android客户端,点击连接即可

参考：http://blog.csdn.net/shimiso