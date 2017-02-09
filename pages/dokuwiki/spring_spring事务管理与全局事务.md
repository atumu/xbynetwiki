title: spring_spring事务管理与全局事务 

#  Spring事务管理与全局事务 
参考：http://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/

Spring 的事务管理是 Spring 框架中一个比较重要的知识点，**该知识点本身并不复杂，只是由于其比较灵活，导致初学者很难把握。**
本教程从基础知识开始，详细分析了 Spring 事务管理的使用方法，为读者理清思路

##  Spring 事务属性分析 
事务管理对于企业应用而言至关重要。它保证了用户的每一次操作都是可靠的，即便出现了异常的访问情况，也不至于破坏后台数据的完整性。就像银行的自助取款机，通常都能正常为客户服务，但是也难免遇到操作过程中机器突然出故障的情况，此时，事务就必须确保出故障前对账户的操作不生效，就像用户刚才完全没有使用过取款机一样，以保证用户和银行的利益都不受损失。
在 Spring 中，事务是通过`  TransactionDefinition ` 接口来定义的。该接口包含与事务属性有关的方法。具体如清单1所示：
清单1. TransactionDefinition 接口中定义的主要方法
```

public interface TransactionDefinition{
int getIsolationLevel();//隔离级别
int getPropagationBehavior();//传播行为
int getTimeout();//超时
boolean isReadOnly();//只读
}

```

也许你会奇怪，为什么接口只提供了获取属性的方法，而没有提供相关设置属性的方法。其实道理很简单，事务属性的设置完全是程序员控制的，因此程序员可以自定义任何设置属性的方法，而且保存属性的字段也没有任何要求**。唯一的要求的是，Spring 进行事务操作的时候，通过调用以上接口提供的方法必须能够返回事务相关的属性取值。**

###  事务隔离级别(ISOLATION Level) 
隔离级别是**指若干个并发的事务之间的隔离程度**。` TransactionDefinition ` 接口中定义了**五个表示隔离级别的常量**：
  * TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。**对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。**
  * TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，**因此很少使用该隔离级别。**
  * TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，**这也是大多数情况下的推荐值。**
  * TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。**但是这将严重影响程序的性能。通常情况下也不会用到该级别。**
###  事务传播行为(PROPAGATION) 
所谓事务的传播行为是指，**如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为**。在` TransactionDefinition `定义中包括了如下几个表示**传播行为的常量：**
  * TransactionDefinition.PROPAGATION_REQUIRED：**如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。**
  * TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
  * TransactionDefinition.PROPAGATION_SUPPORTS：**如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。**
  * TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
  * TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
  * TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
  * TransactionDefinition.PROPAGATION_NESTED：**如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。**
这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。**而 PROPAGATION_NESTED是 Spring 所特有的。**以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，**只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。**如果熟悉 JDBC 中的` 保存点（SavePoint） `的概念，那嵌套事务就很容易理解了，**其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。**
###  事务超时(Timeout) 
所谓事务超时，就是**指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务**。在 ` TransactionDefinition ` 中以 int 的值来表示超时时间，其**单位是秒**。
###  事务的只读属性(Read-only) 
事务的只读属性是指，**对事务性资源进行只读操作或者是读写操作**。所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。**如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能**。在`  TransactionDefinition ` 中以 boolean 类型来表示该事务是否只读。
##  事务的回滚规则 
通常情况下，**如果在事务中抛出了` 未检查异常（继承自 RuntimeException 的异常） `，则` 默认将回滚事务 `。如果没有抛出任何异常，或者` 抛出了已检查异常，则仍然提交事务 `。**这通常也是大多数开发者希望的处理方式，也是 EJB 中的默认处理方式。但是，我们可以根据需要人为控制事务在抛出某些未检查异常时任然提交事务，或者在抛出某些已检查异常时回滚事务。

##  声明式事务管理 
###  基于 <tx> 命名空间的声明式事务管理 
前面两种声明式事务配置方式奠定了 Spring 声明式事务管理的基石。(注：本文在顺序上稍有改动，这部分放到后面附录中参考)在此基础上，` Spring 2.x 引入了 <tx> 命名空间，结合使用 <aop> 命名空间，带给开发人员配置声明式事务的全新体验 `，配置变得更加简单和灵活。另外，得益于 <aop> 命名空间的切点表达式支持，声明式事务也变得更加强大。
清单10. 基于 <tx> 的事务管理示例配置文件
可以参考：http://www.cnblogs.com/rushoooooo/archive/2011/08/28/2155960.html
```

<beans......>
......
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
   <property name="sessionFactory">
       <ref bean="mySessionFactory"/>
   </property>
</bean>
  
    <!--  配置事务特性 -->
<tx:advice id="bankAdvice" transaction-manager="transactionManager">
<tx:attributes>
<tx:method name="save*" propagation="REQUIRED"/>
  <tx:method name="add*" propagation="REQUIRED"/>
  <tx:method name="update*" propagation="REQUIRED"/>
  <tx:method name="del*" propagation="REQUIRED"/>
  <tx:method name="find*" propagation="REQUIRED"/>
  <tx:method name="*" propagation="SUPPORTS"/>
</tx:attributes>
</tx:advice>
<!--  配置参与事务的类 -->
<aop:config>
<aop:pointcut id="bankPointcut" expression="execution(* com.test.testAda.test.model.service.*.*(..))"/>
<aop:advisor advice-ref="bankAdvice" pointcut-ref="bankPointcut"/>
</aop:config>
......
</beans>

```

由于使用了切点表达式，我们就不需要针对每一个业务类创建一个代理对象了。另外，如果配置的事务管理器 Bean 的名字取值为“transactionManager”，则我们可以省略 <tx:advice> 的 transaction-manager 属性，因为该属性的默认值即为“transactionManager”。

**需要注意的地方:**
（1） **advice（建议）的命名**：由于` 每个模块都会有自己的Advice `，所以在命名上需要作出规范，初步的构想就是**模块名+Advice（只是一种命名规范）。**
（2）`  tx:attribute标签 `所配置的是作为事务的**方法的命名类型**。
 如` <tx:method name="save*" propagation="REQUIRED"/> `
其中*为通配符，即代表以save为开头的所有方法，即表示符合此命名规则的方法作为一个事务。propagation="REQUIRED"代表支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。

（3）`  aop:pointcut `标签**配置参与事务的类**，由于是` 在Service中进行数据库业务操作 `，配的应该是包含那些**作为事务的方法的Service类**。
首先应该特别注意的是id的命名，同样由于每个模块都有自己事务切面，所以我觉得初步的命名规则因为 all+模块名+ServiceMethod。而且每个模块之间不同之处还在于以下一句：
expression="execution(* com.test.testAda.test.model.service.*.*(..))"
**其中第一个*代表返回值，第二*代表service下子包，第三个*代表方法名，“（..）”代表方法参数。**

（4）`  aop:advisor `标签就是把上面我们**所配置的事务管理两部分属性` 整合 `起来作为整个事务管理。**

###  基于 @Transactional 的声明式事务管理 
**除了基于命名空间的事务配置方式，Spring 2.x 还引入了基于 Annotation 的方式**，具体主要涉及` @Transactional ` 标注。
**@Transactional 可以作用于接口、接口方法、类以及类方法上**。当作用于类上时，该类的所有`  public 方法 `将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。
清单12. 基于 @Transactional 的事务管理示例配置文件
```

@Transactional(propagation = Propagation.REQUIRED)
public boolean transfer(Long fromId， Long toId， double amount) {
return bankDao.transfer(fromId， toId， amount);
}

```
Spring 使用 BeanPostProcessor 来处理 Bean 中的标注，因此我们需要在配置文件中作如下声明来激活该后处理 Bean，如清单13所示：
清单13. 启用后处理Bean的配置
```

<tx:annotation-driven transaction-manager="transactionManager"/>

```
与前面相似，transaction-manager 属性的默认值是 transactionManager，如果事务管理器 Bean 的名字即为该值，则可以省略该属性。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，**但是 Spring 小组建议不要在接口或者接口方法上使用该注解，**因为这只有在使用基于接口的代理时它才会生效。另外， ` @Transactional 注解应该只被应用到 public 方法上 `，**这是由 Spring AOP 的本质决定的**。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，**这将被忽略，也不会抛出任何异常。**

**基于 <tx> 命名空间和基于 @Transactional 的事务声明方式各有优缺点。**
基于 <tx> 的方式，其优点是与切点表达式结合，功能强大。利用切点表达式，一个配置可以匹配多个方法，
而基于 @Transactional 的方式必须在每一个需要使用事务的方法或者类上用 @Transactional 标注，尽管可能大多数事务的规则是一致的，但是对 @Transactional 而言，也无法重用，必须逐个指定。

另一方面，基于 @Transactional 的方式使用起来非常简单明了，没有学习成本。**开发人员可以根据需要，任选其中一种使用，甚至也可以根据需要混合使用这两种方式。**

如果不是对遗留代码进行维护，则不建议再使用基于 TransactionInterceptor 以及基于TransactionProxyFactoryBean 的声明式事务管理方式，但是，学习这两种方式非常有利于对底层实现的理解。
虽然上面共列举了四种声明式事务管理方式，但是这样的划分只是为了便于理解，其实后台的实现方式是一样的，只是用户使用的方式不同而已。

##  结束语 
本教程的知识点大致总结如下：
  * 基于 TransactionDefinition、PlatformTransactionManager、TransactionStatus 编程式事务管理是 Spring 提供的最原始的方式，通常我们不会这么写，但是了解这种方式对理解 Spring 事务管理的本质有很大作用。
  * 基于 TransactionTemplate 的编程式事务管理是对上一种方式的封装，使得编码更简单、清晰。
  * 基于 TransactionInterceptor 的声明式事务是 Spring 声明式事务的基础，通常也不建议使用这种方式，但是与前面一样，了解这种方式对理解 Spring 声明式事务有很大作用。
  * 基于 TransactionProxyFactoryBean 的声明式事务是上中方式的改进版本，简化的配置文件的书写，这是 Spring 早期推荐的声明式事务管理方式，但是在 Spring 2.0 中已经不推荐了。
  * **基于 <tx> 和 <aop> 命名空间的声明式事务管理是` 目前推荐 `的方式，**其最大特点是与 Spring AOP 结合紧密，可以充分利用切点表达式的强大支持，使得管理事务更加灵活。
  * **基于 @Transactional 的方式将声明式事务管理简化到了极致**。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置，然后在需要实施事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不必其他方式逊色。

更多请参考：http://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/
http://www.cnblogs.com/rushoooooo/archive/2011/08/28/2155960.html