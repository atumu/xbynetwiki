title: rxjava学习1 

#  RxJava学习之介绍 
github:https://github.com/ReactiveX/RxJava
javadoc:http://reactivex.io/RxJava/javadoc/
wiki:https://github.com/ReactiveX/RxJava/wiki
中文翻译:https://mcxiaoke.gitbooks.io/rxdocs/content/
书籍:rxjava-essentials-cn
ReactiveX是Reactive Extensions的缩写，一般简写为Rx.Rx是一个编程模型，**目标是提供一致的编程接口，帮助开发者更方便的处理异步数据流**，Rx库支持.NET、JavaScript和C++，Rx近几年越来越流行了，现在已经支持几乎全部的流行编程语言了，Rx的大部分语言库由ReactiveX这个组织负责维护，**比较流行的有RxJava/RxJS/Rx.NET**，社区网站是 reactivex.io。
Rx是一个函数库，让开发者可以利用**可观察序列**和**LINQ风格查询操作符**来**编写异步和基于事件的程序**，使用Rx，开发者可以**用Observables表示异步数据流，用LINQ操作符查询异步数据流， 用Schedulers参数化异步数据流的并发处理**，Rx可以这样定义：**Rx = Observables + LINQ + Schedulers。**
ReactiveX.io给的定义是，Rx是一个使用可观察数据流进行异步编程的编程接口，ReactiveX结合了观察者模式、迭代器模式和函数式编程的精华。

**RxJava是 ReactiveX 在JVM上的一个实现，ReactiveX使用Observable序列组合异步和基于事件的程序。**
RxJava的特点：
  * Zero Dependencies
  * < 1MB Jar
  * Java 6+ & Android 2.3+
  * Java 8 lambda support
  * Polyglot (Scala, Groovy, Clojure and Kotlin)
  * Non-opinionated about source of concurrency (threads, pools, event loops, fibers, actors, etc)
  * Async or synchronous execution
  * Virtual time and schedulers for parameterized concurrency

目前还是处于RxJava1.x时代，2.x正在开发当中。还需要些时日
Maven:
```

<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>1.1.1</version>
</dependency>

```

RxJava 第三方库
下面是可与RxJava协作的第三方库：
  * Hystrix - 用于分布式系统的一个延时和容错处理框架
  * Camel RX - 一个用于Apache Camel 的 RxJava 兼容层
  * rxjava-http-tail - 让你可以跟踪HTTP日志，就像使用 tail -f 一样
  * mod-rxvertx - Extension for VertX - 使用 RxJava 封装的VertX库
  * rxjava-jdbc - 使用RxJava流式处理JDBC连接，还支持语句的函数式组合
  * rtree - 使用RxJava实现的一个纯内存的可变的R-tree和R*-tree

下面的示例从一个字符串列表创建一个Observable，然后使用一个方法订阅这个Observable。
```

public static void hello(String... names) {
    Observable.from(names).subscribe(new Action1<String>() {

        @Override
        public void call(String s) {
            System.out.println("Hello " + s + "!");
        }

    });
}
hello("Ben", "George");

```
Hello Ben!
Hello George!

如何使用RxJava
**要使用RxJava，首先你需要创建Observable（它们发射数据序列），使用Observable操作符变换那些Observables，获取严格符合你要求的数据，然后观察并处理对这些数据序列**（通过实现观察者或订阅者，然后订阅变换后的Observable）。


##  ReactiveX的概念 
Rx模式
**使用观察者模式**
  * 创建：Rx可以方便的创建**事件流和数据流**
  * 组合：Rx使用**查询式的操作符**组合和变换数据流
  * 监听：Rx可以**订阅任何可观察的数据流**并执行操作

简化代码
  * 函数式风格：对**可观察数据流**使用无副作用的输入输出函数，避免了程序里错综复杂的状态
  * 简化代码：**Rx的操作符**通通常可以将复杂的难题简化为很少的几行代码
  * 异步错误处理：传统的try/catch没办法处理异步计算，Rx提供了合适的**错误处理机制**
  * 轻松使用并发：Rx的` Observables和Schedulers `让开发者**可以摆脱底层的线程同步和各种并发问题**

###  使用Observable的优势 
**Rx扩展了观察者模式用于支持数据和事件序列，添加了一些操作符，它让你可以声明式的组合这些序列，而无需关注底层的实现：如线程、同步、线程安全、并发数据结构和非阻塞IO。**
` Observable `通过使用最佳的方式访问异步数据序列填补了这个间隙
```

		单个数据	多个数据
同步	T getData()	Iterable<T> getData()
异步	Future<T> getData()	Observable<T> getData()

```
Rx的Observable模型让你**可以像使用集合数据一样操作异步事件流**，对**异步事件流**使用各种简单、**可组合的操作**。

###  Observable可组合 
对于单层的异步操作来说，Java中Future对象的处理方式是非常简单有效的，但是一旦涉及到嵌套，它们就开始变得异常繁琐和复杂。**使用Future很难很好的组合带条件的异步执行流程**（考虑到运行时各种潜在的问题，甚至可以说是不可能的），当然，要想实现还是可以做到的，但是非常困难，或许你可以用Future.get()，但这样做，异步执行的优势就完全没有了。从另一方面说，**Rx的` Observable `一开始就是为` 组合异步数据流 `准备的。**
###  Observable更灵活 
Rx的Observable不仅支持处理单独的标量值（就像Future可以做的），也支持数据序列，甚至是无穷的数据流。Observable是一个抽象概念，适用于任何场景。Observable拥有它的近亲Iterable的全部优雅与灵活。
**Observable是异步的双向push，Iterable是同步的单向pull**，对比：
事件	Iterable(pull)	Observable(push)
获取数据	T next()	onNext(T)
异常处理	throws Exception	onError(Exception)
任务完成	!hasNext()	onCompleted()

###  Observable的底层无关一致性 
Rx对于对于并发性或异步性没有任何特殊的偏好，Observable可以用任何方式实现，线程池、事件循环、非阻塞IO、Actor模式，任何满足你的需求的，你擅长或偏好的方式都可以。无论你选择怎样实现它，**无论底层实现是阻塞的还是非阻塞的，客户端代码将所有与Observable的交互都当做是异步的。**
Observable是如何实现的？
public Observable<data> getData();
它能与调用者在同一线程同步执行吗？
它能异步地在单独的线程执行吗？
它会将工作分发到多个线程，返回数据的顺序是任意的吗？
它使用Actor模式而不是线程池吗？
它使用NIO和事件循环执行异步网络访问吗？
它使用事件循环将工作线程从回调线程分离出来吗？
从Observer的视角看，这些都无所谓，重要的是：**使用Rx，你可以改变你的观念，你可以在完全不影响Observable程序库使用者的情况下，彻底的改变Observable的底层实现。**
**使用回调存在很多问题**
回调在不阻塞任何事情的情况下，解决了Future.get()过早阻塞的问题。由于响应结果一旦就绪Callback就会被调用，它们天生就是高效率的。不过，就像使用Future一样，**对于单层的异步执行来说，回调很容易使用，对于嵌套的异步组合，它们显得非常笨拙。**

Rx是一个多语言的实现 
Rx在大量的编程语言中都有实现，并尊重实现语言的风格，而且更多的实现正在飞速增加。

###  Rx响应式编程 
Rx提供了一系列的**操作符**，你可以使用它们来**过滤(filter)、选择(select)、变换(transform)、结合(combine)和组合(compose)多个Observable**，这些操作符让执行和复合变得非常高效。
你可以把Observable当做Iterable的推送方式的等价物，使用Iterable，消费者从生产者那拉取数据，线程阻塞直至数据准备好。
**使用Observable，在数据准备好时，生产者将数据推送给消费者。数据可以同步或异步的到达**，这种方式更灵活。
下面的例子展示了相似的高阶函数在Iterable和Observable上的应用
```

// Iterable
getDataFromLocalMemory()
  .skip(10)
  .take(5)
  .map({ s -> return s + " transformed" })
  .forEach({ println "next => " + it })

// Observable
getDataFromNetwork()
  .skip(10)
  .take(5)
  .map({ s -> return s + " transformed" })
  .subscribe({ println "onNext => " + it })

```
Observable类型给GOF的观察者模式添加了两种缺少的语义，这样就和Iterable类型中可用的操作一致了：
生产者可以发信号给消费者，通知它没有更多数据可用了（对于Iterable，一个for循环正常完成表示没有数据了；**对于Observable，就是调用观察者的onCompleted方法**）
生产者可以发信号给消费者，通知它遇到了一个错误（对于Iterable，迭代过程中发生错误会抛出异常；**对于Observable，就是调用观察者(Observer)的onError方法**）
有了这两种功能，Rx就能使Observable与Iterable保持一致了，唯一的不同是数据流的方向。任何对Iterable的操作，你都可以对Observable使用。

###  名词定义 
这里给出一些名词的翻译
  * Reactive 直译为反应性的，有活性的，根据上下文一般翻译为反应式、响应式
  * Iterable 可迭代对象，支持以迭代器的形式遍历，许多语言中都存在这个概念
  * Observable 可观察对象，在Rx中定义为更强大的Iterable，在观察者模式中是被观察的对象，一旦数据产生或发生变化，会通过某种方式通知观察者或订阅者
  * Observer 观察者对象，监听Observable发射的数据并做出响应，Subscriber是它的一个特殊实现
  * emit 直译为发射，发布，发出，含义是Observable在数据产生或变化时发送通知给Observer，调用Observer对应的方法，文章里一律译为发射
  * items 直译为项目，条目，在Rx里是指Observable发射的数据项，文章里一律译为数据，数据项

##  Observable说明 
在ReactiveX中，**一个观察者(Observer)订阅一个可观察对象(Observable)。观察者对Observable发射的数据或数据序列作出响应。这种模式可以极大地简化并发操作**，因为它创建了一个处于待命状态的观察者哨兵，在未来某个时刻响应Observable的通知，不需要阻塞等待Observable发射数据。
这篇文章会解释什么是响应式编程模式(reactive pattern)，以及什么是可观察对象(Observables)和观察者(observers)，其它几篇文章会展示如何用操作符组合和改变Observable的行为。
![](/data/dokuwiki/opensourcelearn/pasted/20160310-212052.png)
创建观察者
本文使用类似于Groovy的伪代码举例，但是ReactiveX有多种语言的实现。
在异步模型中流程更像这样的：
  * 定义一个方法，它完成某些任务，然后从异步调用中返回一个值，这个方法是观察者的一部分
  * 将这个异步调用本身定义为一个Observable
  * 观察者通过订阅(Subscribe)操作关联到那个Observable
  * 继续你的业务逻辑，等方法返回时，Observable会发射结果，观察者的方法会开始处理结果或结果集
用代码描述就是：
```

// defines, but does not invoke, the Subscriber's onNext handler
// (in this example, the observer is very simple and has only an onNext handler)
def myOnNext = { it -> do something useful with it };
// defines, but does not invoke, the Observable
def myObservable = someObservable(itsParameters);
// subscribes the Subscriber to the Observable, and invokes the Observable
myObservable.subscribe(myOnNext);
// go on about my business

```
###  回调方法 (onNext, onCompleted, onError) 
**subscribe方法用于将观察者连接到Observable**，你的观察者需要实现以下方法的一个子集：
  * onNext(T item) Observable调用这个方法发射数据，方法的参数就是Observable发射的数据，这个方法可能会被调用多次，取决于你的实现。
  * onError(Exception ex) 当Observable遇到错误或者无法返回期望的数据时会调用这个方法，这个调用会终止Observable，后续不会再调用onNext和onCompleted，onError方法的参数是抛出的异常。+
  * onComplete 正常终止，如果没有遇到错误，Observable在最后一次调用onNext之后调用此方法。
根据Observable协议的定义，**onNext可能会被调用零次或者很多次，最后会有一次onCompleted或onError调用（不会同时），传递数据给onNext通常被称作发射，onCompleted和onError被称作通知。**
```

def myOnNext     = { item -> /* do something useful with item */ };
def myError      = { throwable -> /* react sensibly to a failed call */ };
def myComplete   = { /* clean up after the final response */ };
def myObservable = someMethod(itsParameters);
myObservable.subscribe(myOnNext, myError, myComplete);
// go on about my business

```
###  取消订阅 (Unsubscribing) 
在一些ReactiveX实现中，有一个特殊的观察者接口Subscriber，它有一个unsubscribe方法。调用这个方法表示你不关心当前订阅的Observable了，因此Observable可以选择停止发射新的数据项（如果没有其它观察者订阅）。
取消订阅的结果会传递给这个Observable的操作符链，而且会导致这个链条上的每个环节都停止发射数据项。这些并不保证会立即发生，然而，对一个Observable来说，即使没有观察者了，它也可以在一个while循环中继续生成并尝试发射数据项。
###  Observables的"热"和"冷" 
Observable什么时候开始发射数据序列？
这取决于Observable的实现，
  * **一个"热"的Observable可能一创建完就开始发射数据**，因此所有后续订阅它的观察者可能从序列中间的某个位置开始接受数据（**有一些数据错过了**）。
  * **一个"冷"的Observable会一直等待，直到有观察者订阅它才开始发射数据，因此这个观察者可以确保会收到整个数据序列。**
  * 在一些ReactiveX实现里，还存在一种被称作Connectable的Observable，不管有没有观察者订阅它，这种Observable都不会开始发射数据，除非Connect方法被调用。
###  用操作符组合Observable 
对于ReactiveX来说，` Observable和Observer `仅仅是个开始，它们本身不过是标准观察者模式的一些轻量级扩展，目的是为了更好的**处理事件序列**。
ReactiveX真正强大的地方在于它的操作符，**操作符让你可以变换、组合、操纵和处理Observable发射的数据。**
Rx的操作符让你可以**用声明式的风格组合异步操作序列**，它拥有**回调**的所有效率优势，同时又避免了典型的异步系统中**嵌套回调**的缺点。
**下面是常用的操作符列表：**
  * 创建操作 Create, Defer, Empty/Never/Throw, From, Interval, Just, Range, Repeat, Start, Timer
  * 变换操作 Buffer, FlatMap, GroupBy, Map, Scan和Window
  * 过滤操作 Debounce, Distinct, ElementAt, Filter, First, IgnoreElements, Last, Sample, Skip, SkipLast, Take, TakeLast
  * 组合操作 And/Then/When, CombineLatest, Join, Merge, StartWith, Switch, Zip
  * 错误处理 Catch和Retry
  * 辅助操作 Delay, Do, Materialize/Dematerialize, ObserveOn, Serialize, Subscribe, SubscribeOn, TimeInterval, Timeout, Timestamp, Using
  * 条件和布尔操作 All, Amb, Contains, DefaultIfEmpty, SequenceEqual, SkipUntil, SkipWhile, TakeUntil, TakeWhile
  * 算术和集合操作 Average, Concat, Count, Max, Min, Reduce, Sum
  * 转换操作 To
  * 连接操作 Connect, Publish, RefCount, Replay
  * 反压操作，用于增加特殊的流程控制策略的操作符
这些操作符并不全都是ReactiveX的核心组成部分，有一些是语言特定的实现或可选的模块。

在RxJava中，一个实现了Observer接口的对象可以订阅(subscribe)一个Observable 类的实例。订阅者(subscriber)对Observable发射(emit)的任何数据或数据序列作出响应。这种模式简化了并发操作，因为它不需要阻塞等待Observable发射数据，而是创建了一个处于待命状态的观察者哨兵，哨兵在未来某个时刻响应Observable的通知。

##  Single-特殊的Observable 
RxJava（以及它派生出来的RxGroovy和RxScala）中**有一个名为Single的Observable变种。**
**Single类似于Observable，不同的是，它总是只发射一个值，或者一个错误通知，而不是发射一系列的值。**
因此，不同于Observable需要三个方法onNext, onError, onCompleted，**订阅Single只需要两个方法：**
  * onSuccess - Single发射单个的值到这个方法
  * onError - 如果无法发射需要的值，Single发射一个Throwable对象到这个方法
` Single只会调用这两个方法中的一个，而且只会调用一次，调用了任何一个方法之后，订阅关系终止。 `
###  Single的操作符 
<html>
<table>
<thead>
<tr>
<th>操作符</th>
<th>返回值</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>compose</td>
<td>Single</td>
<td>创建一个自定义的操作符</td>
</tr>
<tr>
<td>concat and concatWith</td>
<td>Observable</td>
<td>连接多个Single和Observable发射的数据</td>
</tr>
<tr>
<td>create</td>
<td>Single</td>
<td>调用观察者的create方法创建一个Single</td>
</tr>
<tr>
<td>error</td>
<td>Single</td>
<td>返回一个立即给订阅者发射错误通知的Single</td>
</tr>
<tr>
<td>flatMap</td>
<td>Single</td>
<td>返回一个Single，它发射对原Single的数据执行flatMap操作后的结果</td>
</tr>
<tr>
<td>flatMapObservable</td>
<td>Observable</td>
<td>返回一个Observable，它发射对原Single的数据执行flatMap操作后的结果</td>
</tr>
<tr>
<td>from</td>
<td>Single</td>
<td>将Future转换成Single</td>
</tr>
<tr>
<td>just</td>
<td>Single</td>
<td>返回一个发射一个指定值的Single</td>
</tr>
<tr>
<td>map</td>
<td>Single</td>
<td>返回一个Single，它发射对原Single的数据执行map操作后的结果</td>
</tr>
<tr>
<td>merge</td>
<td>Single</td>
<td>将一个Single(它发射的数据是另一个Single，假设为B)转换成另一个Single(它发射来自另一个Single(B)的数据)</td>
</tr>
<tr>
<td>merge and mergeWith</td>
<td>Observable</td>
<td>合并发射来自多个Single的数据</td>
</tr>
<tr>
<td>observeOn</td>
<td>Single</td>
<td>指示Single在指定的调度程序上调用订阅者的方法</td>
</tr>
<tr>
<td>onErrorReturn</td>
<td>Single</td>
<td>将一个发射错误通知的Single转换成一个发射指定数据项的Single</td>
</tr>
<tr>
<td>subscribeOn</td>
<td>Single</td>
<td>指示Single在指定的调度程序上执行操作</td>
</tr>
<tr>
<td>timeout</td>
<td>Single</td>
<td>它给原有的Single添加超时控制，如果超时了就发射一个错误通知</td>
</tr>
<tr>
<td>toSingle</td>
<td>Single</td>
<td>将一个发射单个值的Observable转换为一个Single</td>
</tr>
<tr>
<td>zip and zipWith</td>
<td>Single</td>
<td>将多个Single转换为一个，后者发射的数据是对前者应用一个函数后的结果</td>
</tr>
</tbody>
</table>
</html>

