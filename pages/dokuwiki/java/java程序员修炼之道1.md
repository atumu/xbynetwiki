title: java程序员修炼之道1 

#  Java程序员修炼之道之依赖注入 
内容大纲
  * 控制反转IoC和依赖注入DI
  * JSR-330同一个Java中的DI。以及注解介绍，如@Inject
  * Guice3简介，JSR-330的参考实现

IoC是一种机制，使用这种机制的用例很多，实现方式很多，DI只是其中一种具体用例的具体实现方式。
DI能降低代码之间的耦合度，让代码更易于测试、更易读。
Guice3是JSR-330的参考实现，一个轻量、精巧的DI框架
DI是IoC的一种特定形态,DI的好处列表如下:
  * 松耦合
  * 易测性
  * 更强的内聚性
  * 可重用的组件
  * 更轻盈的代码
DI的关键思想:面向接口编程。
##  Java中标准化的DI 
javax.inject包
  * Provider<T>接口
  * 5个注解类型@Inject,@Qualifier、@Named、@Scope、@Singleton
@Inject注解：直接通过类型匹配注入
@Qualifire限定(标识)要注入的对象，而不是笼统地通过类型匹配。
@Named:@Qualifier的默认实现，可以用名字标记要注入的对象。将@Named和@Inject一起使用，符合指定名称并且类型正确的对象会被注入。
@Scope注解:用于定义注入器(即IoC容器)对注入对象的重用方式。
@Singletion：@Scope的默认实现。以单例方式重用要注入的对象。大多数DI框架都将@Singletion作为注入对象的默认生命周期。无需显式声明。
接口Provider<T>：如果你想对DI框架注入代码中的对象拥有更多的控制权，可以要求DI框架将Provider<T>接口实现注入对象<T>.控制对象的好处在于:
  * 可以获取该对象的多个实例
  * 可以延迟获取该对象
  * 可以打破循环依赖
  * 可以定义作用域。
```

class MurmurMessage {
  boolean someGlobalCondition = true;

  @Inject
  MurmurMessage(Provider<Message> messageProvider) {
    Message msg1 = messageProvider.get();
    if (someGlobalCondition) {
      Message copyOfMsg1 = messageProvider.get();
    }
    // Do stuff with msg1 and copyOfMsg1 得到Message的副本。
  }
  private class Message {
  }
}

```
##  Guice3简介 
github:https://github.com/google/guice
```

<dependency>
  <groupId>com.google.inject</groupId>
  <artifactId>guice</artifactId>
  <version>4.0</version>
</dependency>

```
Guice(读'Juice').是JSR-330规范的完整参考实现。
常用语:对象关系图、绑定、模块、注入器
通过模块这个配置类，我们可以先创建各种绑定关系。以完成依赖项的配置。
```

import com.google.inject.AbstractModule;
public class AgentFinderModule extends AbstractModule { //模块继承自AbstractModule
  @Override
  protected void configure() {
    //绑定接口到一个实现.这就是声明绑定关系，这样注入器就会构建对象关系图。
    bind(AgentFinder.class).to(WebServiceAgentFinder.class); //定义接口AgentFinder与具体实现WebServiceAgentFinder之间的绑定关系。
  }
}

```
```

public class HollywoodServiceGuice {

  private AgentFinder finder = null;

  @Inject
  public HollywoodServiceGuice(AgentFinder agentFinder) {
    this.finder = agentFinder;
  }
  ....

```

###  JSE程序构建对象关系图 
```

import com.google.inject.Guice;
import com.google.inject.Injector;
import com.java7developer.chapter3.listing_3_9.AgentFinderModule;
public class HollywoodServiceClient {
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(new AgentFinderModule());
    HollywoodServiceGuice hollywoodService = injector
        .getInstance(HollywoodServiceGuice.class);
    List<Agent> agents = hollywoodService.getFriendlyAgents();
    // Do stuff with agents.
  }

}

```
###  构建web应用程序的对象关系图 
web.xml配置:
```

 <filter>
    <filter-name>guiceFilter</filter-name>
    <filter-class>com.google.inject.servlet.GuiceFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>guiceFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

```
扩展GuiceServletContextListener以便使用Guice的ServletModule
```

public class MyGuiceServletConfig extends GuiceServletContextListener {
  @Override
  protected Injector getInjector() {
    return Guice.createInjector(new ServletModule());
  }
}

```
最后一步，把下面这些配置加到web.xml文件中:
```

<listener>
  <listener-class>com.example.MyGuiceServletConfig</listener-class>
</listener>

```

###  Guice的各种绑定 
Guice提供了多种绑定方式:
  * 链接绑定
  * 绑定注解
  * 实例绑定
  * @Provides方法
  * Provide绑定
  * 无目标绑定
  * 内置绑定
  * 即时绑定
我们就讲一下最常用的一些绑定:链接绑定、绑定注解、@Provides方法和Provider<T>绑定。

**1、链接绑定:-最简单的绑定**
```

public class AgentFinderModule extends AbstractModule { //模块继承自AbstractModule
  @Override
  protected void configure() {
    //绑定接口到一个实现.这就是声明绑定关系，这样注入器就会构建对象关系图。
    bind(AgentFinder.class).to(WebServiceAgentFinder.class); //定义接口AgentFinder与具体实现WebServiceAgentFinder之间的绑定关系。
  }
}

```

**2、绑定注解**：将注入类的类型和额外的标识符组合起来，以标识恰当的注入对象。可以通过@Qualifier自定义绑定注解,不过我们使用@Named标注注解。
```

import com.google.inject.AbstractModule;
import com.google.inject.name.Names;
public class AgentFinderModule extends AbstractModule {
  @Override
  protected void configure() {
    bind(AgentFinder.class).annotatedWith(Names.named("primary")).to(
        WebServiceAgentFinder.class);
  }

```
使用:
```

public class HollywoodService {

  private AgentFinder finder = null;

  @Inject
  public HollywoodService(@Named("primary") AgentFinder finder) {
    this.finder = finder;
  }
}

```

**3、@Provides和Provider<T>:提供完全定制的对象**
注入器会查看所有标记了@Provides注解方法的返回类型，以决定要注入哪个对象。
```

import com.google.inject.AbstractModule;
import com.google.inject.Provides;
public class AgentFinderModule extends AbstractModule {

  @Override
  protected void configure() {
  }
  @Provides
  AgentFinder provideAgentFinder() {
    SpreadsheetAgentFinder finder = new SpreadsheetAgentFinder();
    finder.setType("Excel 97");
    finder.setPath("c:/temp/agents.xls");
    return finder;
  }
}

```
**这中方式有个缺点，那就是会导致@Provides方法越来越多，为了不把模块类撑爆。我们可以提供实现Provider<T>接口的类。**
```

public class AgentFinderProvider implements Provider<AgentFinder> {

  @Override
  public AgentFinder get() {
    SpreadsheetAgentFinder finder = new SpreadsheetAgentFinder();
    finder.setType("Excel 97");
    finder.setPath("C:/temp/agents.xls");
    return finder;
  }
}

```
```

public class AgentFinderModule extends AbstractModule {

  @Override
  protected void configure() {
    bind(AgentFinder.class).toProvider(AgentFinderProvider.class);
  }
}

```

###  限定注入对象的生命周期 
Guice提供了不同级别的生命周期:
  * @RequestScoped -请求级别
  * @SessionScoped -会话级别
  * @Singletion -应用级别
应用依赖项的生命周期方式:
  * 在要注入的类中提供注解
  * 作为绑定声明的一部分(比如bind().to().in())
  * 和@Provides一起使用注解声明。
**1、在要注入的类中提供注解**
```

@Singletion
public class A{
  ...
}

```
**2、用bind()方法设置生命周期**
bind()后加上.in(<Scope>.class)就行
```

    bind(AgentFinder.class).annotatedWith(Names.named("primary")).to(
        WebServiceAgentFinder.class).in(Singleton.class);

```
**3、设置@Provides对象的生命周期**
```

  @Provides @Singleton
  TransactionLog provideTransactionLog() {
    ...
  }

```
