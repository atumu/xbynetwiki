title: guava学习之eventbus 

#  Guava学习之EventBus 
传统上，Java的进程内事件分发都是通过发布者和订阅者之间的显式注册实现的。设计EventBus就是为了取代这种显示注册方式，使组件间有了更好的解耦。EventBus不是通用型的发布-订阅实现，不适用于进程间通信。
```

// Class is typically registered by the container.
class EventBusChangeRecorder {
  @Subscribe public void recordCustomerChange(ChangeEvent e) {
    recordChange(e.getChange());
  }
}
// somewhere during initialization
eventBus.register(new EventBusChangeRecorder());
// much later
public void changeCustomer() {
  ChangeEvent event = getChangeEvent();
  eventBus.post(event);
}

```
把已有的进程内事件分发系统迁移到EventBus非常简单。
##  事件监听者[Listeners] 
监听特定事件（如，CustomerChangeEvent）：
  * 传统实现：定义相应的事件监听者类，如CustomerChangeEventListener；
  * EventBus实现：以CustomerChangeEvent为唯一参数创建方法，并用` @Subscribe `注解标记。

#### = 把事件监听者注册到事件生产者 
  * 传统实现：调用事件生产者的registerCustomerChangeEventListener方法；这些方法很少定义在公共接口中，因此开发者必须知道所有事件生产者的类型，才能正确地注册监听者；
  * EventBus实现：在EventBus实例上调用**EventBus.register(Object)**方法；` 请保证事件生产者和监听者共享相同的EventBus实例 `。

###  按事件超类监听（如，EventObject甚至Object）： 
传统实现：很困难，需要开发者自己去实现匹配逻辑；
EventBus实现：EventBus自动把事件分发给事件超类的监听者，并且允许监听者声明监听接口类型和泛型的通配符类型（wildcard，如 ? super XXX）。
###  检测没有监听者的事件： 
传统实现：在每个事件分发方法中添加逻辑代码（也可能适用AOP）；
EventBus实现：**监听DeadEvent**；EventBus会把所有发布后没有监听者处理的事件包装为DeadEvent（对调试很便利）。
##  事件生产者[Producers] 
###  管理和追踪监听者： 

传统实现：用列表管理监听者，还要考虑线程同步；或者使用工具类，如EventListenerList；
EventBus实现：EventBus内部已经实现了监听者管理。
###  向监听者分发事件： 
传统实现：开发者自己写代码，包括事件类型匹配、异常处理、异步分发；
EventBus实现：**把事件传递给 EventBus.post(Object)方法**。**异步分发可以直接用EventBus的子类AsyncEventBus。**

##  EventBus基本用法： 
参考：http://javarticles.com/2015/04/guava-eventbus-examples.html
使用Guava之后, 如果要订阅消息, 就不用再继承指定的接口, 只需要在指定的方法上加上` @Subscribe `注解即可。代码如下：
**Guava发布的事件默认不会处理线程安全的，但我们可以标注` @AllowConcurrentEvents `来保证其线程安全**
```

public class SimpleListener {
    @Subscribe
    public void task(String s) {
        System.out.println("do task(" + s + ")");
    }
}

```
```

public class SimpleEventBusExample {
    public static void main(String[] args) {
        EventBus eventBus = new EventBus();
        eventBus.register(new SimpleListener());
        System.out.println("Post Simple EventBus Example");
        eventBus.post("Simple EventBus Example");
    }
}

```
Output:
Post Simple EventBus Example
do task(Simple EventBus Example)
###  MultiListener的使用： 
只需要在要订阅消息的方法上加上@Subscribe注解即可实现对多个消息的订阅，代码如下：
```

public class MultipleListeners {
    @Subscribe
    public void task1(String s) {
        System.out.println("do task1(" + s +")");
    }
     
    @Subscribe
    public void task2(String s) {
        System.out.println("do task2(" + s +")");
    }
     
    @Subscribe
    public void intTask(Integer i) {
        System.out.println("do intTask(" + i +")");
    }
}

```
测试类：
```

public class MultipleEventTypeBusExample {
    public static void main(String[] args) {
        EventBus eventBus = new EventBus();
        eventBus.register(new MultipleListeners());
        System.out.println("Post 'Multiple Listeners Example'");
        eventBus.post("Multiple Listeners Example");      
        eventBus.post(1);
    }
}

```
输出信息
Post 'Multiple Listeners Example'
do task2(Multiple Listeners Example)
do task1(Multiple Listeners Example)
do intTask(1)
###  Dead Event： 
如果EventBus发送的消息都不是订阅者关心的称之为Dead Event。实例如下：
说明：如果没有消息订阅者监听消息， EventBus将发送DeadEvent消息，这时我们可以通过log的方式来记录这种状态。
```

public class DeadEventListener {
    boolean notDelivered = false;  
    @Subscribe  
    public void listen(DeadEvent event) {  
        
        notDelivered = true;  
    }  
   
    public boolean isNotDelivered() {  
        return notDelivered;  
    }  
}

```
测试类：
```

public class TestDeadEventListeners {
    @Test  
    public void testDeadEventListeners() throws Exception {  
       
        EventBus eventBus = new EventBus("test");               
        DeadEventListener deadEventListener = new DeadEventListener();  
        eventBus.register(deadEventListener);  

        eventBus.post(new Object());            
       
        System.out.println("deadEvent:"+deadEventListener.isNotDelivered());

    }  
}

```
输出信息
deadEvent:true
##  Listener的继承 
In this example, the listener class extends BaseListener which in turn extends AbstractListener. Once an event is published, **all the @Subscribe annotated methods in the listener’s hierarchy will be notified.**
```

public class ListenerHierarchy extends BaseListener {
    @Subscribe
    public void task(String s) {
        System.out.println("do task(" + s +")");
    }    
}
public class BaseListener extends AbstractListener {
    @Subscribe
    public void baseTask(String s) {
        System.out.println("do baseTask(" + s + ")");
    }
}
public abstract class AbstractListener {
    @Subscribe
    public void commonTask(String s) {
        System.out.println("do commonTask(" + s + ")");
    }
}

```
```

public class ListenerHierarchyEventBusExample {
    public static void main(String[] args) {
        EventBus eventBus = new EventBus();
        eventBus.register(new ListenerHierarchy());
        System.out.println("Post 'Listener Hierarchy Example'");
        eventBus.post("Listener Hierarchy Example");      
    }
}

```
输出：
Post 'Listener Hierarchy Example'
do commonTask(Listener Hierarchy Example)
do task(Listener Hierarchy Example)
do baseTask(Listener Hierarchy Example)
##  Event的继承： 
如果Listener A监听Event A, 而Event A有一个子类Event B, **此时Listener A将同时接收Event A和B消息**，实例如下：
Listener 类：
```

public class FruitEaterListener {
    @Subscribe
    public void eat(Fruit fruit) throws RawFruitException {
        System.out.println("eat(Fruit " + fruit +")");
    }  
     
    @Subscribe
    public void eat(Apple apple) {
        System.out.println("eat(" + apple +")");
    } 
}

public class Fruit {
    private String name;
    public Fruit(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }   
    public String toString() {
        return name;
    }
}
public class Apple extends Fruit {
 
    public Apple() {
        super("Apple");
    }
 
}
测试类：
public class EventHierarchyEventBusExample {
    public static void main(String[] args) {
        EventBus eventBus = new EventBus();
        eventBus.register(new FruitEaterListener());
        System.out.println("Post 'Apple'");
        eventBus.post(new Apple());  
        System.out.println("Post 'Orange as Fruit'");
        eventBus.post(new Fruit("Orange"));  
    }
}

```
输出
Post 'Apple'
eat(Apple)
eat(Fruit Apple)
Post 'Orange as Fruit'
eat(Fruit Orange)

##  Unregister Event 
```

public class UnregisterEventBusExample {
    public static void main(String[] args) {
        EventBus eventBus = new EventBus();
        SimpleListener simpleListener = new SimpleListener();
        eventBus.register(simpleListener);
        System.out.println("Post Simple EventBus Example");
        eventBus.post("Simple EventBus Example");
        System.out.println("Unregister the subscriber");
        eventBus.unregister(simpleListener);
        System.out.println("Post Simple EventBus Example once again");
        eventBus.post("Simple EventBus Example");
    }
}

```
##  将EventBus封装为单例模式使用 
```

public class EventBusUtil {
	private static final EventBus gBus;
	/**
	 * 保存已经注册的监听器，防止监听器重复注册
	 */
	private static final ConcurrentHashMap<String, Object> registerListenerContainers;

	static {
		gBus = new EventBus();
		registerListenerContainers = new ConcurrentHashMap<>();
	}

	public static void post(Object obj) {
		gBus.post(obj);
	}

	public static void register(Object obj) {
		String clazzName = obj.getClass().getName();
		if (!registerListenerContainers.containsKey(clazzName)) {
			synchronized (registerListenerContainers) {
				if (!registerListenerContainers.containsKey(clazzName)) {
					registerListenerContainers.put(clazzName, obj);
					gBus.register(obj);
				}
			}
		}
	}

	public static void unregister(Object obj) {
		unregister(obj.getClass());
	}

	public static void unregister(Class clazz) {
		synchronized (registerListenerContainers) {
			Object obj = registerListenerContainers.remove(clazz.getName());
			if (obj != null) {
				gBus.unregister(obj);
			}
		}
	}

	public static Object getRegisterObject(Class clazz) {
		return registerListenerContainers.get(clazz.getName());

	}

}


```
##  术语表 

事件总线系统使用以下术语描述事件分发：

事件	可以向事件总线发布的对象
**订阅**	向事件总线注册监听者以接受事件的**行为**
监听者	提供一个处理方法，希望接受和处理事件的对象
**处理方法**	监听者提供的公共方法，事件总线使用该方法向监听者发送事件；该方法应该用**@Subscribe**注解
发布消息	通过事件总线向所有匹配的监听者提供事件
##  常见问题解答[FAQ] 
###  为什么一定要创建EventBus实例，而不是使用单例模式？ 
EventBus不想给定开发者怎么使用；你可以在应用程序中按照不同的组件、上下文或业务主题分别使用不同的事件总线。这样的话，在测试过程中开启和关闭某个部分的事件总线，也会变得更简单，影响范围更小。
当然，如果你想在进程范围内使用唯一的事件总线，你也可以自己这么做。比如在容器中声明EventBus为全局单例，或者用一个静态字段存放EventBus，如果你喜欢的话。
简而言之，**EventBus不是单例模式，是因为我们不想为你做这个决定。你喜欢怎么用就怎么用吧**。
###  可以从事件总线中注销监听者吗？ 
当然可以，使用**EventBus.unregister(Object)**方法，但我们发现这种需求很少：
大多数监听者都是在启动或者模块懒加载时注册的，并且在应用程序的整个生命周期都存在；
可以使用特定作用域的事件总线来处理临时事件，而不是注册/注销监听者；比如在请求作用域[request-scoped]的对象间分发消息，就可以同样适用请求作用域的事件总线；
**销毁和重建事件总线的成本很低**，有时候可以通过销毁和重建事件总线来更改分发规则。
###  如果我注册了一个没有任何处理方法的监听者，会发生什么？ 
` 什么也不会发生。 `这与Android中的GreenRobot开发的EventBus不同。后者将会导致错误。
EventBus旨在与容器和模块系统整合，Guice就是个典型的例子。在这种情况下，可以方便地让容器/工厂/运行环境传递任意创建好的对象给EventBus的register(Object)方法。
这样，任何容器/工厂/运行环境创建的对象都可以简便地通过暴露处理方法挂载到系统的事件模块。
###  为什么我不能在EventBus上使用<泛型魔法>？ 
EventBus旨在很好地处理一大类用例。我们更喜欢针对大多数用例直击要害，而不是在所有用例上都保持体面。
此外，泛型也让EventBus的可扩展性——让它有益、高效地扩展，同时我们对EventBus的增补不会和你们的扩展相冲突——成为一个非常棘手的问题。
**如果你真的很想用泛型，EventBus目前还不能提供**，你可以提交一个问题并且设计自己的替代方案。
参考：
http://ifeve.com/google-guava-eventbus/