title: android_eventbus 

#  Android组件间通信框架EventBus 

#  事件总线EventBus模式概述 

在不使用事件总线的情况下：
在应用中的多个地方，控件经常需要根据某个状态来更新他们显示的内容。这种场景常见的解决方式就是定义一个接口，需要关注该事件的控件来实现这个接口。然后事件触发的地方来注册/取消注册这些对该事件感兴趣的控件。例如，陌陌依赖手机位置信息来获取附近的用户，所以在位置更新管理器(MmLocationManager)中定义了一个接口来监听位置更新的事件(MmLocationListener):
```

interface MmLocationListener {  
 void onLocationChanged(Location location);  
}  

```
然后在应用的各个需要响应该事件的地方来实现上面的接口，然后在位置更新管理器(MmLocationManager)中注册/取消注册事件监听接口的实现类：  
```

mLocationManager.get().register(this);  

```
上面的解决方案是没问题的，但是不是理想方案。每个控件实现这个接口，导致这些控件和位置管理器注册强耦合在一起。这还意味着，当单元测试的时候，您需要模拟(mocked)位置管理器来生成位置更新事件。
随着应用功能的增加，需要监听的事件越来越多，而越来越多的控件需要监听不同的事件，则导致越来越多的控件需要注册到各种事件管理器上：
```

// 代码开始变得无法控制…  
mLocationManager.get().register(this);  
userAuthenticator.get().register(this);  
settingsManager.get().register(this);  
syncManager.get().register(this);  
configurationMonitor.get().register(this);  

```
注册和取消注册这些事件慢慢的会变得越来越难以管理。导致测试越来越困难，并将导致开发者的效率越来越低，同时在您的应用中越来越容易引入各种奇怪的Bug。
**Event Bus模式 — 也被称为Message Bus或者发布者/订阅者(publisher/subscriber)模式 — 可以让两个组件相互通信，但是他们之间并不相互知晓。**
和需要注册各个事件的监听器相比，一个组件现在只用在Event Bus上注册一次即可：
```

bus.register(this);  

```
上面的注册告诉Event Bus我们现在希望接收各个事件的更新。 然后Bus检测该类中每个带有@Subscribe注解的函数，当相关的事件发生的时候就调用这些带有注解的函数。
**上面示例中的位置监听功能，不用实现位置监听接口和里面的函数了，只需要提供一个带有@Subscribe注解的函数即可：**
```

@Subscribe  
public void locationChanged(LocationChangedEvent event) {  
   // TODO React to location change.  
}  

```
现在Event Bus会把所有的LocationChangedEvent 事件都发送给上面的函数。**现在 MmLocationManager 类不用注册监听器了，当位置改变的时候 只需要向Event Bus发布事件即可：**
```

bus.post(new LocationChangedEvent(37.892818, -121.772608));  

```
**这样 组件间相互解耦了，而单元测试也变得简单了。任何事件都可以发布给Event Bus，然后Event Bus会找到对该事件感兴趣的函数来调用。**
注意：您也许已经发现该模式在Android上层也存在 — Intent系统就是这样设计的！

可以发现使用EventBus模式之后，我们简化了几个工作：
  * 不用再编写众多监听器接口，直接使用注解到方法。
  * 不用再编写监听器注册/取消注册的管理类，取而代之的是EventBus这个同意的事件总线。
  * 不用再对事件进行注册或者取消注册。事件总线会在事件被发布之后调用所有对它感兴趣的方法。
  * 通过事件总线这个中介通信的双方实现了解耦。
总之可以这么说：事件总线模式就是` 针对事件提供统一订阅，发布以达到组件间通信的解决方案。 `
![](/data/dokuwiki/opensourcelearn/pasted/20150511-080527.png?800x300)
<WRAP center round tip 60%>
订阅者可以订阅多个事件，发送者可以发布任何事件，发布者同时也可以是订阅者。
</WRAP>

##  Android中的两个EventBus模式的框架： 

  * Otto是Square公司在他们应用中使用的Event Bus实现。从Guava中演变而来，并且专注于Android平台。
  * EventBus 是greenrobot 出品的另外一个Event Bus类库，功能稍微多一点。本文将以EventBus类库为例进行学习。
单从使用上看，**EventBus > Otto > BroadcastReceiver**(当然BroadcastReceiver作为系统内置组件，有一些前两者没有的功能).
EventBus最简洁，Otto最符合Guava EventBus的设计思路， BroadcastReceiver最难使用。第一选择是EventBus。
#  Android EventBus类库介绍 

##  安装： 
```

Gradle:
compile 'de.greenrobot:eventbus:2.4.0'  
Maven:
<dependency>  
    <groupId>de.greenrobot</groupId>  
    <artifactId>eventbus</artifactId>  
    <version>2.4.0</version>  
</dependency>  

```
EventBus是GreenRobot出品的Android系统的一个Event Bus类库（其他的有Square的Otto类似），用来简化应用组件之间的通信。
当一个Android应用功能越来越多的时候，保证应用的各个部分之间高效的通信将变得越来越困难。
**EventBus**是一个Android端优化的publish/subscribe消息总线，简化了应用程序内各组件间、组件与后台线程间的通信。比如请求网络，等网络返回时通过Handler或Broadcast通知UI，两个Fragment之间需要通过Listener通信，这些需求都可以通过**EventBus**实现。
作为一个消息总线，有三个主要的元素：
  * Event：事件
  * Subscriber：事件订阅者，接收特定的事件
  * Publisher:事件发布者，用于通知Subscriber有事件发生
##  EventBus特点： 

**NOT based on annotations and Based on conventions**:事件订阅函数不是基于注解(Annotation)的，而是基于命名约定的，在Android 中(尤其是4.0之前),注解解析起来比较慢。 在EventBus中，使用约定来指定事件订阅者以简化使用。即所有事件订阅都都是以onEvent开头的函数，具体来说，函数的名字是` onEvent，onEventMainThread，onEventBackgroundThread，onEventAsync `这四个，这个和ThreadMode有关.
可以在任意线程任意位置发送事件，直接调用EventBus的`post(Object)`方法，可以自己实例化EventBus对象，但一般使用默认的单例就好了：`EventBus.getDefault()`，根据post函数参数的类型，会自动调用订阅相应类型事件的函数。

##  ThreadMode： 

前面说了，Subscriber函数的名字只能是那4个，因为每个事件订阅函数都是和一个`ThreadMode`相关联的，ThreadMode指定了会调用的函数。有以下四个ThreadMode：
  * PostThread：事件的处理在和事件的发送在相同的进程。对应的函数名是` onEvent `。
  * MainThread: 事件的处理会在UI线程中执行。对应的函数名是` onEventMainThread `。
  * BackgroundThread：事件的处理会在一个后台线程中执行，对应的函数名是` onEventBackgroundThread `。如果线程是后台线程，会直接执行事件，如果当前线程是UI线程，事件会被加到一个队列中，由一个线程依次处理这些事件，如果某个事件处理时间太长，会阻塞后面的事件的派发或处理。
<WRAP center round tip 60%>
` 上面的3种事件响应函数，应该能够很快的执行完，不然的话会阻塞各自的事件发布 `。
</WRAP>

  * Async：对应的函数名为` onEventAsync `事件处理会在单独的线程中执行，主要用于在后台线程中执行耗时操作，每个事件会开启一个线程（有线程池），但最好限制线程的数目。该线程和发布线程、主线程相互独立。如果事件响应函数需要较长的时间来执行，则应该使用该模式，例如 网络访问等。

根据事件订阅都函数名称的不同，会使用不同的ThreadMode，比如果在后台线程加载了数据想在UI线程显示，订阅者只需把函数命名为onEventMainThread。
**总结一下ThreadMode工作原理：**
  * 如果是PostThread，直接执行
  * 如果是MainThread，判断当前线程，如果本来就是UI线程就直接执行，否则加入`mainThreadPoster`队列
  * 如果是后台线程，如果当前是UI线程，加入`backgroundPoster`队列，否则直接执行
  * 如果是Async，加入`asyncPoster`队列

##  基本的使用步骤： 

  * 定义事件类型：public class MyEvent {}`
  * 定义事件处理方法：public void onEventMainThread`
  * 注册订阅者：EventBus.getDefault().register(this)` `注意一旦注册了订阅者代码中必须要有至少一个onEvent()方法，否则会报错 `。
  * 发送事件：EventBus.getDefault().post(new MyEvent())`
###  StickEvent: 

有时候某个事件可能会用到多次，比如在前面介绍Event Bus模型一文的示例中，最新的位置更新信息，可能需要多次用到，真对这种情况，您可以把该事件发布为Sticky Event，然后，当需要查询该信息的时候，可以通过Bus的getStickyEvent(ClasseventType) 函数来查询最新发布的Event对象。
同一类型的事件只保存最新的Event对象。注册和发布事件的函数分别为 registerSticky(…) 和 postSticky(Object event).


参考：
http://blog.chengyunfeng.com/?p=449
http://blog.csdn.net/djun100/article/details/23762621
https://github.com/greenrobot/EventBus/blob/V2.4.0/HOWTO.md
http://www.cnblogs.com/qianxudetianxia/p/4216949.html
http://www.cnblogs.com/angeldevil/p/3715934.html