title: rxjava学习10 

#  RxJava学习之操作符学习七辅助操作 
这个页面列出了很多用于Observable的辅助操作符
  * materialize( ) — 将Observable转换成一个通知列表convert an Observable into a list of Notifications
  * dematerialize( ) — 将上面的结果逆转回一个Observable
  * timestamp( ) — 给Observable发射的每个数据项添加一个时间戳
  * serialize( ) — 强制Observable按次序发射数据并且要求功能是完好的
  * cache( ) — 记住Observable发射的数据序列并发射相同的数据序列给后续的订阅者
  * **observeOn( ) — 指定观察者观察Observable的调度器**
  * **subscribeOn( ) — 指定Observable执行任务的调度器**
  * doOnEach( ) — 注册一个动作，对Observable发射的每个数据项使用
  * doOnCompleted( ) — 注册一个动作，对正常完成的Observable使用
  * doOnError( ) — 注册一个动作，对发生错误的Observable使用
  * doOnTerminate( ) — 注册一个动作，对完成的Observable使用，无论是否发生错误
  * doOnSubscribe( ) — 注册一个动作，在观察者订阅时使用
  * doOnUnsubscribe( ) — 注册一个动作，在观察者取消订阅时使用
  * finallyDo( ) — 注册一个动作，在Observable完成时使用
  * delay( ) — 延时发射Observable的结果
  * delaySubscription( ) — 延时处理订阅请求
  * timeInterval( ) — 定期发射数据
  * using( ) — 创建一个只在Observable生命周期存在的资源
  * single( ) — 强制返回单个数据，否则抛出异常
  * singleOrDefault( ) — 如果Observable完成时返回了单个数据，就返回它，否则返回默认数据
  * toFuture( ), toIterable( ), toList( ) — 将Observable转换为其它对象或数据结构
##  Delay 
延迟一段指定的时间再发射来自Observable的发射物
**RxJava的实现是 delay和delaySubscription。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-111539.png)
delay默认在computation调度器上执行，你可以通过参数指定使用其它的调度器。
还有一个操作符` delaySubscription `让你你可以延迟订阅原始Observable。它结合搜一个定义延时的参数。
delaySubscription默认在computation调度器上执行，你可以通过参数指定使用其它的调度器。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-111710.png)

##  Do 
注册一个动作作为原始Observable生命周期事件的一种占位符
![](/data/dokuwiki/opensourcelearn/pasted/20160311-111804.png)
你可以**注册回调**，当Observable的某个事件发生时，Rx会在与Observable链关联的正常通知集合中调用它。Rx实现了多种操作符用于达到这个目的。
RxJava实现了很多Do操作符的变体。
###  doOnEach 
doOnEach操作符让你可以注册一个回调，它产生的Observable每发射一项数据就会调用它一次。你可以以Action的形式传递参数给它，这个Action接受一个onNext的变体Notification作为它的唯一参数，你也可以传递一个Observable给doOnEach，这个Observable的onNext会被调用，就好像它订阅了原始的Observable一样。
###  doOnNext 
doOnNext操作符类似于doOnEach(Action1)，但是它的Action不是接受一个Notification参数，而是接受发射的数据项。
```

Observable.just(1, 2, 3)
          .doOnNext(new Action1<Integer>() {
          @Override
          public void call(Integer item) {
            if( item > 1 ) {
              throw new RuntimeException( "Item exceeds maximum value" );
            }
          }
        }).subscribe(new Subscriber<Integer>() {
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
输出
Next: 1
Error: Item exceeds maximum value
###  doOnSubscribe 
doOnSubscribe操作符注册一个动作，当观察者订阅它生成的Observable它就会被调用
###  doOnUnsubscribe 
doOnUnsubscribe操作符注册一个动作，当观察者取消订阅它生成的Observable它就会被调用。
###  doOnCompleted 
doOnCompleted 操作符注册一个动作，当它产生的Observable正常终止调用onCompleted时会被调用。
###  doOnError 
doOnError 操作符注册一个动作，当它产生的Observable异常终止调用onError时会被调用。
###  doOnTerminate/finallyDo 
doOnTerminate/finallyDo 操作符注册一个动作，当它产生的Observable终止之前会被调用，无论是正常还是异常终止。
##  Materialize/Dematerialize 
Materialize将数据项和事件通知都当做数据项发射，Dematerialize刚好相反。
RxJava的materialize将来自原始Observable的通知转换为` Notification对象 `，然后它返回的Observable会发射这些数据。+
materialize默认不在任何特定的调度器 (Scheduler) 上执行。
Dematerialize操作符是Materialize的逆向过程，**它将Materialize转换的结果还原成它原本的形式。**
dematerialize反转这个过程，将原始Observable发射的Notification对象还原成Observable的通知。
dematerialize默认不在任何特定的调度器 (Scheduler) 上执行。
##  ObserveOn 
指定一个**观察者在哪个调度器上观察**这个Observable
很多ReactiveX实现都使用调度器 "Scheduler"来管理多线程环境中Observable的转场。你可以**使用ObserveOn操作符指定Observable在一个特定的调度器上发送通知给观察者 (调用观察者的onNext, onCompleted, onError方法)。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-112553.png)
注意：当遇到一个异常时ObserveOn会立即向前传递这个onError终止通知，它不会等待慢速消费的Observable接受任何之前它已经收到但还没有发射的数据项。**这可能意味着onError通知会跳到（并吞掉）原始Observable发射的数据项前面，正如图例上展示的。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-112657.png)
SubscribeOn操作符的作用类似，但它是用于指定Observable本身在特定的调度器上执行，它同样会在那个调度器上给观察者发通知。
RxJava中，要指定Observable应该在哪个调度器上调用观察者的onNext, onCompleted, onError方法，你需要使用observeOn操作符，传递给它一个合适的Scheduler。
##  SubscribeOn 
**指定Observable自身在哪个调度器上执行**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-112738.png)
很多ReactiveX实现都使用调度器 "Scheduler"来管理多线程环境中Observable的转场。你可以使用SubscribeOn操作符指定Observable在一个特定的调度器上运转。
` ObserveOn操作符的作用类似，但是功能很有限，它指示Observable在一个指定的调度器上给观察者发通知。 `
在某些实现中还有一个UnsubscribeOn操作符。
##  Serialize 
强制一个Observable连续调用并保证行为正确
![](/data/dokuwiki/opensourcelearn/pasted/20160311-112859.png)
一个Observable可以异步调用它的观察者的方法，可能是从不同的线程调用。这可能会让Observable行为不正确，它可能会在某一个onNext调用之前尝试调用onCompleted或onError方法，或者从两个不同的线程同时调用onNext方法。**使用Serialize操作符，你可以纠正这个Observable的行为，保证它的行为是正确的且是同步的。**
RxJava中的实现是serialize，它默认不在任何特定的调度器上执行。
##  Subscribe 
操作来自Observable的发射物和通知
**Subscribe操作符是连接观察者和Observable的胶水。**一个观察者要想看到Observable发射的数据项，或者想要从Observable获取错误和完成通知，它首先必须使用这个操作符订阅那个Observable。
Subscribe操作符的一般实现可能会接受一到三个方法（然后由观察者组合它们），或者接受一个实现了包含这三个方法的接口的对象（有时叫做Observer或Subscriber）：
onNext
每当Observable发射了一项数据它就会调用这个方法。这个方法的参数是这个Observable发射的数据项。
onError
Observable调用这个方法表示它无法生成期待的数据或者遇到了其它错误。这将停止Observable，它在这之后不会再调用onNext或onCompleted。onError方法的参数是导致这个错误的原因的一个表示（有时可能是一个Exception或Throwable对象，其它时候也可能是一个简单的字符串，取决于具体的实现）。
onCompleted
**如果没有遇到任何错误，Observable在最后一次调用onCompleted之后会调用这个方法。**
` 如果一个Observable直到有一个观察者订阅它才开始发射数据项，就称之为"冷"的Observable；如果一个Observable可能在任何时刻开始发射数据，就称之为"热"的Observable，一个订阅者可能从开始之后的某个时刻开始观察它发射的数据序列，它可能会错过在订阅之前发射的数据。 `
RxJava中的实现是subscribe方法。
如果你使用无参数的版本，它将触发对Observable的一个订阅，但是将忽略它的发射物和通知。这个操作会激活一个"冷"的Observable。
你也可以传递一到三个函数给它，它们会按下面的方法解释：
onNext
onNext和onError
onNext, onError和onCompleted
最后，你还可以传递一个Observer或Subscriber接口给它，Observer接口包含这三个以on开头的方法。Subscriber接口也实现了这三个方法，而且还添加了几个额外的方法，用于支持使用反压操作(reactive pull backpressure)，这让Subscriber可以在Observable完成前取消订阅。
**subscribe方法返回一个实现了Subscription接口的对象。这个接口包含unsubscribe方法，任何时刻你都可以调用它来断开subscribe方法建立的Observable和观察者之间的订阅关系。**
###  foreach 
forEach方法是简化版的subscribe，你同样可以传递一到三个函数给它，解释和传递给subscribe时一样。
不同的是，你无法使用forEach返回的对象取消订阅。也没办法传递一个可以用于取消订阅的参数。因此，只有当你明确地需要操作Observable的所有发射物和通知时，你才应该使用这个操作符。
BlockingObservable
BlockingObservable类中也有一个类似的叫作forEach的方法。详细的说明见 BlockingObservable
##  TimeInterval 
将一个发射数据的Observable转换为发射那些数据发射时间间隔的Observable
![](/data/dokuwiki/opensourcelearn/pasted/20160311-113358.png)
TimeInterval操作符拦截原始Observable发射的数据项，替换为发射表示相邻发射物时间间隔的对象。
**RxJava中的实现为timeInterval，这个操作符将原始Observable转换为另一个Obserervable，后者发射一个标志替换前者的数据项，这个标志表示前者的两个连续发射物之间流逝的时间长度**。新的Observable的第一个发射物表示的是在观察者订阅原始Observable到原始Observable发射它的第一项数据之间流逝的时间长度。不存在与原始Observable发射最后一项数据和发射onCompleted通知之间时长对应的发射物。
timeInterval默认在immediate调度器上执行，你可以通过传参数修改。
##  Timeout 
对原始Observable的一个镜像，如果过了一个指定的时长仍没有发射数据，它会发一个错误通知
![](/data/dokuwiki/opensourcelearn/pasted/20160311-113509.png)
如果原始Observable过了指定的一段时长没有发射任何数据，Timeout操作符会以一个onError通知终止这个Observable。
RxJava中的实现为timeout，但是有好几个变体。
##  Timestamp 
给Observable发射的数据项附加一个时间戳
RxJava中的实现为timestamp，它将一个发射T类型数据的Observable转换为一个发射类型为` Timestamped<T> `的数据的Observable，**每一项都包含数据的原始发射时间。**
**timestamp默认在immediate调度器上执行**，但是可以通过参数指定其它的调度器。
##  Using 
创建一个只在Observable生命周期内存在的**一次性资源**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-113735.png)
Using操作符让你可以指示Observable创建一个只在它的生命周期内存在的资源，当Observable终止时这个资源会被自动释放。
using操作符接受三个参数：
  * 一个用户创建一次性资源的工厂函数
  * 一个用于创建Observable的工厂函数
  * 一个用于释放资源的函数

当一个观察者订阅using返回的Observable时，using将会使用Observable工厂函数创建观察者要观察的Observable，同时使用资源工厂函数创建一个你想要创建的资源。当观察者取消订阅这个Observable时，或者当观察者终止时（无论是正常终止还是因错误而终止），using使用第三个函数释放它创建的资源。
using默认不在任何特定的调度器上执行。
##  To 
将Observable转换为另一个对象或数据结构
![](/data/dokuwiki/opensourcelearn/pasted/20160311-113900.png)
在某些ReactiveX实现中，还有一个操作符用于将Observable转换成阻塞式的。一个阻塞式的Ogbservable在普通的Observable的基础上增加了几个方法，用于操作Observable发射的数据项。
  * getIterator操作符只能用于` BlockingObservable `的子类，要使用它，你首先必须把原始的Observable转换为一个BlockingObservable。可以使用这两个操作符：BlockingObservable.from或the Observable.toBlocking。
这个操作符将Observable转换为一个Iterator，你可以通过它迭代原始Observable发射的数据集。
Javadoc: BlockingObservable.getIterator())
  * toFuture操作符也是只能用于` BlockingObservable `。这个操作符将Observable转换为一个返回单个数据项的Future，如果原始Observable发射多个数据项，Future会收到一个IllegalArgumentException；如果原始Observable没有发射任何数据，Future会收到一个NoSuchElementException。
如果你想将发射多个数据项的Observable转换为Future，可以这样用：myObservable.toList().toBlocking().toFuture()。
Javadoc: BlockingObservable.toFuture())
  * toIterable操作符也是只能用于BlockingObservable。这个操作符将Observable转换为一个Iterable，你可以通过它迭代原始Observable发射的数据集。
Javadoc: BlockingObservable.toIterable())
通常，发射多项数据的Observable会为每一项数据调用onNext方法。
  * 用toList操作符改变这个行为，让Observable将多项数据组合成一个List，然后调用一次onNext方法传递整个列表。
如果原始Observable没有发射任何数据就调用了onCompleted，toList返回的Observable会在调用onCompleted之前发射一个空列表。如果原始Observable调用了onError，toList返回的Observable会立即调用它的观察者的onError方法。
toList默认不在任何特定的调度器上执行。
  * toMap收集原始Observable发射的所有数据项到一个Map（默认是HashMap）然后发射这个Map。你可以提供一个用于生成Map的Key的函数，还可以提供一个函数转换数据项到Map存储的值（默认数据项本身就是值）。+
toMap默认不在任何特定的调度器上执行。
  * toMultiMap类似于toMap，不同的是，它生成的这个Map同时还是一个ArrayList（默认是这样，你可以传递一个可选的工厂方法修改这个行为）。
toMultiMap默认不在任何特定的调度器上执行。
  * toSortedList类似于toList，不同的是，它会对产生的列表排序，默认是自然升序，如果发射的数据项没有实现Comparable接口，会抛出一个异常。然而，你也可以传递一个函数作为用于比较两个数据项，这是toSortedList不会使用Comparable接口。
toSortedList默认不在任何特定的调度器上执行。
  * nest操作符有一个特殊的用途：将一个Observable转换为一个发射这个Observable的Observable。