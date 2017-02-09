title: spring-jms 

#  spring-jms 
一般的RPC、RMI等机制采用的都是同步通信机制。
**JMS**（Java Message Service）是面向**异步消息**而制定的标准API
利用Spring的` JmsTemplate `可以简化JMS异步消息的发送，和消息的异步接收。
通过**ActiveMQ**可以搭建一个强大的消息代理服务器，和一套消息代理API实现。
![](/data/dokuwiki/spring/pasted/20150811-044742.png)
在JMS中有两个主要的概念:**消息代理(message broker)和目的地(destination)。**
消息代理一般采用的是ActiveMQ这一实现，目的地一般指实现中的**队列或主题**，分别对应**点对点消息模型和发布-订阅消息模型**。
点对点消息模型，一个消息只能被一个接受者接收，而发布-订阅模型一个消息可以被多个接受者处理。

![](/data/dokuwiki/spring/pasted/20150811-045138.png)![](/data/dokuwiki/spring/pasted/20150811-045146.png)

传统同步通信的缺点：
![](/data/dokuwiki/spring/pasted/20150811-045215.png)
**JMS的优点**：
异步无需等待；
面向消息和解耦，JMS发送消息是以数据为中心的，没有与特定方法绑定。
位置独立：通过消息代理服务集群实现。
确保投递：通过ActiveMQ消息代理实现。
##  在Spring中搭建消息代理 
安装ActiveMQ:http://activemq.apache.org/
###  配置maven依赖： 
```
 
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-context</artifactId>  
            <version>${spring-version}</version>  
        </dependency>  
        <dependency>  
            <groupId>org.springframework</groupId>  
            <artifactId>spring-jms</artifactId>  
            <version>${spring-version}</version>  
        </dependency>   
        <dependency>  
            <groupId>javax.annotation</groupId>  
            <artifactId>jsr250-api</artifactId>  
            <version>1.0</version>  
        </dependency>  
        <dependency>  
            <groupId>org.apache.activemq</groupId>  
            <artifactId>activemq-core</artifactId>  
            <version>5.7.0</version>  
        </dependency>  

```
###  创建连接工厂： 

方式一：使用activemq的spring命名空间：
 xmlns:amq="http://activemq.apache.org/schema/core"
  <amq:connectionFactory id="connectionFactory" 这个名称是有意义的
      brokerURL="tcp://localhost:61616"/>

方式二：使用声明bean的形：
```

<bean id="connectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory">
	<property name="brokerURL" value="tcp://localhost:61616"/>
</bean>

```

参考普通形式：
```

<!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供-->  
<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">  
    <property name="brokerURL" value="tcp://localhost:61616"/>  
</bean>  
  
<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->  
<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">  
    <!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->  
    <property name="targetConnectionFactory" ref="targetConnectionFactory"/>  
</bean>  

```
 ActiveMQ为我们提供了一个` PooledConnectionFactory `，通过往里面注入一个ActiveMQConnectionFactory可以用来将Connection、Session和MessageProducer**池化**，这样可以大大的减少我们的资源消耗。当使用PooledConnectionFactory时，我们在定义一个ConnectionFactory时应该是如下定义：
```

<!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供-->  
<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">  
    <property name="brokerURL" value="tcp://localhost:61616"/>  
</bean>  
  
<bean id="pooledConnectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory">  
    <property name="connectionFactory" ref="targetConnectionFactory"/>  
    <property name="maxConnections" value="10"/>  
</bean>  
  
<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">  
    <property name="targetConnectionFactory" ref="pooledConnectionFactory"/>  
</bean>  

```

###  声明ActiveMQ消息目的地 

目的地可以是一个队列或者一个主题。
方式一：使用activemq的spring命名空间:
 xmlns:amq="http://activemq.apache.org/schema/core"
<amq:queue id="queue" physicalName="spitter.queue" />
<amq:topic id="queue" physicalName="spitter.topic" />
方式二：使用声明bean的形式：
```

<bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
	<constructor-arg value="spitter.queue"/>  
</bean>
<bean id="topic" class="org.apache.activemq.command.ActiveMQTopic">
	<constructor-arg value="spitter.topic"/>  
</bean>

```
##  使用Spring的JMS模板 
JMS API太过繁琐，JmsTemplate封装了模板化代码。简化开发。
###  装配JmsTemplate 
```

 <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->  
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
        <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->  
        <property name="connectionFactory" ref="connectionFactory"/>  
    </bean> 

``` 
###  发送异步消息 
面向接口编程，声明一个接口：用于组装和发送消息
```

public interface AlertService {
  void sendSpittleAlert(Spittle spittle);
}

```
接口实现类中发送消息：
```

public class AlertServiceImpl implements AlertService {
  public void sendSpittleAlert(final Spittle spittle) {
    jmsTemplate.send(
      "spittle.queue", //指定目的地
      new MessageCreator() {      
        public Message createMessage(Session session)
                throws JMSException {
          return session.createObjectMessage(spittle); //创建消息。
        }
      }
    );
  }
  
  @Autowired
  JmsTemplate jmsTemplate;  //注入jms模板
}

```
###  为JmsTemplate配置默认目的地： 
```

 <!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->  
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
        <!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->  
        <property name="connectionFactory" ref="connectionFactory"/>  
       <property name="defaultDestinationName" value="spitter.queue"/>  
    </bean> 

```
这样调用jmsTemplate的send()方法时就可以不用指定目的地了。
###  同步方式接收消息 
只需要调用jmsTemplate的` receive() `方法即可：
```

@Component
public class AlertMessageReceiver {
  @Autowired
  JmsTemplate jmsTemplate;
  public Spittle getAlert() {
    try {
      ObjectMessage receivedMessage = 
          (ObjectMessage) jmsTemplate.receive(); //注意此处为同步接收，如果没有可用消息，receive()则会一直等待，直到获得消息或者超时为止。
      
      return (Spittle) receivedMessage.getObject();//<co id="co_getObject"/>
    } catch (JMSException jmsException) {
      throw JmsUtils.convertJmsAccessException(jmsException);//<co id="co_throwException"/>
    }
  }
}

```
jmsTemplate.receive(); / /注意此处为**同步**接收，如果没有可用消息，**receive()则会一直等待**，直到获得消息或者超时为止。
为什么消息可以异步发送而不能异步接收呢。？这很奇怪，不过幸运的是，Spring提供了类似于EJB3当中的MDB消息驱动bean的解决方案MDP即消息驱动POJO。
##  创建消息驱动的POJO 
用于**异步**接收消息：
**写POJO用于异步接收处理消息：**
```

public class SpittleAlertHandler {
  
  public void processSpittle(Spittle spittle) {
    // ... implementation goes here...
  }
}

```
**配置消息监听器**
首先将POJO声明为一个bean：
<bean id="spittleHandler" class="com.css.SpittleAlertHandler"/>
然后，为了把这个POJO转化为消息驱动的POJO，需要把bean声明为消息监听器：
 xmlns:jms="http://www.springframework.org/schema/jms"
```

  <jms:listener-container connection-factory="connectionFactory">
    <jms:listener destination="spitter.queue"
         ref="spittleHandler" method="processSpittle" /> 同时指定处理消息的方法，方法参数会被注入值
  </jms:listener-container>  

```
![](/data/dokuwiki/spring/pasted/20150811-053315.png)
##  使用基于消息的RPC 
略。。。

更多参考：
http://www.iteye.com/blogs/subjects/springjms