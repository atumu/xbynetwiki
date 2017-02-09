title: rxjava学习8 

#  RxJava学习之操作符学习五结合操作 
这个页面展示的操作符**可用于组合多个Observables。**
  * startWith( ) — 在数据序列的开头增加一项数据
  * merge( ) — 将多个Observable合并为一个
  * mergeDelayError( ) — 合并多个Observables，让没有错误的Observable都完成后再发射错误通知
  * **zip( ) — 使用一个函数组合多个Observable发射的数据集合，然后再发射这个结果**
  * and( ), then( ), and when( ) — (rxjava-joins) 通过模式和计划组合多个Observables发射的数据集合
  * combineLatest( ) — 当两个Observables中的任何一个发射了一个数据时，通过一个指定的函数组合每个Observable发射的最新数据（一共两个数据），然后发射这个函数的结果
  * join( ) and groupJoin( ) — 无论何时，如果一个Observable发射了一个数据项，只要在另一个Observable发射的数据项定义的时间窗口内，就将两个Observable发射的数据合并发射
  * switchOnNext( ) — 将一个发射Observables的Observable转换成另一个Observable，后者发射这些Observables最近发射的数据
(rxjava-joins) — 表示这个操作符当前是可选的**rxjava-joins包**的一部分，还没有包含在标准的RxJava操作符集合里
##  And/Then/When 
使用Pattern和Plan作为中介，将两个或多个Observable发射的数据集合并到一起
![](/data/dokuwiki/opensourcelearn/pasted/20160311-105205.png)
And/Then/When操作符组合的行为类似于zip，但是它们使用一个中间数据结构。接受两个或多个Observable，一次一个将它们的发射物合并到Pattern对象，然后操作那个Pattern对象，变换为一个Plan。随后将这些Plan变换为Observable的发射物。
它们属于**rxjava-joins模块**，不是核心RxJava包的一部分。
##  CombineLatest 
当两个Observables中的任何一个发射了数据时，使用一个函数结合每个Observable发射的最近数据项，并且基于这个函数的结果发射数据。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-105355.png)
CombineLatest操作符行为类似于zip，但是只有当原始的Observable中的每一个都发射了一条数据时zip才发射数据。CombineLatest则在原始的Observable中任意一个发射了数据时发射一条数据。当原始Observables的任何一个发射了一条数据时，CombineLatest使用一个函数结合它们最近发射的数据，然后发射这个函数的返回值。
RxJava将这个操作符实现为combineLatest，它接受二到九个Observable作为参数，或者单个Observables列表作为参数。它默认不在任何特定的调度器上执行。

##  Join 
任何时候，只要在另一个Observable发射的数据定义的时间窗口内，这个Observable发射了一条数据，就结合两个Observable发射的数据。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-105556.png)
Join操作符结合两个Observable发射的数据，基于时间窗口（你定义的针对每条数据特定的原则）选择待集合的数据项。你将这些时间窗口实现为一些Observables，它们的生命周期从任何一条Observable发射的每一条数据开始。当这个定义时间窗口的Observable发射了一条数据或者完成时，与这条数据关联的窗口也会关闭。只要这条数据的窗口是打开的，它将继续结合其它Observable发射的任何数据项。你定义一个用于结合数据的函数。
join默认不在任何特定的调度器上执行。
Javadoc: Join(Observable,Func1,Func1,Func2))
groupJoin默认不在任何特定的调度器上执行。
Javadoc: groupJoin(Observable,Func1,Func1,Func2))

##  Merge 
合并多个Observables的发射物
![](/data/dokuwiki/opensourcelearn/pasted/20160311-105757.png)
**使用Merge操作符你可以将多个Observables的输出合并，就好像它们是一个单个的Observable一样。**
Merge可能会让合并的Observables发射的数据交错（有一个类似的操作符Concat不会让数据交错，它会按顺序一个接着一个发射多个Observables的发射物）。+
正如图例上展示的，任何一个原始Observable的onError通知会被立即传递给观察者，而且会终止合并后的Observable。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-105840.png)
**在很多ReactiveX实现中还有一个叫MergeDelayError的操作符，它的行为有一点不同，它会保留onError通知直到合并后的Observable所有的数据发射完成，在那时它才会把onError传递给观察者。+**
**RxJava将它实现为merge, mergeWith和mergeDelayError。**
示例代码
```

Observable<Integer> odds = Observable.just(1, 3, 5).subscribeOn(someScheduler);
Observable<Integer> evens = Observable.just(2, 4, 6);
Observable.merge(odds, evens)
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
输出
Next: 1
Next: 3
Next: 5
Next: 2
Next: 4
Next: 6
Sequence complete.
##  StartWith 
在数据序列的开头插入一条指定的项
![](/data/dokuwiki/opensourcelearn/pasted/20160311-110123.png)
如果你想要一个Observable在发射数据之前先发射一个指定的数据序列，可以使用StartWith操作符。（如果你想一个Observable发射的数据末尾追加一个数据序列可以使用Concat操作符。）
![](/data/dokuwiki/opensourcelearn/pasted/20160311-110141.png)
##  Switch 
将一个发射多个Observables的Observable转换成另一个单独的Observable，后者发射那些Observables最近发射的数据项
![](/data/dokuwiki/opensourcelearn/pasted/20160311-110342.png)
Switch订阅一个发射多个Observables的Observable。**它每次观察那些Observables中的一个**，Switch返回的这个Observable取消订阅前一个发射数据的Observable，开始发射最近的Observable发射的数据。注意：当原始Observable发射了一个新的Observable时（不是这个新的Observable发射了一条数据时），它将取消订阅之前的那个Observable。**这意味着，在后来那个Observable产生之后到它开始发射数据之前的这段时间里，前一个Observable发射的数据将被丢弃（就像图例上的那个黄色圆圈一样）。**
Java将这个操作符实现为switchOnNext。它默认不在任何特定的调度器上执行。
##  Zip 
通过一个函数将多个Observables的发射物结合到一起，基于这个函数的结果为每个结合体发射单个数据项。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-110549.png)
Zip操作符返回一个Obversable，它使用这个函数按顺序结合两个或多个Observables发射的数据项，然后它发射这个函数返回的结果。它按照严格的顺序应用这个函数。它只发射与发射数据项最少的那个Observable一样多的数据。
**RxJava将这个操作符实现为zip和zipWith。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-110630.png)
zip和zipWith默认不在任何特定的操作符上执行。

