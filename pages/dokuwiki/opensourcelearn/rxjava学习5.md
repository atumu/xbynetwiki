title: rxjava学习5 

#  RxJava学习之操作符学习二创建操作 
这个页面展示了创建**Observable的各种方法用于创建一个Observable**
  * just( ) — 将一个或多个对象转换成发射这个或这些对象的一个Observable
  * from( ) — 将一个Iterable, 一个Future, 或者一个数组转换成一个Observable
  * repeat( ) — 创建一个重复发射指定数据或数据序列的Observable
  * repeatWhen( ) — 创建一个重复发射指定数据或数据序列的Observable，它依赖于另一个Observable发射的数据
  * create( ) — 使用一个函数从头创建一个Observable
  * defer( ) — 只有当订阅者订阅才创建Observable；为每个订阅创建一个新的Observable

  * range( ) — 创建一个发射指定范围的整数序列的Observable
  * interval( ) — 创建一个按照给定的时间间隔发射整数序列的Observable
  * timer( ) — 创建一个在给定的延时之后发射单个数据的Observable

  * empty( ) — 创建一个什么都不做直接通知完成的Observable
  * error( ) — 创建一个什么都不做直接通知错误的Observable
  * never( ) — 创建一个不发射任何数据的Observable
下面分别来介绍这些方法的使用：
##  Create操作符 
RxJava将这个操作符实现为rx.Observable<T>的create 方法。
public static <T> Observable<T> create(Observable.OnSubscribe<T> f)
create方法默认不在任何特定的调度器上执行。
参数解释：
Type Parameters:
T - 要被发射的事件流类型
Parameters:
f - a function that accepts an Subscriber<T>, and invokes its onNext, onError, and onCompleted methods as appropriate
Returns:
an Observable that, when a Subscriber subscribes to it, will execute the specified function

使用一个函数从头开始创建一个Observable
![](/data/dokuwiki/opensourcelearn/pasted/20160310-221811.png)
你可以使用Create操作符从头开始创建一个Observable，**给这个操作符传递一个接受观察者作为参数的函数，编写这个函数让它的行为表现为一个Observable--恰当的调用观察者的onNext，onError和onCompleted方法。
一个形式正确的有限Observable必须尝试调用观察者的onCompleted正好一次或者它的onError正好一次，而且此后不能再调用观察者的任何其它方法。**

建议你在传递给create方法的函数中**检查观察者的isUnsubscribed状态**，以便在没有观察者的时候，让你的Observable停止发射数据或者做昂贵的运算。
```

Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> observer) {
        try {
            if (!observer.isUnsubscribed()) {
                for (int i = 1; i < 5; i++) {
                    observer.onNext(i);
                }
                observer.onCompleted();
            }
        } catch (Exception e) {
            observer.onError(e);
        }
    }
 } ).subscribe(new Subscriber<Integer>() {
        @Override
        public void onNext(Integer item) {
            System.out.println("Next: " + item);
        }

        @Override
        public void onError(Throwable error) {
            System.err.println("Error: " + error.getMessage());
        }

        @Override
        public void onCompleted() {
            System.out.println("Sequence complete.");
        }
    });

```
##  Defer 
**直到有观察者订阅时才创建Observable，并且为每个观察者创建一个新的Observable**
RxJava将这个操作符实现为 defer 方法。这个操作符接受一个你选择的Observable工厂函数作为单个参数（这个函数没有参数，返回一个Observable。）
public static <T> Observable<T> defer(Func0<Observable<T>> observableFactory)
defer方法默认不在任何特定的调度器上执行。
![](/data/dokuwiki/opensourcelearn/pasted/20160310-222831.png)
Defer操作符会一直等待直到有观察者订阅它，然后它使用Observable工厂方法生成一个Observable。它对每个观察者都这样做，因此尽管每个订阅者都以为自己订阅的是同一个Observable，**事实上每个订阅者获取的是它们自己的单独的数据序列。**
在某些情况下，等待直到最后一分钟（就是知道订阅发生时）才生成Observable可以确保Observable包含最新的数据。
##  Empty/Never/Throw 
**Empty**
创建一个不发射任何数据但是立即调用Subscriber的onCompleted()方法的正常终止的Observable
public static <T> Observable<T> empty()
Returns an Observable that emits no items to the Observer and immediately invokes its onCompleted method.

**Never**
创建一个不发射数据也不终止的Observable
public static <T> Observable<T> never()
Returns an Observable that never sends any items or notifications to an Observer.

**Throw**
创建一个不发射数据以一个错误终止的Observable。（当订阅者订阅时才调用onError()方法）
public static <T> Observable<T> error(java.lang.Throwable exception)
Returns an Observable that invokes an Observer's onError method when the Observer subscribes to it.
这三个操作符生成的Observable行为非常特殊和受限。测试的时候很有用，有时候也用于结合其它的Observables，或者作为其它需要Observable的操作符的参数。
**RxJava将这些操作符实现为 empty，never和error。**error操作符需要一个Throwable参数，你的Observable会以此终止。
**这些操作符默认不在任何特定的调度器上执行**，但是empty和error有一个可选参数是Scheduler，如果你传递了Scheduler参数，它们会在这个调度器上发送通知。

##  From(对象和数据类型转换为Observable)
将其它种类的对象和数据类型转换为Observable
![](/data/dokuwiki/opensourcelearn/pasted/20160310-223851.png)
当你使用Observable时，如果你要处理的数据都可以转换成展现为Observables，而不是需要混合使用Observables和其它类型的数据，会非常方便。这让你在数据流的整个生命周期中，可以使用一组统一的操作符来管理它们。
例如，Iterable可以看成是同步的Observable；Future，可以看成是总是只发射单个数据的Observable。通过显式地将那些数据转换为Observables，你可以像使用Observable一样与它们交互。
因此，**大部分ReactiveX实现都提供了将语言特定的对象和数据结构转换为Observables的方法。**
1、public static <T> Observable<T> from(java.util.concurrent.Future<? extends T> future)
Converts a Future into an Observable.**注意：This Observable is blocking; you cannot unsubscribe from it.**

2、public static <T> Observable<T> from(java.util.concurrent.Future<? extends T> future,long timeout,java.util.concurrent.TimeUnit unit)
Converts a Future into an Observable, with a timeout on the Future.**注意：This Observable is blocking; you cannot unsubscribe from it.**

3、public static <T> Observable<T> from(java.util.concurrent.Future<? extends T> future,Scheduler scheduler)
Converts a Future, operating on a specified Scheduler, into an Observable.

4、public static <T> Observable<T> from(java.lang.Iterable<? extends T> iterable)
Converts an Iterable sequence into an Observable that emits the items in the sequence.

5、public static <T> Observable<T> from(T[] array)
Converts an Array into an Observable that emits the items in the Array.

**在RxJava中，from操作符可以转换Future、Iterable和数组。**对于Iterable和数组，产生的Observable会发射Iterable或数组的每一项数据。
```

Integer[] items = { 0, 1, 2, 3, 4, 5 };
Observable myObservable = Observable.from(items);

myObservable.subscribe(
    new Action1<Integer>() {
        @Override
        public void call(Integer item) {
            System.out.println(item);
        }
    },
    new Action1<Throwable>() {
        @Override
        public void call(Throwable error) {
            System.out.println("Error encountered: " + error.getMessage());
        }
    },
    new Action0() {
        @Override
        public void call() {
            System.out.println("Sequence complete");
        }
    }
);

```
对于Future，它会发射Future.get()方法返回的单个数据。from方法有一个可接受两个可选参数的版本，分别指定超时时长和时间单位。如果过了指定的时长Future还没有返回一个值，这个Observable会发射错误通知并终止。
from默认不在任何特定的调度器上执行。然而你可以将Scheduler作为可选的第二个参数传递给Observable，它会在那个调度器上管理这个Future。
##  Interval(创建按固定时间间隔发射整数序列的Observable) 
创建一个按固定时间间隔发射整数序列的Observable.Interval操作符返回一个Observable，它按固定的时间间隔发射一个无限递增的整数序列。
RxJava将这个操作符实现为interval方法。它接受一个表示时间间隔的参数和一个表示时间单位的参数。
public static Observable<java.lang.Long> interval(long interval,java.util.concurrent.TimeUnit unit)
public static Observable<java.lang.Long> interval(long interval,java.util.concurrent.TimeUnit unit, Scheduler scheduler)
public static Observable<java.lang.Long> interval(long initialDelay,long period,java.util.concurrent.TimeUnit unit)
**interval默认在computation调度器上执行**。你也可以传递一个可选的Scheduler参数来指定调度器。
![](/data/dokuwiki/opensourcelearn/pasted/20160310-225148.png)
##  Just(发射指定值) 
创建一个发射指定值的Observable
**Just将单个数据转换为发射那个数据的Observable。**
**Just类似于From，但是From会将数组或Iterable的素具取出然后逐个发射，而Just只是简单的原样发射，将数组或Iterable当做单个数据。**
注意：如果你传递null给Just，它会返回一个发射null值的Observable。不要误认为它会返回一个空Observable（完全不发射任何数据的Observable），如果需要空Observable你应该使用Empty操作符。
RxJava将这个操作符实现为just函数，它接受一至九个参数，返回一个按参数列表顺序发射这些数据的Observable。
```

Observable.just(1, 2, 3)
          .subscribe(new Subscriber<Integer>() {
        @Override
        public void onNext(Integer item) {
            System.out.println("Next: " + item);
        }

        @Override
        public void onError(Throwable error) {
            System.err.println("Error: " + error.getMessage());
        }

        @Override
        public void onCompleted() {
            System.out.println("Sequence complete.");
        }
    });

```
##  Range(发射特定整数序列) 
创建一个发射特定整数序列的Observable
![](/data/dokuwiki/opensourcelearn/pasted/20160310-225750.png)
Range操作符发射一个范围内的有序整数序列，你可以指定范围的起始和长度。
RxJava将这个操作符实现为range函数，它接受两个参数，一个是范围的起始值，一个是范围的数据的数目。如果你将第二个参数设为0，将导致Observable不发射任何数据（如果设置为负数，会抛异常）。
**range默认不在任何特定的调度器上执行。**有一个变体可以通过可选参数指定Scheduler。
##  Repeat(发射特定数据重复多次) 
创建一个发射特定数据重复多次的Observable
![](/data/dokuwiki/opensourcelearn/pasted/20160310-225918.png)
RxJava将这个操作符实现为repeat方法。它不是创建一个Observable，而是重复发射原始Observable的数据序列，这个序列或者是无限的，或者通过repeat(n)指定重复次数。
**repeat操作符默认在trampoline调度器上执行**。有一个变体可以通过可选参数指定Scheduler。
Javadoc: repeat()) 
Javadoc: repeat(long)) 
Javadoc: repeat(Scheduler)) 
Javadoc: repeat(long,Scheduler))
###  repeatWhen 
还有一个叫做repeatWhen的操作符，它不是缓存和重放原始Observable的数据序列，**而是有条件的重新订阅和发射原来的Observable。**
public final Observable<T> repeatWhen(Func1<? super Observable<? extends java.lang.Void>,? extends Observable<?>> notificationHandler, Scheduler scheduler)
将原始Observable的终止通知（完成或错误）当做一个void数据传递给一个通知处理器，它以此来决定是否要重新订阅和发射原来的Observable。
这个通知处理器就像一个Observable操作符，接受一个发射void通知的Observable为输入，返回一个发射void数据（意思是，重新订阅和发射原始Observable）或者直接终止（意思是，使用repeatWhen终止发射数据）的Observable。
**repeatWhen操作符默认在trampoline调度器上执行。**有一个变体可以通过可选参数指定Scheduler。
![](/data/dokuwiki/opensourcelearn/pasted/20160310-230509.png)
## Async Operators 
**位于可选的rxjava-async模块**
###  Start 
返回一个Observable，它发射一个类似于函数声明的值
![](/data/dokuwiki/opensourcelearn/pasted/20160310-230932.png)
Start操作符的多种RxJava实现都属于**可选的rxjava-async模块。**
rxjava-async模块包含start操作符，它接受一个函数作为参数，调用这个函数获取一个值，然后返回一个会发射这个值给后续观察者的Observable。
**注意：这个函数只会被执行一次，即使多个观察者订阅这个返回的Observable。**
###  toAsync 
rxjava-async模块还包含这几个操作符：toAsync, asyncAction, 和asyncFunc。它们接受一个函数或一个Action作为参数。
对于函数(functions)，这个操作符调用这个函数获取一个值，然后返回一个会发射这个值给后续观察者的Observable（和start一样）。
对于动作(Action)，过程类似，但是没有返回值，在这种情况下，这个操作符在终止前会发射一个null值。+
**注意：这个函数或动作只会被执行一次，即使多个观察者订阅这个返回的Observable。**
###  startFuture 
rxjava-async模块还包含一个startFuture操作符，传递给它一个返回Future的函数，startFuture会立即调用这个函数获取Future对象，然后调用Future的get()方法尝试获取它的值。它返回一个发射这个值给后续观察者的Observable。
###  deferFuture 
rxjava-async模块还包含一个deferFuture操作符，传递给它一个返回Future的函数（这个Future返回一个Observable），deferFuture返回一个Observable，但是不会调用你提供的函数，知道有观察者订阅它返回的Observable。这时，它立即调用Future的get()方法，然后镜像发射get()方法返回的Observable发射的数据。+
用这种方法，你可以在Observables调用链中包含一个返回Observable的Future对象。
###  fromAction 
rxjava-async模块还包含一个fromAction操作符，它接受一个Action作为参数，返回一个Observable，一旦Action终止，它发射这个你传递给fromAction的数据。
###  fromCallable 
rxjava-async模块还包含一个fromCallable操作符，它接受一个Callable作为参数，返回一个发射这个Callable的结果的Observable。
###  fromRunnable 
rxjava-async模块还包含一个fromRunnable操作符，它接受一个Runnable作为参数，返回一个Observable，一旦Runnable终止，它发射这个你传递给fromRunnable的数据。
###  forEachFuture 
rxjava-async模块还包含一个forEachFuture操作符。它其实不算Start操作符的一个变体，而是有一些自己的特点。你传递一些典型的观察者方法（如onNext, onError和onCompleted）给它，Observable会以通常的方式调用它。但是forEachFuture自己返回一个Future并且在get()方法处阻塞，直到原始Observable执行完成，然后它返回，完成还是错误依赖于原始Observable是完成还是错误。
如果你想要一个函数阻塞直到Observable执行完成，可以使用这个操作符。
###  runAsync 
rxjava-async模块还包含一个runAsync操作符。**它很特殊，返回一个叫做StoppableObservable的特殊Observable。**
传递一个Action和一个Scheduler给runAsync，它返回一个使用这个Action产生数据的StoppableObservable。这个Action接受一个Observable和一个Subscription作为参数，它使用Subscription检查unsubscribed条件，一旦发现条件为真就立即停止发射数据。在任何时候你都可以使用unsubscribe方法手动停止一个StoppableObservable（这会同时取消订阅与这个StoppableObservable关联的Subscription）。
**由于runAsync会立即调用Action并开始发射数据，在你创建StoppableObservable之后到你的观察者准备好接受数据之前这段时间里，可能会有一部分数据会丢失。**
**如果这不符合你的要求，可以使用runAsync的一个变体，它也接受一个Subject参数，传递一个ReplaySubject给它，你可以获取其它丢失的数据了。**
在RxJava中还有一个版本的From操作符可以将Future转换为Observable，与start相似。

##  Timer(在给定的延迟后发射一个特殊的值) 
创建一个Observable，它在一个给定的延迟后发射一个特殊的值。
![](/data/dokuwiki/opensourcelearn/pasted/20160310-231214.png)
Timer操作符创建一个在给定的时间段之后返回一个特殊值的Observable。
RxJava将这个操作符实现为timer函数。
timer返回一个Observable，它在延迟一段给定的时间后发射一个简单的数字0。
**timer操作符默认在computation调度器上执行。**有一个变体可以通过可选参数指定Scheduler。
Javadoc: timer(long,TimeUnit))
Javadoc: timer(long,TimeUnit,Scheduler))