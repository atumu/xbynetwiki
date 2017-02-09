title: rxjava学习18 

#  RxJava学习之自定义插件 
插件让你可以用多种方式修改RxJava的默认行为：
  * 修改默认的计算、IO和新线程调度器集合
  * 为RxJava可能遇到的特殊错误注册一个**错误处理器**
  * 注册一个函数**记录**一些常规RxJava活动的发生

##  RxJavaSchedulersHook 
这个插件让你可以使用你选择的调度器**覆盖默认的计算、IO和新线程调度 (Scheduler**)，要做到这些，需要继承 ` RxJavaSchedulersHook ` 类并覆写这些方法：
  * Scheduler getComputationScheduler( )
  * Scheduler getIOScheduler( )
  * Scheduler getNewThreadScheduler( )
  * Action0 onSchedule(action)
然后是下面这些步骤：
  * 创建一个你实现的 RxJavaSchedulersHook 子类的对象。
  * 使用 RxJavaPlugins.getInstance( ) 获取全局的RxJavaPlugins对象。
  * 将你的默认调度器对象传递给 RxJavaPlugins 的 registerSchedulersHook( ) 方法。
  * 完成这些后，RxJava会开始使用你的方法返回的调度器，而不是内置的默认调度器。
##  RxJavaErrorHandler 
这个插件让你可以注册一个函数处理传递给 Subscriber.onError(Throwable) 的错误。要做到这一点，需要继承`  RxJavaErrorHandler ` 类并覆写这个方法：
void handleError(Throwable e)
然后是下面这些步骤：
  * 创建一个你实现的 RxJavaErrorHandler 子类的对象。
  * 使用 ` RxJavaPlugins.getInstance( ) 获取全局的RxJavaPlugins对象 `。
  * 将你的错误处理器对象传递给`  RxJavaPlugins 的 registerErrorHandler( ) ` 方法。
  * 完成这些后，RxJava会开始使用你的错误处理器处理传递给 Subscriber.onError(Throwable) 的错误。
##  RxJavaObservableExecutionHook 
这个插件让你可以**注册一个函数用于记录日志或者性能数据收集**，RxJava在某些常规活动时会调用它。要做到这一点，需要继承 ` RxJavaObservableExecutionHook ` 类并覆写这些方法：
方法	何时调用
  * onCreate( )	在 Observable.create( )方法中
  * onSubscribeStart( )	在 Observable.subscribe( )之前立刻
  * onSubscribeReturn( )	在 Observable.subscribe( )之后立刻
  * onSubscribeError( )	在Observable.subscribe( )执行失败时
  * onLift( )	在Observable.lift( )方法中
然后是下面这些步骤：
  * 创建一个你实现的 RxJavaObservableExecutionHook 子类的对象。
  * 使用 RxJavaPlugins.getInstance( ) 获取全局的RxJavaPlugins对象。
  * 将你的Hook对象传递给 RxJavaPlugins 的 registerObservableExecutionHook( ) 方法。
  * 完成这些后，在满足某些特殊的条件时，RxJava会开始调用你的方法。