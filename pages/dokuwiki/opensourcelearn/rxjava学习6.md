title: rxjava学习6 

#  RxJava学习之操作符学习三变换操作 
这个页面展示了**可用于对Observable发射的数据执行变换操作的各种操作符。**
  * **map( ) — 对序列的每一项都应用一个函数来变换Observable发射的数据序列**
  * flatMap( ), concatMap( ), and flatMapIterable( ) — 将Observable发射的数据集合变换为Observables集合，然后将这些Observable发射的数据平坦化的放进一个单独的Observable
  * switchMap( ) — 将Observable发射的数据集合变换为Observables集合，然后只发射这些Observables最近发射的数据
  * scan( ) — 对Observable发射的每一项数据应用一个函数，然后按顺序依次发射每一个值
  * groupBy( ) — 将Observable分拆为Observable集合，将原始Observable发射的数据按Key分组，每一个Observable发射一组不同的数据
  * buffer( ) — 它定期从Observable收集数据到一个集合，然后把这些数据集合打包发射，而不是一次发射一个
  * window( ) — 定期将来自Observable的数据分拆成一些Observable窗口，然后发射这些窗口，而不是每次发射一项
  * cast( ) — 在发射之前强制将Observable发射的所有数据转换为指定类型

##  Buffer(定期收集数据，一次发送) 
**定期收集Observable的数据放进一个数据包裹，然后发射这些数据包裹，而不是一次发射一个值。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-094239.png)
Buffer操作符将一个Observable变换为另一个，原来的Observable正常发射数据，变换产生的Observable发射这些数据的缓存集合。Buffer操作符在很多语言特定的实现中有很多种变体，它们在如何缓存这个问题上存在区别。
**注意：如果原来的Observable发射了一个onError通知，Buffer会立即传递这个通知，而不是首先发射缓存的数据，**即使在这之前缓存中包含了原始Observable发射的数据。
Window操作符与Buffer类似，但是它在发射之前把收集到的数据放进单独的Observable，而不是放进一个数据结构。
在RxJava中有许多Buffer的变体：
  * **buffer(count)**以列表(List)的形式发射非重叠的缓存，每一个缓存至多包含来自原始Observable的count项数据（最后发射的列表数据可能少于count项）
  * **buffer(count, skip)**从原始Observable的第一项数据开始创建新的缓存，此后每当收到skip项数据，**用count项数据填充缓存：开头的一项和后续的count-1项**，它以列表(List)的形式发射缓存，取决于count和skip的值，这些缓存可能会有重叠部分（比如skip < count时），也可能会有间隙（比如skip > count时）。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-094712.png)
  * **buffer(bufferClosingSelector)**当它订阅原来的Observable时，buffer(bufferClosingSelector)开始将数据收集到一个List，然后它调用bufferClosingSelector生成第二个Observable，当第二个Observable发射一个TClosing时，buffer发射当前的List，然后重复这个过程：开始组装一个新的List，然后调用bufferClosingSelector创建一个新的Observable并监视它。它会一直这样做直到原来的Observable执行完成。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-094721.png)
  * **buffer(boundary)**监视一个名叫boundary的Observable，每当这个Observable发射了一个值，它就创建一个新的List开始收集来自原始Observable的数据并发射原来的List。
buffer(bufferOpenings, bufferClosingSelector)监视这个叫bufferOpenings的Observable（它发射BufferOpening对象），每当bufferOpenings发射了一个数据时，它就创建一个新的List开始手机原始Observable的数据，并将bufferOpenings传递给closingSelector函数。这个函数返回一个Observable。buffer监视这个Observable，当它检测到一个来自这个Observable的数据时，就关闭List并且发射它自己的数据（之前的那个List）。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-094915.png)
  * buffer(timespan, unit[, scheduler])定期以List的形式发射新的数据，每个时间段，收集来自原始Observable的数据（从前面一个数据包裹之后，或者如果是第一个数据包裹，从有观察者订阅原来的Observale之后开始）。还有另一个版本的buffer接受一个Scheduler参数，默认情况下会使用computation调度器。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-095137.png)
  * buffer(timespan, unit, count[, scheduler])每当收到来自原始Observable的count项数据，或者每过了一段指定的时间后，buffer(timespan, unit, count)就以List的形式发射这期间的数据，即使数据项少于count项。还有另一个版本的buffer接受一个Scheduler参数，默认情况下会使用computation调度器。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-095308.png)
  * buffer(timespan, timeshift, unit[, scheduler])在每一个timeshift时期内都创建一个新的List,然后用原始Observable发射的每一项数据填充这个列表（在把这个List当做自己的数据发射前，从创建时开始，直到过了timespan这么长的时间）。如果timespan长于timeshift，它发射的数据包将会重叠，因此可能包含重复的数据项。还有另一个版本的buffer接受一个Scheduler参数，默认情况下会使用computation调度器。
###  buffer-backpressure 
你可以使用Buffer操作符实现反压backpressure（意思是，处理这样一个Observable：它产生数据的速度可能比它的观察者消费数据的速度快）。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-095957.png)
Buffer操作符可以将大量的数据序列缩减为较少的数据缓存序列，让它们更容易处理。例如，你可以按固定的时间间隔，定期关闭和发射来自一个爆发性Observable的数据缓存。这相当于一个缓冲区。+
示例代码Observable<List<Integer>> burstyBuffered = bursty.buffer(500, TimeUnit.MILLISECONDS);
![](/data/dokuwiki/opensourcelearn/pasted/20160311-100019.png)
或者，如果你想更进一步，可以在爆发期将数据收集到缓存，然后在爆发期终止时发射这些数据，使用 Debounce 操作符给buffer操作符发射一个缓存关闭指示器(buffer closing indicator)可以做到这一点。
代码示例：
```

// we have to multicast the original bursty Observable so we can use it
// both as our source and as the source for our buffer closing selector:
Observable<Integer> burstyMulticast = bursty.publish().refCount();
// burstyDebounced will be our buffer closing selector:
Observable<Integer> burstyDebounced = burstyMulticast.debounce(10, TimeUnit.MILLISECONDS);
// and this, finally, is the Observable of buffers we're interested in:
Observable<List<Integer>> burstyBuffered = burstyMulticast.buffer(burstyDebounced);

```

##  FlatMap 
FlatMap将一个发射数据的Observable变换为多个Observables，然后将它们发射的数据合并后放进一个单独的Observable
![](/data/dokuwiki/opensourcelearn/pasted/20160311-100149.png)
**FlatMap操作符使用一个指定的函数对原始Observable发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的Observable，然后FlatMap合并这些Observables发射的数据，最后将合并后的结果当做它自己的数据序列发射。**
这个方法是很有用的，例如，当你有一个这样的Observable：它发射一个数据序列，这些数据本身包含Observable成员或者可以变换为Observable，因此你可以创建一个新的Observable发射这些次级Observable发射的数据的完整集合。
**注意：FlatMap对这些Observables发射的数据做的是合并(merge)操作，因此它们可能是交错的。**
在许多语言特定的实现中，还有一个操作符不会让变换后的Observables发射的数据交错，它按照严格的顺序发射这些数据，这个操作符通常被叫作` ConcatMap `或者类似的名字。
RxJava将这个操作符实现为flatMap函数。
注意：如果任何一个通过这个flatMap操作产生的单独的Observable调用onError异常终止了，这个Observable自身会立即调用onError并终止。
这个操作符有一个接受额外的int参数的一个变体。这个参数设置flatMap从原来的Observable映射Observables的最大同时订阅数。当达到这个限制时，它会等待其中一个终止然后再订阅另一个。
Javadoc: flatMap(Func1))
Javadoc: flatMap(Func1,int))
还有一个版本的flatMap为原始Observable的每一项数据和每一个通知创建一个新的Observable（并对数据平坦化）。
它也有一个接受额外int参数的变体。
Javadoc: flatMap(Func1,Func1,Func0))
Javadoc: flatMap(Func1,Func1,Func0,int))
还有一个版本的flatMap会使用原始Observable的数据触发的Observable组合这些数据，然后发射这些数据组合。它也有一个接受额外int参数的版本。
Javadoc: flatMap(Func1,Func2))
Javadoc: flatMap(Func1,Func2,int))
flatMapIterable这个变体成对的打包数据，然后生成Iterable而不是原始数据和生成的Observables，但是处理方式是相同的。
Javadoc: flatMapIterable(Func1))
Javadoc: flatMapIterable(Func1,Func2))
还有一个concatMap操作符，它类似于最简单版本的flatMap，但是它按次序连接而不是合并那些生成的Observables，然后产生自己的数据序列。
RxJava还实现了switchMap操作符。它和flatMap很像，除了一点：当原始Observable发射一个新的数据（Observable）时，它将取消订阅并停止监视产生执之前那个数据的Observable，只监视当前这一个。
Javadoc: switchMap(Func1))
##  GroupBy 
将一个Observable分拆为一些Observables集合，它们中的每一个发射原始Observable的一个子序列
![](/data/dokuwiki/opensourcelearn/pasted/20160311-100635.png)
GroupBy操作符将原始Observable分拆为一些Observables集合，它们中的每一个发射原始Observable数据序列的一个子序列。哪个数据项由哪一个Observable发射是由一个函数判定的，这个函数给每一项指定一个Key，Key相同的数据会被同一个Observable发射。
**RxJava实现了groupBy操作符。它返回Observable的一个特殊子类GroupedObservable，实现了GroupedObservable接口的对象有一个额外的方法getKey，这个Key用于将数据分组到指定的Observable。**
有一个版本的groupBy允许你传递一个变换函数，这样它可以在发射结果GroupedObservable之前改变数据项。
注意：groupBy将原始Observable分解为一个发射多个GroupedObservable的Observable，一旦有订阅，每个GroupedObservable就开始缓存数据。因此，如果你忽略这些GroupedObservable中的任何一个，这个缓存可能形成一个潜在的内存泄露。因此，如果你不想观察，也不要忽略GroupedObservable。你应该使用像take(0)这样会丢弃自己的缓存的操作符。
如果你取消订阅一个GroupedObservable，那个Observable将会终止。如果之后原始的Observable又发射了一个与这个Observable的Key匹配的数据，groupBy将会为这个Key创建一个新的GroupedObservable。
groupBy默认不在任何特定的调度器上执行。
Javadoc: groupBy(Func1))
Javadoc: groupBy(Func1,Func1))

##  Map(每一项数据应用一个函数) 
对Observable发射的每一项数据应用一个函数，执行变换操作
![](/data/dokuwiki/opensourcelearn/pasted/20160311-100954.png)
Map操作符对原始Observable发射的每一项数据应用一个你选择的函数，然后返回一个发射这些结果的Observable。
RxJava将这个操作符实现为map函数。这个操作符默认不在任何特定的调度器上执行。
Javadoc: map(Func1))
###  cast 
cast操作符将原始Observable发射的每一项数据都强制转换为一个指定的类型，然后再发射数据，它是map的一个特殊版本。+
Javadoc: cast(Class))
##  Scan 
连续地对数据序列的每一项应用一个函数，然后**连续发射**结果
![](/data/dokuwiki/opensourcelearn/pasted/20160311-101231.png)
Scan操作符对原始Observable发射的第一项数据应用一个函数，然后将那个函数的结果作为自己的第一项数据发射。它将函数的结果同第二项数据一起填充给这个函数来产生它自己的第二项数据。它持续进行这个过程来产生剩余的数据序列。这个操作符在某些情况下被叫做accumulator。
RxJava实现了scan操作符。
示例代码：
```

Observable.just(1, 2, 3, 4, 5)
    .scan(new Func2<Integer, Integer, Integer>() {
        @Override
        public Integer call(Integer sum, Integer item) {
            return sum + item;
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
Next: 3
Next: 6
Next: 10
Next: 15
Sequence complete.
有一个scan操作符的变体，你可以传递一个种子值给累加器函数的第一次调用（Observable发射的第一项数据）。如果你使用这个版本，scan将发射种子值作为自己的第一项数据。注意：传递null作为种子值与不传递是不同的，null种子值是合法的。
Javadoc: scan(R,Func2))
这个操作符默认不在任何特定的调度器上执行。

##  Window 
定期将来自原始Observable的数据分解为一个Observable窗口，发射这些窗口，而不是每次发射一项数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-101534.png)
**Window和Buffer类似，但不是发射来自原始Observable的数据包，它发射的是Observables，这些Observables中的每一个都发射原始Observable数据的一个子集，最后发射一个onCompleted通知。**
和Buffer一样，Window有很多变体，每一种都以自己的方式将原始Observable分解为多个作为结果的Observable，每一个都包含一个映射原始数据的window。用Window操作符的术语描述就是，当一个窗口打开(when a window "opens")意味着一个新的Observable已经发射（产生）了，而且这个Observable开始发射来自原始Observable的数据；当一个窗口关闭(when a window "closes")意味着发射(产生)的Observable停止发射原始Observable的数据，并且发射终止通知onCompleted给它的观察者们。
在RxJava中有许多种Window操作符的变体。
