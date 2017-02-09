title: introduce 

#  RxJava与RxAndroid基于异步事件与观察者序列地响应式编程库 
系列文章地址:http://blog.csdn.net/lzyzsd/article/details/41833541
##  概述 
RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences.
RxJava特点：
  * Zero Dependencies
  * < 800KB Jar
  * Java 6+ & Android 2.3+
  * Java 8 lambda support
  * Polyglot (Scala, Groovy, Clojure and Kotlin)
  * Non-opinionated about source of concurrency (threads, pools, event loops, fibers, actors, etc)
  * Async or synchronous execution
  * Virtual time and schedulers for parameterized concurrency
RxJava最核心的两个东西是Observables（被观察者，事件源）和Subscribers（观察者）。Observables发出一系列事件，Subscribers处理这些事件。这里的事件可以是任何你感兴趣的东西（触摸事件，web接口调用返回的数据。。。）
一个Observable可以发出零个或者多个事件，知道结束或者出错。每发出一个事件，就会调用它的Subscriber的onNext方法，最后调用Subscriber.onNext()或者Subscriber.onError()结束。
Rxjava的看起来很想设计模式中的观察者模式，但是有一点明显不同，那就是如果一个Observerble没有任何的的Subscriber，那么这个Observable是不会发出任何事件的。
###  Hello World 
```

//创建一个Observable对象很简单，直接调用Observable.create即可
Observable<String> myObservable = Observable.create(  
    new Observable.OnSubscribe<String>() {  
        @Override  
        public void call(Subscriber<? super String> sub) {  
            sub.onNext("Hello, world!");  
            sub.onCompleted();  
        }  
    }  
); 
//创建一个Subscriber来处理Observable对象发出的字符串。
Subscriber<String> mySubscriber = new Subscriber<String>() {  
    @Override  
    public void onNext(String s) { System.out.println(s); }  
  
    @Override  
    public void onCompleted() { }  
  
    @Override  
    public void onError(Throwable e) { }  
}; 
//通过subscribe函数就可以将我们定义的myObservable对象和mySubscriber对象关联起来，这样就完成了subscriber对observable的订阅
myObservable.subscribe(mySubscriber);  

```
###  更简洁的代码 
RxJava其实提供了很多便捷的函数来帮助我们减少代码。
首先来看看如何简化Observable对象的创建过程。RxJava内置了很多简化创建Observable对象的函数，比如Observable.just就是用来创建只发出一个事件就结束的Observable对象，上面创建Observable对象的代码可以简化为一行
```

Observable<String> myObservable = Observable.just("Hello, world!"); 

``` 
接下来看看如何简化Subscriber，上面的例子中，我们其实并不关心OnComplete和OnError，我们只需要在onNext的时候做一些处理，这时候就可以使用Action1类。
```

Action1<String> onNextAction = new Action1<String>() {  
    @Override  
    public void call(String s) {  
        System.out.println(s);  
    }  
}; 

```
subscribe方法有一个重载版本，接受三个Action1类型的参数，分别对应OnNext，OnComplete， OnError函数。
```

myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);  

```
这里我们并不关心onError和onComplete，所以只需要第一个参数就可以
```

myObservable.subscribe(onNextAction);

```
上面的代码最终可以写成这样
```

Observable.just("Hello, world!")  
    .subscribe(new Action1<String>() {  
        @Override  
        public void call(String s) {  
              System.out.println(s);  
        }  
    });

```
使用java8的lambda可以使代码更简洁
```

Observable.just("Hello, world!")  
    .subscribe(s -> System.out.println(s)); 

``` 
Android开发中，强烈推荐使用retrolambda这个gradle插件，这样你就可以在你的代码中使用lambda了。
###  变换与操作符 
根据响应式函数编程的概念，Subscribers更应该做的事情是“响应”，响应Observable发出的事件，而不是去修改。如果我能在某些中间步骤中对“Hello World！”进行变换是不是很酷？
操作符就是为了解决对Observable对象的变换的问题，操作符用于在Observable和最终的Subscriber之间修改Observable发出的事件。RxJava提供了很多很有用的操作符。
**比如map操作符，就是用来把把一个事件转换为另一个事件的**。
```

Observable.just("Hello, world!")  
  .map(new Func1<String, String>() {  
      @Override  
      public String call(String s) {  
          return s + " -Dan";  
      }  
  })  
  .subscribe(s -> System.out.println(s)); 

```
是不是很酷？map()操作符就是用于变换Observable对象的，map操作符返回一个Observable对象，这样就可以实现**链式调用**，在一个Observable对象上多次使用map操作符，最终将最简洁的数据传递给Subscriber对象。
map操作符更有趣的一点是**它不必返回Observable对象返回的类型**，你可以使用map操作符返回一个发出新的数据类型的observable对象。
比如上面的例子中，subscriber并不关心返回的字符串，而是想要字符串的hash值
```

Observable.just("Hello, world!")  
    .map(new Func1<String, Integer>() {  
        @Override  
        public Integer call(String s) {  
            return s.hashCode();  
        }  
    })  
    .subscribe(i -> System.out.println(Integer.toString(i)));  

```
很有趣吧？我们初始的Observable返回的是字符串，最终的Subscriber收到的却是Integer，当然使用lambda可以进一步简化代码：
```

Observable.just("Hello, world!")  
    .map(s -> s.hashCode())  
    .subscribe(i -> System.out.println(Integer.toString(i)));  

```
前面说过，**Subscriber做的事情越少越好**，我们再增加一个map操作符
```

Observable.just("Hello, world!")  
    .map(s -> s.hashCode())  
    .map(i -> Integer.toString(i))  
    .subscribe(s -> System.out.println(s)); 

```
###  不服？ 

是不是觉得我们的例子太简单，不足以说服你？你需要明白下面的两点:
**1.Observable和Subscriber可以做任何事情**
Observable可以是一个数据库查询，Subscriber用来显示查询结果；Observable可以是屏幕上的点击事件，Subscriber用来响应点击事件；Observable可以是一个网络请求，Subscriber用来显示请求结果。
**2.Observable和Subscriber是独立于中间的变换过程的。**
在Observable和Subscriber**中间可以增减任何数量的map**。整个系统是**高度可组合的**，**操作数据是一个很简单的过程。**

##  RxJava操作符 
RxJava的强大性就来自于它所定义的操作符
略。。。
##  RxJava响应式编程 
###  错误处理 
到目前为止，我们都没怎么介绍onComplete()和onError()函数。这两个函数用来通知订阅者，被观察的对象将停止发送数据以及为什么停止（成功的完成或者出错了）。
```

Observable.just("Hello, world!")
    .map(s -> potentialException(s))
    .map(s -> anotherPotentialException(s))
    .subscribe(new Subscriber<String>() {
        @Override
        public void onNext(String s) { System.out.println(s); }

        @Override
        public void onCompleted() { System.out.println("Completed!"); }

        @Override
        public void onError(Throwable e) { System.out.println("Ouch!"); }
    });

```
###  调度器 
假设你编写的Android app需要从网络请求数据（感觉这是必备的了，还有单机么？）。网络请求需要话费较长的时间，因此你打算在另外一个线程中加载数据。为问题来了！
编写多线程的Android应用程序是很难的，因为你必须确保代码在正确的线程中运行，否则的话可能会导致app崩溃。最常见的就是在非主线程更新UI。
**使用RxJava，你可以使用` subscribeOn() `指定` 观察者 `代码运行的线程，使用` observerOn() `指定` 订阅者 `运行的线程：**
```

myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));

```
是不是很简单？任何在我的Subscriber前面执行的代码都是在I/O线程中运行。最后，操作view的代码在主线程中运行.
最棒的是我可以把subscribeOn()和observerOn()添加到任何Observable对象上。这两个也是操作符！。我不需要关心Observable对象以及它上面有哪些操作符。仅仅运用这两个操作符就可以实现在不同的线程中调度。
如果使用AsyncTask或者其他类似的，我将不得不仔细设计我的代码，找出需要并发执行的部分。使用RxJava，我可以保持代码不变，仅仅在需要并发的时候调用这两个操作符就可以。
##  订阅关系（Subscriptions） 
当调用Observable.subscribe()，会返回一个**Subscription**对象。这个对象代表了被观察者和订阅者之间的联系。
你可以在后面使用这个Subscription对象来操作被观察者和订阅者之间的联系.
```

subscription.unsubscribe();
System.out.println("Unsubscribed=" + subscription.isUnsubscribed());
// Outputs "Unsubscribed=true"

```
RxJava的另外一个好处就是它处理unsubscribing的时候，会停止整个调用链。如果你使用了一串很复杂的操作符，调用unsubscribe将会在他当前执行的地方终止。不需要做任何额外的工作！
##  RxJava在Android中使用响应式编程 
RxAndroid是RxJava的一个针对Android平台的扩展。它包含了一些能够简化Android开发的工具。
项目地址：https://github.com/ReactiveX/RxAndroid
首先，AndroidSchedulers提供了针对Android的线程系统的调度器。需要在UI线程中运行某些代码？很简单，只需要使用AndroidSchedulers.mainThread():
```

retrofitService.getImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));

```
如果你已经创建了自己的Handler，你可以使用HandlerThreadScheduler1将一个调度器链接到你的handler上。
接着要介绍的就是AndroidObservable，它提供了跟多的功能来配合Android的生命周期。bindActivity()和bindFragment()方法默认使用AndroidSchedulers.mainThread()来执行观察者代码，这两个方法会在Activity或者Fragment结束的时候通知被观察者停止发出新的消息。
```

AndroidObservable.bindActivity(this, retrofitService.getImage(url))
    .subscribeOn(Schedulers.io())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap);

```
我自己也很喜欢AndroidObservable.fromBroadcast()方法，它允许你创建一个类似BroadcastReceiver的Observable对象。下面的例子展示了如何在网络变化的时候被通知到：
```

IntentFilter filter = new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION);
AndroidObservable.fromBroadcast(context, filter)
    .subscribe(intent -> handleConnectivityChange(intent));

```
最后要介绍的是ViewObservable,使用它可以给View添加了一些绑定。如果你想在每次点击view的时候都收到一个事件，可以使用ViewObservable.clicks()，或者你想监听TextView的内容变化，可以使用ViewObservable.text()。
```

ViewObservable.clicks(mCardNameEditText, false)
    .subscribe(view -> handleClick(view));

```
###  Retrofit对RxJava的支持 
大名鼎鼎的Retrofit库内置了对RxJava的支持。通常调用发可以通过使用一个Callback对象来获取异步的结果：
```

@GET("/user/{id}/photo")
void getUserPhoto(@Path("id") int id, Callback<Photo> cb);

```
使用RxJava，你可以直接返回一个Observable对象。
```

@GET("/user/{id}/photo")
Observable<Photo> getUserPhoto(@Path("id") int id);

```
现在你可以随意使用Observable对象了。你不仅可以获取数据，还可以进行变换。 
**Retrofit对Observable的支持使得它可以很简单的将多个REST请求结合起来**。比如我们有一个请求是获取照片的，还有一个请求是获取元数据的，我们就可以将这两个请求并发的发出，并且等待两个结果都返回之后再做处理：
```

Observable.zip(
    service.getUserPhoto(id),
    service.getPhotoMetadata(id),
    (photo, metadata) -> createPhotoWithData(photo, metadata))
    .subscribe(photoWithData -> showPhoto(photoWithData));

```
###  遗留代码，运行极慢的代码 
绝大多数时候Observable.just() 和 Observable.from() 能够帮助你从遗留代码中创建 Observable 对象:
###  生命周期 
我把最难的不分留在了最后。如何处理Activity的生命周期？主要就是两个问题： 
1.在configuration改变（比如转屏）之后继续之前的Subscription。
比如你使用Retrofit发出了一个REST请求，接着想在listview中展示结果。如果在网络请求的时候用户旋转了屏幕怎么办？你当然想继续刚才的请求，但是怎么搞？
2.Observable持有Context导致的内存泄露
这个问题是因为创建subscription的时候，以某种方式持有了context的引用，尤其是当你和view交互的时候，这太容易发生！如果Observable没有及时结束，内存占用就会越来越大。 
不幸的是，没有银弹来解决这两个问题，但是这里有一些指导方案你可以参考。

第一个问题的解决方案就是使用RxJava内置的缓存机制，这样你就可以对同一个Observable对象执行unsubscribe/resubscribe，却不用重复运行得到Observable的代码。cache() (或者 replay())会继续执行网络请求（甚至你调用了unsubscribe也不会停止）。这就是说你可以在Activity重新创建的时候从cache()的返回值中创建一个新的Observable对象。
```

Observable<Photo> request = service.getUserPhoto(id).cache();
Subscription sub = request.subscribe(photo -> handleUserPhoto(photo));

// ...When the Activity is being recreated...
sub.unsubscribe();

// ...Once the Activity is recreated...
request.subscribe(photo -> handleUserPhoto(photo));

```
**注意，两次sub是使用的同一个缓存的请求**。当然在哪里去存储请求的结果还是要你自己来做，和所有其他的生命周期相关的解决方案一延虎，必须在生命周期外的某个地方存储。（retained fragment或者单例等等）。
第二个问题的解决方案就是在生命周期的某个时刻取消订阅。**一个很常见的模式就是使用CompositeSubscription来持有所有的Subscriptions，然后在onDestroy()或者onDestroyView()里取消所有的订阅**。
```

private CompositeSubscription mCompositeSubscription
    = new CompositeSubscription();

private void doSomething() {
    mCompositeSubscription.add(
        AndroidObservable.bindActivity(this, Observable.just("Hello, World!"))
        .subscribe(s -> System.out.println(s)));
}
@Override
protected void onDestroy() {
    super.onDestroy();

    mCompositeSubscription.unsubscribe();
}

```
**你可以在Activity/Fragment的基类里创建一个CompositeSubscription对象，在子类中使用它。**
**注意! 一旦你调用了 CompositeSubscription.unsubscribe()，这个CompositeSubscription对象就不可用了**, 如果你还想使用CompositeSubscription，就必须在创建一个新的对象了。
