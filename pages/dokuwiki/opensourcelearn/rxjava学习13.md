title: rxjava学习13 

#  RxJava学习之操作符学习十异步操作 
下面的这些操作符属于**单独的rxjava-async模块**，它们**用于将同步对象转换为Observable。**
  * start( ) — 创建一个Observable，它发射一个函数的返回值
  * toAsync( ) or asyncAction( ) or asyncFunc( ) — 将一个函数或者Action转换为已Observable，它执行这个函数并发射函数的返回值
  * startFuture( ) — 将一个返回Future的函数转换为一个Observable，它发射Future的返回值
  * deferFuture( ) — 将一个返回Observable的Future转换为一个Observable，但是并不尝试获取这个Future返回的Observable，直到有订阅者订阅它
  * forEachFuture( ) — 传递Subscriber方法给一个Subscriber，但是同时表现得像一个Future一样阻塞直到它完成
  * fromAction( ) — 将一个Action转换为Observable，当一个订阅者订阅时，它执行这个action并发射它的返回值
  * fromCallable( ) — 将一个Callable转换为Observable，当一个订阅者订阅时，它执行这个Callable并发射Callable的返回值，或者发射异常
  * fromRunnable( ) — convert a Runnable into an Observable that invokes the runable and emits its result when a Subscriber subscribes将一个Runnable转换为Observable，当一个订阅者订阅时，它执行这个Runnable并发射Runnable的返回值
  * runAsync( ) — 返回一个StoppableObservable，它发射某个Scheduler上指定的Action生成的多个actions
具体介绍请参考：http://wiki.xby1993.net/doku.php?id=opensourcelearn:rxjava%E5%AD%A6%E4%B9%A05&#async_operators