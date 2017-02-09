title: rxjava学习3 

#  RxJava学习之调度器Scheduler 
**如果你想给Observable操作符链添加多线程功能，你可以指定操作符（或者特定的Observable）在特定的调度器(Scheduler)上执行。**
某些ReactiveX的Observable操作符有一些变体，它们可以接受一个Scheduler参数。**这个参数指定操作符将它们的部分或全部任务放在一个特定的调度器上执行。**
使用` ObserveOn和SubscribeOn `操作符，你可以**让Observable在一个特定的调度器上执行**，
**ObserveOn指示一个Observable在一个特定的调度器上调用观察者的onNext, onError和onCompleted方法，SubscribeOn更进一步，它指示Observable将全部的处理过程（包括发射数据和通知）放在特定的调度器上执行。**

###  RxJava调度器的种类 
下表展示了RxJava中可用的调度器种类：
调度器类型	效果
  * Schedulers.computation( )	用于计算任务，如事件循环或和回调处理，**不要用于IO操作(IO操作请使用Schedulers.io())；默认线程数等于处理器的数量**
  * Schedulers.from(executor)	使用指定的Executor作为调度器
  * Schedulers.immediate( )	在当前线程立即开始执行任务
  * Schedulers.io( )	**用于IO密集型任务，如异步阻塞IO操作**，这个调度器的线程池会根据需要增长；对于普通的计算任务，请使用Schedulers.computation()；Schedulers.io( )默认是一个CachedThreadScheduler，很像一个有线程缓存的新线程调度器
  * Schedulers.newThread( )	为每个任务创建一个新线程
  * Schedulers.trampoline( )	当其它排队的任务完成后，在当前线程排队开始执行
###  RxJava操作符的默认调度器列表 
在RxJava中，某些Observable操作符的变体允许你设置用于操作执行的调度器，其它的则不在任何特定的调度器上执行，或者在一个指定的默认调度器上执行。下面的表格个**列出了一些操作符的默认调度器**：
<html>
<table>
<thead>
<tr>
<th>操作符</th>
<th>调度器</th>
</tr>
</thead>
<tbody>
<tr>
<td>buffer(timespan)</td>
<td>computation</td>
</tr>
<tr>
<td>buffer(timespan, count)</td>
<td>computation</td>
</tr>
<tr>
<td>buffer(timespan, timeshift)</td>
<td>computation</td>
</tr>
<tr>
<td>debounce(timeout, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>delay(delay, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>delaySubscription(delay, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>interval</td>
<td>computation</td>
</tr>
<tr>
<td>repeat</td>
<td>trampoline</td>
</tr>
<tr>
<td>replay(time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>replay(buffersize, time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>replay(selector, time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>replay(selector, buffersize, time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>retry</td>
<td>trampoline</td>
</tr>
<tr>
<td>sample(period, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>skip(time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>skipLast(time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>take(time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>takeLast(time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>takeLast(count, time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>takeLastBuffer(time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>takeLastBuffer(count, time, unit)</td>
<td>computation</td>
</tr>
<tr>
<td>throttleFirst</td>
<td>computation</td>
</tr>
<tr>
<td>throttleLast</td>
<td>computation</td>
</tr>
<tr>
<td>throttleWithTimeout</td>
<td>computation</td>
</tr>
<tr>
<td>timeInterval</td>
<td>immediate</td>
</tr>
<tr>
<td>timeout(timeoutSelector)</td>
<td>immediate</td>
</tr>
<tr>
<td>timeout(firstTimeoutSelector, timeoutSelector)</td>
<td>immediate</td>
</tr>
<tr>
<td>timeout(timeoutSelector, other)</td>
<td>immediate</td>
</tr>
<tr>
<td>timeout(timeout, timeUnit)</td>
<td>computation</td>
</tr>
<tr>
<td>timeout(firstTimeoutSelector, timeoutSelector, other)</td>
<td>immediate</td>
</tr>
<tr>
<td>timeout(timeout, timeUnit, other)</td>
<td>computation</td>
</tr>
<tr>
<td>timer</td>
<td>computation</td>
</tr>
<tr>
<td>timestamp</td>
<td>immediate</td>
</tr>
<tr>
<td>window(timespan)</td>
<td>computation</td>
</tr>
<tr>
<td>window(timespan, count)</td>
<td>computation</td>
</tr>
<tr>
<td>window(timespan, timeshift)</td>
<td>computation</td>
</tr>
</tbody>
</table>
</html>
##  使用调度器 
**除了将这些调度器传递给RxJava的Observable操作符，你也可以用它们调度你自己的任务。**下面的示例展示了` Scheduler.Worker `的用法：
```

worker = Schedulers.newThread().createWorker();
worker.schedule(new Action0() {

    @Override
    public void call() {
        yourWork();
    }

});
// some time later...
worker.unsubscribe();

```
###  递归调度器 
要调度递归的方法调用，你可以使用schedule，然后再用schedule(this)，示例：
```

worker = Schedulers.newThread().createWorker();
worker.schedule(new Action0() {

    @Override
    public void call() {
        yourWork();
        // recurse until unsubscribed (schedule will do nothing if unsubscribed)
        worker.schedule(this);
    }

});
// some time later...
worker.unsubscribe();

```
###  检查或设置取消订阅状态 
**Worker类的对象实现了Subscription接口，使用它的isUnsubscribed和unsubscribe方法，所以你可以在订阅取消时停止任务，或者从正在调度的任务内部取消订阅**，示例：
```

Worker worker = Schedulers.newThread().createWorker();
Subscription mySubscription = worker.schedule(new Action0() {

    @Override
    public void call() {
        while(!worker.isUnsubscribed()) {
            status = yourWork();
            if(QUIT == status) { worker.unsubscribe(); }
        }
    }

});

```
**Worker同时是Subscription，因此你可以（通常也应该）调用它的unsubscribe方法通知可以挂起任务和释放资源了。**


###  延时和周期调度器 
你可以使用` schedule(action,delayTime,timeUnit) `在指定的调度器上**延时执行你的任务**，下面例子中的任务将在500毫秒之后开始执行：
someScheduler.schedule(someAction, 500, TimeUnit.MILLISECONDS);
使用另一个版本的schedule，` schedulePeriodically(action,initialDelay,period,timeUnit) `方法让你可以安排一个**定期执行的任务**，下面例子的任务将在500毫秒之后执行，然后每250毫秒执行一次：
someScheduler.schedulePeriodically(someAction, 500, 250, TimeUnit.MILLISECONDS);

###  测试调度器 
TestScheduler让你可以对调度器的时钟表现进行手动微调。这对依赖精确时间安排的任务的测试很有用处。这个调度器有三个额外的方法：
  * advanceTimeTo(time,unit) 向前波动调度器的时钟到一个指定的时间点
  * advanceTimeBy(time,unit) 将调度器的时钟向前拨动一个指定的时间段
  * triggerActions( ) 开始执行任何计划中的但是未启动的任务，如果它们的计划时间等于或者早于调度器时钟的当前时间
