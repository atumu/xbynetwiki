title: rxjava学习21 

#  RxJava学习之RxJava编程实践 
##  流式编程与封装性 
因为操作符可以让你对数据流做任何操作。
将一系列的操作符链接起来就可以完成复杂的逻辑。**代码被分解成一系列可以组合的片段**。这就是**响应式函数编程的魅力**。用的越多，就会越多的改变你的编程思维。
```

query("Hello, world!")  
    .flatMap(urls -> Observable.from(urls))  
    .flatMap(url -> getTitle(url))  
    .filter(title -> title != null)  
    .take(5)  
    .doOnNext(title -> saveTitle(title))  
    .subscribe(title -> System.out.println(title)); 

```
另外，RxJava也使我们处理数据的方式变得更简单。在最后一个例子里，我们调用了两个API，对API返回的数据进行了处理，然后保存到磁盘。**但是我们的Subscriber并不知道这些，它只是认为自己在接收一个Observable<String>对象。良好的封装性也带来了编码的便利！**
##  关于异常和错误 
1.只要有异常发生onError()一定会被调用：这极大的简化了错误处理。只需要在一个地方处理错误即可以。
2.操作符不需要处理异常：**将异常处理交给订阅者来做，**Observerable的操作符调用链中一旦有一个抛出了异常，就会直接执行onError()方法。
3.你能够知道什么时候订阅者已经接收了全部的数据。知道什么时候任务结束能够帮助简化代码的流程。（虽然有可能Observable对象永远不会结束）
我觉得这种错误处理方式比传统的错误处理更简单。传统的错误处理中，通常是在每个回调中处理错误。这不仅导致了重复的代码，并且意味着每个回调都必须知道如何处理错误，你的回调代码将和调用者紧耦合在一起。
**使用RxJava，Observable对象根本不需要知道如何处理错误！操作符也不需要处理错误状态-一旦发生错误，就会跳过当前和后续的操作符。所有的错误处理都交给订阅者来做。**
当然我也不是所有的代码都使用响应式的方式–**仅仅当代码复杂到我想将它分解成简单的逻辑的时候，我才使用响应式代码。**
##  关于线程调度 
假设你编写的Android app需要从网络请求数据（感觉这是必备的了，还有单机么？）。网络请求需要花费较长的时间，因此你打算在另外一个线程中加载数据。那么问题来了！
编写多线程的Android应用程序是很难的，因为你必须确保代码在正确的线程中运行，否则的话可能会导致app崩溃。最常见的就是在非主线程更新UI。
**使用RxJava，你可以使用` subscribeOn()指定观察者(Observable)代码运行的线程 `，使用` observerOn()指定订阅者(Subcriber)运行的线程 `：**
```

myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));

```
是不是很简单？任何在我的Subscriber前面执行的代码都是在I/O线程中运行。最后，操作view的代码在主线程中运行.
最棒的是我**可以把subscribeOn()和observerOn()添加到任何Observable对象上。这两个也是操作符！。我不需要关心Observable对象以及它上面有哪些操作符。仅仅运用这两个操作符就可以实现在不同的线程中调度。**
如果使用AsyncTask或者其他类似的，我将不得不仔细设计我的代码，找出需要并发执行的部分。**使用RxJava，我可以保持代码不变，仅仅在需要并发的时候调用这两个操作符就可以。**

##  关于订阅（Subscriptions） 
当调用Observable.subscribe()，会返回一个` Subscription `对象。这个对象代表了被观察者和订阅者之间的联系。
```

Subscription subscription = Observable.just("Hello, World!")
    .subscribe(s -> System.out.println(s));

```
你可以在后面使用这个Subscription对象来操作被观察者和订阅者之间的联系.
```

subscription.unsubscribe();
System.out.println("Unsubscribed=" + subscription.isUnsubscribed());
// Outputs "Unsubscribed=true"

```
**RxJava的另外一个好处就是它处理unsubscribing的时候，会停止整个调用链。如果你使用了一串很复杂的操作符，调用unsubscribe将会在他当前执行的地方终止。不需要做任何额外的工作！**


