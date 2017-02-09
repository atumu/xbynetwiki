title: android_eventbus 

#  android_eventbus库源码分析 
##  概述： 
基于EventBus 2.4版本进行分析。
关于EventBus的使用请看[[opensourcelearn:android_eventbus]]
greenrobot开发的EventBus是一个很优雅的Android开源库。本文将从源码的角度对其进行分析。
下面是其主要类成员：
![](/data/dokuwiki/source/pasted/20150831-101446.png)

##  首先分析EventBus这个类： 
###  属性分析 

**EventBus类会在多线程环境下使用，所以需要确保其为线程安全的。** 
private static final ` EventBusBuilder ` DEFAULT_BUILDER  = new EventBusBuilder();静态初始化，由于构造器参数较多，且大多数为可选参数，所以采用了构建器Builder模式设计。

private static final ` Map<Class<?>, List<Class<?>>> eventTypesCache ` 
**eventTypesCache key为event class对象，value为该event class的所有层级的接口和层级继承类（它通过lookupAllEventTypes得到）。 **很显然事件对象可能有父类或者接口。而注册的方法参数一般都是接口或父类。当post一个子类时，这样我们就能正确地将其分派到对应的注册方法中。
设置这个cache的目的也是为了避免每次对同一个类的对象调用注册方法时进行方法查找，这样便可以提升性能。（SubscriberMethodFinder类的findSubscriberMethods首先会查询这个map，若有则直接返回，否则进行查找。）

 private final ` Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType `;
**subscriptionsByEventType定义了key为EventType的Class对象,value为针对这一事件类型的订阅关系列表**(Subscription是一个订阅关系封装类：封装了订阅者对象，订阅方法SubscriberMethod，优先级)
主要用于EventType与subscription的class对象的一对多对应。（即事件类型与订阅者方法的一对多关系）

private final`  Map<Object, List<Class<?>>> typesBySubscriber `;
**key为订阅者对象，value为订阅的EventType的class对象。**因为一个订阅者对象可能有多个订阅方法，每个订阅方法又有可能有不同的EventType
主要用于Subscriber与EventType的class对象的一对多对应。
**注册与取消注册，主要的工作在于维护上面两个map:subscriptionsByEventType和typesBySubscriber**

 private final ` ThreadLocal<PostingThreadState> currentPostingThreadState `
**内部类：调用post()方法时会对它进行检测。为了线程安全性，它用作全局的ThreadLocal对象。**
其内部维护着当**前线程相关的eventQueue事件队列**。每调用post一次都会添加一个。
```

   /** For ThreadLocal, much faster to set (and get multiple values). 
   *由于它将采用ThreadLocal模式。所以是线程安全的。
   */
    final static class PostingThreadState {
      	//快速初始化事件队列。EventBus上的事件派发队列。由于是ThreadLocal模式，所以会为每个相关维护一个事件派发队列。
        final List<Object> eventQueue = new ArrayList<Object>();
        boolean isPosting;//是否正在派发事件
        boolean isMainThread;//是否在主线程中。即Android中的UI线程
 	Subscription subscription;//与Posting状态相关的订阅关系对象。
 	Object event;//与Posting状态相关的event对象
        boolean canceled;//已经被取消派发？这个标志会在EventBus的cancelEventDelivery(Object event)被调用时设置为true。事件派发后是可以取消的。稍后分析。
 }

```

三个Poster，用于不同的场景
private final ` HandlerPoster ` mainThreadPoster;   用于onEventMainThread(...)，其内部通过Android的Handler机制将事件调用发送到MainThread
private final ` BackgroundPoster ` backgroundPoster;用于onEventBackground(...)，其内部是通过一个线程处理所有事件
private final ` AsyncPoster ` asyncPoster;用于onEventAsync(...)其内部通过一个线程池处理所有事件。

private final ` SubscriberMethodFinder ` subscriberMethodFinder;用于寻找订阅者方法
private final ` ExecutorService ` executorService;  便利地提供一个线程池。

通过以上属性分析，你也许注意到所有的属性声明都是` private  final ` 因为我们需要确保线程安全性：采用private封装属性，可见性最小化，采用final，尽可能减少可变因素。

![](/data/dokuwiki/source/pasted/20150904-060928.png)
###  EventBus获取默认实例 

线程安全的单例模式
```

  public static EventBus getDefault() {
    //线程安全的双重检查机制
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
   }
 }
 }
        return defaultInstance;
 }

```
###  EventBus中的构建器模式 
```

    //对外提供一个构建器
    public static EventBusBuilder builder() {
        return new EventBusBuilder();
    }

   /**用于获取一个EventBus实例，可以看到这个构造器并非私有的,那么为什么提供了单例的getDefault()方法还要公开构造器呢？
   *其实这样做的目的是很明显的，我们的应用程序不可能只需要一条事件总线，很多情况下我们需要多个事件总线来协调应用中不同部分的通信。
   *所有如果默认的事件总线单例满足不了你，你可以自己新建多个实例，用于不同场景。这与Guava中的Eventbus是一致的。
   */
    public EventBus() {
        this(DEFAULT_BUILDER);
    }
    /**当构造器有多个参数，且大多数参数是可选的时候我们采用了构建器 */
    EventBus(EventBusBuilder builder) {
      	//使用CopyOnWriteArrayList这个并发List
        subscriptionsByEventType = new HashMap<Class<?>, CopyOnWriteArrayList<Subscription>>();
        typesBySubscriber = new HashMap<Object, List<Class<?>>>();
      
      	//使用并发集合ConcurrentHashMap
        stickyEvents = new ConcurrentHashMap<Class<?>, Object>();
      
        mainThreadPoster = new HandlerPoster(this, Looper.getMainLooper(), 10);
        backgroundPoster = new BackgroundPoster(this);
        asyncPoster = new AsyncPoster(this);
      
        subscriberMethodFinder = new SubscriberMethodFinder(builder.skipMethodVerificationForClasses);
      
        logSubscriberExceptions = builder.logSubscriberExceptions;
        logNoSubscriberMessages = builder.logNoSubscriberMessages;
        sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
        sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
        throwSubscriberException = builder.throwSubscriberException;
      	//eventtype是否启用继承层次寻找标志，默认为true启用。这影响到eventTypesCache这个map以及EventBus的lookupAllEventTypes的行为。
        eventInheritance = builder.eventInheritance;
        executorService = builder.executorService;
    }

```
###  SubscriberMethod与Subscription 
```

/** SubscriberMethod订阅方法类。维护订阅方法、ThreadMode以及EventType等。 采用final修饰保证可变性最小化
*很显然该类是线程安全的不可变类。
*/
  final class SubscriberMethod {
    final Method method;//保存注册的方法
    final ThreadMode threadMode;//枚举对象，保存事件传递调用的模式。决定了事件传递时采用什么方式调用注册方法
    final Class<?> eventType;//事件类class对象
    /** Used for efficient comparison 
    * String不用final进行修饰的原因在于String本身就是不可变的。
    */
    String methodString;

    SubscriberMethod(Method method, ThreadMode threadMode, Class<?> eventType) {
        this.method = method;
        this.threadMode = threadMode;
        this.eventType = eventType;
    }
  //这部分是equals与hashCode重写，这是必要的。
     @Override
    public boolean equals(Object other) {
        if (other instanceof SubscriberMethod) {
            checkMethodString();
            SubscriberMethod otherSubscriberMethod = (SubscriberMethod)other;
            otherSubscriberMethod.checkMethodString();
            // Don't use method.equals because of http://code.google.com/p/android/issues/detail?id=7811#c6
            return methodString.equals(otherSubscriberMethod.methodString);
        } else {
            return false;
        }
    }

    private synchronized void checkMethodString() {
        if (methodString == null) {
            // Method.toString has more overhead, just take relevant parts of the method
            StringBuilder builder = new StringBuilder(64);
            builder.append(method.getDeclaringClass().getName());
            builder.append('#').append(method.getName());
            builder.append('(').append(eventType.getName());
            methodString = builder.toString();
        }
    }

    @Override
    public int hashCode() {
        return method.hashCode();
    }
}

/**Subscription 订阅关系类。维护订阅对象和订阅方法等。采用final修饰，保证可变性最小化
*虽然这个类不是不可变类，也不是严格意义上的线程安全类，但它是事实上的线程安全类。
*唯一可变的就是subscriber和active.但是由于subscriber实际上我们不会受其状态影响而且也不会对齐状态作出改变。而active由于是volatile boolean保证可见性。而且由于boolean操作一般为简单赋值操作JVM能够保证其原子  性。所以也可以认为是线程安全的。
*/
final class Subscription {
  	//下面的属性均为final以保证可变性最小化。
    final Object subscriber;//订阅者对象。subscriber这里声明类型为final Object，故而隐去了其原有的方法同时保证了final。所以可以认为是不可变的。
    final SubscriberMethod subscriberMethod;//订阅方法。不可变
    final int priority;//优先级。不可变
    /**
     *标志Subscription是否可用。一旦被EventBus#unregister(Object)调用就被设为false。同时它会被EventBus#invokeSubscriber(PendingPost)中的事件分发队列进行检测以防止发生竟态条件
     *volatile修饰符保证了可见性，JVM能够提供非复合赋值操作的原子性。所以简单赋值操作下可以保证线程安全性。
     */
    volatile boolean active;//可变，但是简单赋值情况下可保证原子性。

    Subscription(Object subscriber, SubscriberMethod subscriberMethod, int priority) {
        this.subscriber = subscriber;
        this.subscriberMethod = subscriberMethod;
        this.priority = priority;
        active = true;
    }

    @Override
    public boolean equals(Object other) {
        if (other instanceof Subscription) {
            Subscription otherSubscription = (Subscription) other;
            return subscriber == otherSubscription.subscriber
                    && subscriberMethod.equals(otherSubscription.subscriberMethod);
        } else {
            return false;
        }
    }

    @Override
    public int hashCode() {
        return subscriber.hashCode() + subscriberMethod.methodString.hashCode();
    }
}

```
###  ThreadMode枚举类 
定义了四种传递事件的机制。
```

public enum ThreadMode {
PostThread,//在当前线程中调用注册方法，对应于onEvent()
MainThread,//在主线程即UI线程中调用注册方法,对应于onEventMainThread()
BackgroundThread,//在一个后台线程中调用注册方法，对应于onEventBackgroundThread()
Async//开启一个新线程调用注册方法，对应于onEventAsync()
}

```
###  事件注册 
![](/data/dokuwiki/source/pasted/20150904-060641.png)
在EventBus上进行注册的时候会通过` SubscriberMethodFinder `查找所有的注册方法，因为一个注册者会有多个注册方法，然后调用subscribe方法进行整理。
` register `有多个重载的方法。
```

/**采用同步方法保证线程安全性。因为注册远比事件分发要少得多，所以对事件注册采用同步方法对性能影响不大。同时Java6以后同步机制也得到较大的性能提升。*/
private synchronized void register(Object subscriber, boolean sticky, int priority) {
  //关于subscriberMethodFinder源码稍后进行分析。
 List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass());
  
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
          //将找到的注册方法进行subscriptionsByEventType和typesBySubscriber两个map维护。
	 subscribe(subscriber, subscriberMethod, sticky, priority);
 	}
 }

```
` subscribe(subscriber, subscriberMethod, sticky, priority) `;方法负责维护` subscriptionsByEventType和typesBySubscriber `两个map：前者主要用于EventType与subscription的class对象的一对多对应。后者主要用于Subscriber与EventType的class对象的一对多对应。
```

// Must be called in synchronized block。这个方法没有进行同步，所以必须在同步块中调用。
    private void subscribe(Object subscriber, SubscriberMethod subscriberMethod, boolean sticky, int priority) {
        Class<?> eventType = subscriberMethod.eventType;
      //维护subscriptionsByEventType这个map
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod, priority);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<Subscription>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
          	//看到了吗？由于这里要通过contains进行比较，所以现在我们知道为什么一定要让Subscription覆盖equals方法和hashCode了。
            if (subscriptions.contains(newSubscription)) {
              /**咦，这里为什么要抛出异常呢？
              *首先我们可以看到EventBusException extends RuntimeException，这是一个非受检异常，所以不会导致程序终止。
              *其实前面我们说过Subscription包含final Object subscriber;而每个对象不能在EventBus上多次调用register进行多次注册。这是不允许的。
              *所以最好的办法是抛出异常提醒使用者不要在同一对象上进行多次注册或者多个方法采用同一个事件接口参数。
              */
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }

        // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
        // subscriberMethod.method.setAccessible(true);
	
      	//根据优先级调整在List中的顺序。
        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || newSubscription.priority > subscriptions.get(i).priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }
	
      /**维护typesBySubscriber这个map.
      *咦，这里为什么没有重复元素判定呢？其实原因就在于，前面维护subscriptionsByEventType时就已经进行过判定。如果出现重复前面已抛出异常，不会执行到本段代码。
      *所以这里就没有必要判定。
      */
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<Class<?>>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);
	
      //sticky事件相关。
        if (sticky) {
            Object stickyEvent;
            synchronized (stickyEvents) {
                stickyEvent = stickyEvents.get(eventType);
            }
            if (stickyEvent != null) {
                // If the subscriber is trying to abort the event, it will fail (event is not tracked in posting state)
                // --> Strange corner case, which we don't take care of here.
                postToSubscription(newSubscription, stickyEvent, Looper.getMainLooper() == Looper.myLooper());
            }
        }
    }

```
你可能已经注意到这个register方法采用了同步方法机制private synchronized void register(...)，我们前面说过，EventBus是处于多线程并发修改的情况下，所以我们必须时刻保证其线程安全性。
你可以看到我们的这个subscribe(subscriber, subscriberMethod, sticky, priority);方法的注释是必须在同步代码块中被调用。
**为什么采用同步方法，而不缩小同步范围？**
  * 第一、因为注册远比事件分发要少得多，所以对事件注册采用同步方法对性能影响不大。同时Java6以后同步机制也得到较大的性能提升。
  * 第二、由于可变条件过多，与其采用多个碎小的同步块而造成地混乱，不如同步整个方法。
####  SubscriberMethodFinder分析 
```

/**线程安全类*/
class SubscriberMethodFinder {
    private static final String ON_EVENT_METHOD_NAME = "onEvent";//所有注册方法以onEvent开头。

    /*
     * In newer class files, compilers may add methods. Those are called bridge or synthetic methods.
     * EventBus must ignore both. There modifiers are not public but defined in the Java class file format:
     * http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.6-200-A.1
     *这里说得很详细了。
     */
    private static final int BRIDGE = 0x40;
    private static final int SYNTHETIC = 0x1000;
	/**需要忽略的方法修饰符包括abstract,static,以及编译器生成的BRIDGE 和SYNTHETIC*/  
    private static final int MODIFIERS_IGNORE = Modifier.ABSTRACT | Modifier.STATIC | BRIDGE | SYNTHETIC;
    /**存储找到的注册方法*/
    private static final Map<String, List<SubscriberMethod>> methodCache = new HashMap<String, List<SubscriberMethod>>();
	//不是很明白这个忽略方法校验。
    private final Map<Class<?>, Class<?>> skipMethodVerificationForClasses;

    SubscriberMethodFinder(List<Class<?>> skipMethodVerificationForClassesList) {
        skipMethodVerificationForClasses = new ConcurrentHashMap<Class<?>, Class<?>>();
        if (skipMethodVerificationForClassesList != null) {
            for (Class<?> clazz : skipMethodVerificationForClassesList) {
                skipMethodVerificationForClasses.put(clazz, clazz);
            }
        }
    }
	/**线程安全方法。用于在注册类及其父类继承层级中查找符合要求的注册方法。*/
    List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
        String key = subscriberClass.getName();
        List<SubscriberMethod> subscriberMethods;
      /**这里采用同步代码块，确保线程安全*/
        synchronized (methodCache) {
            subscriberMethods = methodCache.get(key);
        }
        if (subscriberMethods != null) {
            return subscriberMethods;
        }
      	//保存方法
        subscriberMethods = new ArrayList<SubscriberMethod>();
        Class<?> clazz = subscriberClass;

        HashSet<String> eventTypesFound = new HashSet<String>();//后面能看到其用途
        StringBuilder methodKeyBuilder = new StringBuilder();
      //以clazz!=null作为判定循环。
        while (clazz != null) {
            String name = clazz.getName();
          	//防止与JDK类库和Android类库冲突。同时也提高检索效率和性能。
            if (name.startsWith("java.") || name.startsWith("javax.") || name.startsWith("android.")) {
                // Skip system classes, this just degrades performance
                break;
            }

           
            Method[] methods = clazz.getDeclaredMethods();
            for (Method method : methods) {
                String methodName = method.getName();
                if (methodName.startsWith(ON_EVENT_METHOD_NAME)) {//如果方法名以onEvent开头
                    int modifiers = method.getModifiers();//获取所有修饰符
                   // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
                    if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) { //要求方法必须是public的非static，非abstract方法。
                        Class<?>[] parameterTypes = method.getParameterTypes();//获取参数类型
                        if (parameterTypes.length == 1) {//参数必须只有一个
                            String modifierString = methodName.substring(ON_EVENT_METHOD_NAME.length());//获取onEvent后面的字符串。以此判断采用何种ThreadMode
                            ThreadMode threadMode;
                            if (modifierString.length() == 0) {
                                threadMode = ThreadMode.PostThread;
                            } else if (modifierString.equals("MainThread")) {
                                threadMode = ThreadMode.MainThread;
                            } else if (modifierString.equals("BackgroundThread")) {
                                threadMode = ThreadMode.BackgroundThread;
                            } else if (modifierString.equals("Async")) {
                                threadMode = ThreadMode.Async;
                            } else {
                                if (skipMethodVerificationForClasses.containsKey(clazz)) {
                                    continue;
                                } else {
                                    throw new EventBusException("Illegal onEvent method, check for typos: " + method);
                                }
                            }
                            Class<?> eventType = parameterTypes[0];
                            methodKeyBuilder.setLength(0);
                            methodKeyBuilder.append(methodName);
                            methodKeyBuilder.append('>').append(eventType.getName());
                            String methodKey = methodKeyBuilder.toString();
                            if (eventTypesFound.add(methodKey)) { //由于要层级寻找父类继承层次中的所有注册方法，所以需要判断以防重复注册。
                                // Only add if not already found in a sub class
                                subscriberMethods.add(new SubscriberMethod(method, threadMode, eventType));
                            }
                        }
                    } else if (!skipMethodVerificationForClasses.containsKey(clazz)) {
                        Log.d(EventBus.TAG, "Skipping method (not public, static or abstract): " + clazz + "."
                                + methodName);
                    }
                }
            }
            clazz = clazz.getSuperclass();//循环所有继承层次。
        }
        if (subscriberMethods.isEmpty()) {
          //这里可以看到，如果注册了却找不到方法，那么就会报异常，这点与Guava中的EventBus什么事情都不做有点区别。其实想想这样设计更为合理点。没有注册方法那注册还有什么意义呢？是吧。
            throw new EventBusException("Subscriber " + subscriberClass + " has no public methods called "
                    + ON_EVENT_METHOD_NAME);
        } else {
          //同步
            synchronized (methodCache) {
                methodCache.put(key, subscriberMethods);//保存到缓存中。
            }
            return subscriberMethods;//返回找到的所有方法。
        }
    }
     //清除找到的方法缓存。
    static void clearCaches() {
        synchronized (methodCache) {
            methodCache.clear();
        }
    }

}

```
###  取消注册 
![](/data/dokuwiki/source/pasted/20150904-060703.png)
通过维护typesBySubcriber、subscriptionsByEventType达到取消注册的目的。
由` unregister `维护typesBySubcriber：同时会调用` unubscribeByEventType `来间接维护subscriptionsByEventType以达到取消注册的目的。
```

   /** Unregisters the given subscriber from all event classes. 
   *采用同步方法。原因同注册分析。
   */
    public synchronized void unregister(Object subscriber) {
	 List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
        if (subscribedTypes != null) {
          //循环地调用unubscribeByEventType从EventType为key的map中移除该subcriber.
            for (Class<?> eventType : subscribedTypes) {
      		 unubscribeByEventType(subscriber, eventType);
   		}
            typesBySubscriber.remove(subscriber);
	 } 
 }

``` 
` unubscribeByEventType `用于维护` subscriptionsByEventType `来进行取消订阅
 这个方法只会维护subscriptionsByEventType，所以这个方法的调用者需要自己维护typesBySubscriber（其实就是Unregister方法）
```

    /** Only updates subscriptionsByEventType, not typesBySubscriber! Caller must update typesBySubscriber. 
    *注意，这个方法必须在同步代码块中被调用
    */
    private void unubscribeByEventType(Object subscriber, Class<?> eventType) {
        List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions != null) {
            int size = subscriptions.size();
            for (int i = 0; i < size; i++) {
                Subscription subscription = subscriptions.get(i);
                if (subscription.subscriber == subscriber) {
                  //通过设置active属性禁用subscription对象。
                    subscription.active = false;
                    subscriptions.remove(i);
                    i--;//千万别忘了
                    size--;//千万别忘了
                }
            }
        }
    }

```



###  事件发送post 
![](/data/dokuwiki/source/pasted/20150904-060751.png)
post方法：它通过` PostingThreadState （采用ThreadLocal模式） `进行一系列状态判断与设定然后**维护当前线程相关的eventQueue**
具体的post委托给` postSingleEvent `方法。
```

    /** Posts the given event to the event bus. 
    *post方法是线程安全的，这里没有采用同步方法或同步代码块。
    *原因在于post方法中唯一可变状态就是postingState对象，但是它是采用ThreadLocal模式的。所以是线程安全的。没有必要采用同步机制。
    */
    public void post(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();//从ThreadLocal中获取当前线程相关的PostingThreadState对象。
    List<Object> eventQueue = postingState.eventQueue; //获取当前线程相关的事件分派队列
        eventQueue.add(event);//添加事件到当前线程相关的队列
 	
      /**判定是否有事件正在派发，如果有，将事件加入线程上下文的事件队列后则忽略这段直接方法返回。该事件会在 while (!eventQueue.isEmpty()) {循环时随后被派发。
      *但是我们要注意的是事件注册方法调用时执行时间不宜过长，否则会影响到事件派发的效率。导致事件派发队列越来越大。影响性能。
      */
        if (!postingState.isPosting) {
            postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
            postingState.isPosting = true;
            //其实这个判断是否取消只是在由isPosting = false;进入时才会被判断。也就是说不是要想取消特定的事件派发不是随意的。
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
        	}
          
          //注意，有关设置关键状态的行为有必要在try{}finally{}块中进行，以确保行为一致性。这点很重要。
            try {
              /**通过循环遍历当前线程上下文相关的事件分派队列，直到为空，也包括遍历期间加入到队列的。
              *咦？到了这里你应该会很疑惑。我们的post调用是与线程相关的postingState和线程相关的eventQueue联系在一起的。任何时刻当前线程都只会有一个post方法被调用。
              *当前线程正在调用一个post方法时，对于当前线程的其他post()调用甚至其他代码根本不会执行。那么为什么这里还要设定一个while循环来迭代eventQueue呢？你循环一个event有意思么？
              *其实这里隐藏了一个很优雅而严密的考虑。那就是我们也可以在被调用的事件注册方法执行中由该再次调用post()方法。这完全是可以的。那么在这种情况下我们会再次进入post()代码中，
              *然后我们将其加入事件分派队列，然后判定isPosting，结果为true，跳过后续代码直接返回。这样我们的eventQueue就多了一个事件。以此类推。
              *所以这个循环是非常有必要的，必须存在的。也必须结合 if (!postingState.isPosting) 这个条件判定的。
              *举个例子就是A注册了事件总线然后A中有个B注册的方法，B方法体中有调用post()方法。那么当分派到这个B方法时就会出现上述情况。
              */
                while (!eventQueue.isEmpty()) {
                	postSingleEvent(eventQueue.remove(0), postingState);//具体发送任务交给了它。同时从队列中移除。
            }
        } finally {
              //完成后恢复postingState初始状态。
                postingState.isPosting = false;
                postingState.isMainThread = false;
        }
    }
 }

```
```

/**我们前面可以看到post方法没有采用同步机制，调用的postSingleEvent这个方法也没有采用同步机制。
*但是这个方法仍然是线程安全的，因为它并没有修改对象的状态。
*/
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
 Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;//标志，用于是否需要记录日志的判定
        if (eventInheritance) {//默认为true,假定事件继承,这个标志判定是非常严谨的考虑。
         List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);//由于存在多态现象，所以我们根据eventclass对象寻找它实现和继承的树层次所有接口和父类。
            int countTypes = eventTypes.size();
            for (int h = 0; h < countTypes; h++) {
             Class<?> clazz = eventTypes.get(h);
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
         }
     } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
     }
  。。。//日志相关，略去
}
    /** Looks up all Class objects including super classes and interfaces. Should also work for interfaces. 
    *用于查找事件类型所有父类和实现的接口。返回所有父类和接口，同时包括其本身的列表。
    *很显然在它的调用层次上面都没有采用同步机制。但是这个方法需要修改对象的状态变量eventTypesCache。所以在这里对其采用了同步代码块机制。
    *这个方法是线程安全的。
    */
    private List<Class<?>> lookupAllEventTypes(Class<?> eventClass) {
    	//同步代码块，确保eventTypesCache一致性与可见性。
        synchronized (eventTypesCache) {
            List<Class<?>> eventTypes = eventTypesCache.get(eventClass);
            if (eventTypes == null) {
                eventTypes = new ArrayList<Class<?>>();
                Class<?> clazz = eventClass;
                while (clazz != null) {
                    eventTypes.add(clazz);
                    addInterfaces(eventTypes, clazz.getInterfaces());//添加所有实现的接口
                    clazz = clazz.getSuperclass();
                }
                eventTypesCache.put(eventClass, eventTypes);
            }
            return eventTypes;
        }
    }

    /** Recurses through super interfaces.递归添加所有实现的接口
    *由于在调用层次上这个方法是在同步代码块中被调用。所以这里就没有对其进行同步。
    */
    static void addInterfaces(List<Class<?>> eventTypes, Class<?>[] interfaces) {
        for (Class<?> interfaceClass : interfaces) {
            if (!eventTypes.contains(interfaceClass)) {//这一步判定是必须的。
                eventTypes.add(interfaceClass);
                addInterfaces(eventTypes, interfaceClass.getInterfaces());//由于接口可能还有父接口。所以这个递归是必须的。
            }
        }
    }

```
```

具体的post重任还是交给了postSingleEventForEventType
 private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
  	/**可以看到，这里采用并发集合CopyOnWriteArrayList，这是JDK7新加的。我们可不可以不采用这个并发List，而采用普通的ArrayList?
        *答案是不可以的。因为subscriptions可能随时被其他线程修改而我们后面还需要对其进行状态判定，所以采用并发集合可以确保数据始终一致性。
        *那么我们知道CopyOnWriteArrayList是一种修改即复制的一种并发集合。当在修改非常频繁的情况下效率会很低。
        *但是我们可以忽略这种情况。我们知道subscriptions只有在注册与取消注册的情况下才会被修改。而注册与取消注册，相比事件派发，那是非常少的。
        *所以我们可以认为subscriptions所处的环境是读远远大于写的情景。CopyOnWriteArrayList的性能影响可以被忽略。
        */
     CopyOnWriteArrayList<Subscription> subscriptions;
  	//采用同步机制确保subscriptionsByEventType读取一致性。很重要的一点。
        synchronized (this) {
            subscriptions = subscriptionsByEventType.get(eventClass);
     }
        if (subscriptions != null && !subscriptions.isEmpty()) {
            for (Subscription subscription : subscriptions) {
                postingState.event = event;
                postingState.subscription = subscription;
                boolean aborted = false;//由于调用完毕后会对postingState状态进行清除，所以需要一个单独的标志来保存是否被取消。
              //调用之后在finally中进行初始状态恢复是很有必要的。
                try {
                 postToSubscription(subscription, event, postingState.isMainThread); //具体任务还是交给了它
                    aborted = postingState.canceled;//在状态被清除之前进行保存，以便后续使用。
             } finally {
                    postingState.event = null;
                    postingState.subscription = null;
                    postingState.canceled = false;
             }
               /**如果其中的某个被调用的注册方法执行过程中调用cancelEventDelivery(Object event).那么就会阻止该事件的继续传递。*/
              if (aborted) { 
                    break;
                }
 。。。。。。。。。
 }

```
它还是把具体的post任务交付给了postToSubscription
` postToSubsription根据subscription.subscriberMethod.threadMode `来判定采用何种形式传递事件。并调用相应的xxxPoster类传递。
```

/**这个方法本身就是线程安全的。*/
   private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case PostThread:
              invokeSubscriber(subscription, event);//可以看到最后还是交给了它
                break;
            case MainThread:
                if (isMainThread) {
                  invokeSubscriber(subscription, event);//可以看到最后还是交给了它
              } else {
                    mainThreadPoster.enqueue(subscription, event);//稍后分析，其实最终也是交给了invokeSubscriber，只不过是在不同的线程上下文环境
              }
                break;
            case BackgroundThread:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);//稍后分析，其实最终也是交给了invokeSubscriber，只不过是在不同的线程上下文环境
              } else {
                  invokeSubscriber(subscription, event);//可以看到最后还是交给了它
              }
                break;
            case Async:
                asyncPoster.enqueue(subscription, event);//稍后分析，其实最终也是交给了invokeSubscriber，只不过是在不同的线程上下文环境。
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
      }
  }

```
` invokeSubscriber `执行具体的事件方法调用任务，通过反射调用。
```

/**这个方法也是线程安全的。所以我们无需担心*/
  void invokeSubscriber(Subscription subscription, Object event) {
        try {
          //通过反射进行调用
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
      } catch (InvocationTargetException e) {
          handleSubscriberException(subscription, event, e.getCause());
      } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
      }
    }
	/**线程安全，用于通过XxxPoster调用的情况。注意：这个方法是在非posting thread中调用的。*/
    void invokeSubscriber(PendingPost pendingPost) {
        Object event = pendingPost.event;
        Subscription subscription = pendingPost.subscription;
      //PendingPost内部维护着预Post链表关系，从预Post链表中删除这个post。
        PendingPost.releasePendingPost(pendingPost);
      /**由于取消注册时，虽然从subscriptionsByEventType删除了对应的subscription，但是并没有从PendingPost删除。所以在取消注册的时候给它设置了一个标志。这里用来判定。\
      *前面我们说过 volatile boolean active;所以可以确保可见性和简单赋值操作的一致性。那么为什么前面的invokeSubscriber(Subscription subscription, Object event)不采用这个标志进行判断？
      *那是由于通过PendingPost传递的都是与posting thread事件派发线程不相同的线程。这是为了在事件派发线程调用取消注册时，就没必要再进行事件具体调用了。而处于同一线程或前面判断过则没有这种竟态条件。
      *由衷地佩服作者的严谨。
      */
        if (subscription.active) {
            invokeSubscriber(subscription, event);
        }
    }



``` 
###  中断事件传递Post 
![](/data/dokuwiki/source/pasted/20150904-060836.png)
` cancelEventDelivery `应该被称为中断事件post，而不应该称为取消。因为其只能中断事件继续传递到某个对象后的其他对象，而不能取消正在传递到某个对象的事件。可以从前面` postSingleEventForEventType方法 `的分析中看出来。
```

	/**线程安全的，无需担心。*/
    public void cancelEventDelivery(Object event) {//注意传入的参数为事件对象，而非订阅者对象。这是很显然的。
        PostingThreadState postingState = currentPostingThreadState.get();
      /**咦！有没有发现问题，为什么会出现这一判定？PostingThreadState不是采用ThreadLocal模式吗？那么为什么还需要这一判定呢？
      *其实作者的逻辑是非常严谨的。设想当我们在posting thread的event handling method之外的调用这个方法。如果不设置这个标志，那么我们就会把 postingState.canceled = true;然后我们的下一个post就会无法发送。
      *从逻辑角度讲就是，都还没有事件被派送，你就取消了。没有意义。只有当事件被派送了取消才有意义。同时我们也只有在事件处理方法中调用cancelEventDelivery才有意义。
      *同时这里隐含了一个限制，就是只有在事件处理方法与事件派发线程处于同一线程的情况下 才能取消事件派发。
      *onEventAsync是无法取消的。onEventBackground在posting thread是主线程的情况下也是无法取消的。onEventMainThread在posting thread是非主线程环境下是无法被取消的。onEvent是绝对可以被取消的。
      */
        if (!postingState.isPosting) {
          //所以这里会抛出RuntimeException子类提示。
            throw new EventBusException(
                    "This method may only be called from inside event handling methods on the posting thread");
          //非空检查。
        } else if (event == null) {
            throw new EventBusException("Event may not be null");
          //正在派发的事件与传入的事件对象不一致。
        } else if (postingState.event != event) {
            throw new EventBusException("Only the currently handled event may be aborted");
         //这里作者做了限制，即onEvent()是可以取消的。作者非常严谨。
        } else if (postingState.subscription.subscriberMethod.threadMode != ThreadMode.PostThread) {
            throw new EventBusException(" event handlers may only abort the incoming event");
        }
	//最后设置状态。
        postingState.canceled = true;
    }
	/**作为一个框架，尽量抛出异常提示用户不要错误地进行方法调用。给我一个感慨用好异常才能写好框架。*/

```
###  判定是否已经注册 
```

	/**可以看到为了确保线程安全性，采取了同步方法。千万不要认为读取就不要保证同步了。
        *其实就是从typesBySubscriber这个map中进行查询来判定
        */
    public synchronized boolean isRegistered(Object subscriber) {
        return typesBySubscriber.containsKey(subscriber);
    }

```
###  其余方法 
```

	/**返回一个线程池用于onEventAsync和onEventBackground的情形*/
    ExecutorService getExecutorService() {
        return executorService;
    }

```

好，到这里已经分析了EventBus事件传递机制的总体逻辑。
接下来我们看一下传递的细节。
##  接着分析PendingPost和PendingPostQueue类 
###  PendingPost 
预Post的event与subscription封装池。
```

/**PendingPost名字很显然，待post的事件调用封装。事实上的线程安全类。
*不知你有没有注意，在这个库中，大部分类都是final的。这是尽量减少可变因素。
*而且类的可见性也控制得很好，看到没，是默认可见性，而非public。只是包可见性。因为客户端代码完全没有必要关心它。
*/
final class PendingPost {
  /**维护一个待post对象池。采用对象池技术，包含PendingPost池。这里采用的是强引用，如果采用弱引用池是否更好？*/
    private final static List<PendingPost> pendingPostPool = new ArrayList<PendingPost>();
    Object event; //事件对象
    Subscription subscription;//订阅关系
    PendingPost next;//用于与PendingPostQueue组合形成一个链表。

/**从PendingPost对象池中获取一个对象，线程安全的*/
 static PendingPost obtainPendingPost(Subscription subscription, Object event) {
   	/**进行同步，确保对象池一致性。如果池中含有对象*/
        synchronized (pendingPostPool) {
            int size = pendingPostPool.size();
            if (size > 0) {
                PendingPost pendingPost = pendingPostPool.remove(size - 1);
                pendingPost.event = event;
                pendingPost.subscription = subscription;
                pendingPost.next = null; 
                return pendingPost;
            }
        }
   	//如果池中没有对象，则直接返回创建一个新对象。
        return new PendingPost(event, subscription);
    }

/**释放PendingPost对象到对象池中。线程安全的*/
    static void releasePendingPost(PendingPost pendingPost) {
      //清空关联
        pendingPost.event = null;
        pendingPost.subscription = null;
        pendingPost.next = null;
      //这里同步以确保线程安全性
        synchronized (pendingPostPool) {
            // Don't let the pool grow indefinitely，对池大小作出限制大小为10000
            if (pendingPostPool.size() < 10000) {
                pendingPostPool.add(pendingPost);
            }
        }
    }

```
###  PendingPostQueue 
待Post封装队列
```

/**线程安全类*/
final class PendingPostQueue {
  /**这个队列采用链表队列形式进行维护，维护链表的头部和尾部*/
    private PendingPost head;
    private PendingPost tail;


/**入列，加入链表尾部，采用同步方法，内部通过wait-notifyAll机制形成生产者-消费者模式*/
  synchronized void enqueue(PendingPost pendingPost) {
        if (pendingPost == null) {
          //作为框架，参数检查失败抛出运行时异常，远比直接返回要好。
            throw new NullPointerException("null cannot be enqueued");
        }
        if (tail != null) {
            tail.next = pendingPost;
            tail = pendingPost;
        } else if (head == null) {
            head = tail = pendingPost;
        } else {
            throw new IllegalStateException("Head present, but no tail");
        }
    	//只要生产者将其放入队列，则通知消费者队列不为空。
        notifyAll();
    }
/**出列，抛出头部，非阻塞获取，立即返回，即使为空。同步方法。*/
    synchronized PendingPost poll() {
        PendingPost pendingPost = head;
        if (head != null) {
            head = head.next;
            if (head == null) {
                tail = null;
            }
        }
        return pendingPost;
    }
/**出列，超时阻塞获取，如果队列为空则超时等待，否则立即返回。同步方法。*/
    synchronized PendingPost poll(int maxMillisToWait) throws InterruptedException {
        if (head == null) {
            wait(maxMillisToWait);
        }
        return poll();
    }

}

```
##  三个Poster分析 
HandlerPoster、BackgroundPoster、AsyncPoster
AysncPoster实现Runnable接口
BackgroundPoster实现Runnable接口
HandlerPoster继承android.os.Handler
###  HandlerPoster 
主线程事件派发器
```

/**可以看到继承了Handler，采取android通用的主线程Handler通信机制。类似于Swing中的EventQueue.invokeLater()
*可以看到这也是一个final修饰的class
*/
final class HandlerPoster extends Handler {
	//维护一个预处理队列,由于PendingPostQueue是线程安全的，故而对这个final域的简单操作也是线程安全的。
    private final PendingPostQueue queue;
  //消息处理超时
    private final int maxMillisInsideHandleMessage;
  //与EventBus存在着双向引用的关系
    private final EventBus eventBus;
  //设置处理器是否可用。
    private boolean handlerActive;

    HandlerPoster(EventBus eventBus, Looper looper, int maxMillisInsideHandleMessage) {
        super(looper);
        this.eventBus = eventBus;
        this.maxMillisInsideHandleMessage = maxMillisInsideHandleMessage;
        queue = new PendingPostQueue();
    }
   /**入列处理。由于这里存在并发访问的情况，所以修改有同步机制*/
    void enqueue(Subscription subscription, Object event) {
      	//从PendingPost维护的对象池中获取一个对象。
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
	  /**这里采用同步机制，虽然我们可以保证queue的所有操作是线程安全的
     	  *但是这远远不够。我们还需要保证其一是保证在调用在这段入列，标志设置的过程中，阻塞handleMessage继续派发。其二是因为确保handlerActive可见性和对handlerActive的访问存在竞态条件
      	  */
        synchronized (this) {
            queue.enqueue(pendingPost);
          //入列的时候同时设置激活状态
            if (!handlerActive) {
                handlerActive = true;
              //如果传递失败
                if (!sendMessage(obtainMessage())) {
                    throw new EventBusException("Could not send handler message");
                }
            }
        }
    }
	/**消息处理,在主线程被调用。*/
    @Override
    public void handleMessage(Message msg) {
        boolean rescheduled = false;
        try {
            long started = SystemClock.uptimeMillis();
          /**由于只有一条线程处理所有事件的派发，所以需要循环队列进行派发，这一点与AsyncPoster不同*/
            while (true) {
              //以下采用线程安全的双重判定。
                PendingPost pendingPost = queue.poll();
                if (pendingPost == null) {
                    /**虽然我们可以看到queue的所有操作都是同步方法，对queue的操作本身是线程安全的，但是我们无法保证在这个poster对象中queue.enqueue在前面判断之后是否会被调用。
                      *所以我们需要再次同步，通过对这个poster对象进行同步以确保在此期间不会有enqueue的干扰，能够判断queue真正的为null。
                      */
                    synchronized (this) {
                        // Check again, this time in synchronized
                        pendingPost = queue.poll();
                        if (pendingPost == null) {
                            handlerActive = false;
                            return;
                        }
                    }
                }
              //真正进行方法调用处理。
                eventBus.invokeSubscriber(pendingPost);
                long timeInMethod = SystemClock.uptimeMillis() - started;
              //如果处理超时。
                if (timeInMethod >= maxMillisInsideHandleMessage) {
                    if (!sendMessage(obtainMessage())) {
                        throw new EventBusException("Could not send handler message");
                    }
                    rescheduled = true;
                    return;
                }
            }
        } finally {
            handlerActive = rescheduled;
        }
    }
}

```
###  AsyncPoster 
异步分派器，AysncPoster它通过线程池执行注册方法。每次分派都会通过不同线程调用。
```

/**实现Runnable接口*/
class AsyncPoster implements Runnable {
	//内部维护着分派队列，都是final，其一保证可见性，其二最小化可变性
    private final PendingPostQueue queue;
    private final EventBus eventBus;
	/**构造器的可访问性也是很有讲究，并没有public修饰，而是采用默认的包可访问性。因为客户端代码无需了解这些东西。*/
    AsyncPoster(EventBus eventBus) {
        this.eventBus = eventBus;
        queue = new PendingPostQueue();
    }
    public void enqueue(Subscription subscription, Object event) {
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        queue.enqueue(pendingPost);
      //每入列一个事件则通过线程池派发一个事件调用。
        eventBus.getExecutorService().execute(this);
    }

    @Override
    public void run() {
      /**因为queue的所有操作都是线程安全的，所以这里无需担心这个问题。
      *每个事件分派一个线程，所以这里无需进行循环。与HandlerPoster是有区别的。
      *这里之所以没有像其他两个Poster一样采取同步块，是因为队列是否为空，对这个poster的派发本身没有影响。因为每个事件都会得到一条单独的线程进行派发。
      */
        PendingPost pendingPost = queue.poll();
        if(pendingPost == null) {
            throw new IllegalStateException("No pending post available");
        }
        eventBus.invokeSubscriber(pendingPost);
    }

}

```
###  BackgroundPoster 
后台派发器：只是维持一个后台线程进行事件派发。
```

final class BackgroundPoster implements Runnable {

    private final PendingPostQueue queue;
    private final EventBus eventBus;
	/**虽然通过线程池进行，但是通过这个标志保证只有一个线程在执行任务。采用volatile确保可见性*/
    private volatile boolean executorRunning;

    BackgroundPoster(EventBus eventBus) {
        this.eventBus = eventBus;
        queue = new PendingPostQueue();
    }

    public void enqueue(Subscription subscription, Object event) {
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
      /**这里采用同步机制，虽然我们可以保证queue的所有操作是线程安全的，也可以保证对executorRunning这个boolean的简单赋值操作是可见的。
      *但是这远远不够。我们还需要保证其一是保证在调用在这段入列，标志设置的过程中，阻塞run()继续派发。因为其二是因为对executorRunning的访问存在竞态条件
      */
        synchronized (this) {
            queue.enqueue(pendingPost);
          //通过这个标志确保只有一个线程在执行任务。
            if (!executorRunning) {
                executorRunning = true;
                eventBus.getExecutorService().execute(this);
            }
        }
    }

    @Override
    public void run() {
        try {
            try {
              //无限循环队列派发事件。
                while (true) {
                    PendingPost pendingPost = queue.poll(1000);//采取超时poll。
                    if (pendingPost == null) {
                      /**虽然我们可以看到queue的所有操作都是同步方法，对queue的操作本身是线程安全的，但是我们无法保证在这个poster对象中queue.enqueue在前面判断之后是否会被调用。
                      *所以我们需要再次同步，通过对这个poster对象进行同步以确保在此期间不会有enqueue的干扰，能够判断queue真正的为null。
                      */
                        synchronized (this) {
                            // Check again, this time in synchronized
                            pendingPost = queue.poll();
                            if (pendingPost == null) {
                                executorRunning = false;
                                return;
                            }
                        }
                    }
                    eventBus.invokeSubscriber(pendingPost);
                }
            } catch (InterruptedException e) {
                Log.w("Event", Thread.currentThread().getName() + " was interruppted", e);
            }
        } finally {
          //注意最后需要在finally块中恢复状态。
            executorRunning = false;
        }
    }


```
  
##  异常机制分析 
###  EventBusException 
```

/**
 * An {@link RuntimeException} thrown in cases something went wrong inside EventBus.
 *可以看到采用的是运行时异常，非受捡。一般开源库都会有一套自己的异常处理机制。
 */
public class EventBusException extends RuntimeException {

    private static final long serialVersionUID = -2912559384646531479L;
	/**一般自定义异常需要覆盖重载的这三个构造器*/
    public EventBusException(String detailMessage) {
        super(detailMessage);
    }

    public EventBusException(Throwable throwable) {
        super(throwable);
    }

    public EventBusException(String detailMessage, Throwable throwable) {
        super(detailMessage, throwable);
    }

}

```
