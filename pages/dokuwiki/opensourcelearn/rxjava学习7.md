title: rxjava学习7 

#  RxJava学习之操作符学习四过滤操作 
这个页面展示的操作符可**用于过滤和选择Observable发射的数据序列。**
  * filter( ) — 过滤数据
  * takeLast( ) — 只发射最后的N项数据
  * last( ) — 只发射最后的一项数据
  * lastOrDefault( ) — 只发射最后的一项数据，如果Observable为空就发射默认值
  * takeLastBuffer( ) — 将最后的N项数据当做单个数据发射
  * skip( ) — 跳过开始的N项数据
  * skipLast( ) — 跳过最后的N项数据
  * take( ) — 只发射开始的N项数据
  * first( ) and takeFirst( ) — 只发射第一项数据，或者满足某种条件的第一项数据
  * firstOrDefault( ) — 只发射第一项数据，如果Observable为空就发射默认值
  * elementAt( ) — 发射第N项数据
  * elementAtOrDefault( ) — 发射第N项数据，如果Observable数据少于N项就发射默认值
  * sample( ) or throttleLast( ) — 定期发射Observable最近的数据
  * throttleFirst( ) — 定期发射Observable发射的第一项数据
  * throttleWithTimeout( ) or debounce( ) — 只有当Observable在指定的时间后还没有发射数据时，才发射一个数据
  * timeout( ) — 如果在一个指定的时间段后还没发射数据，就发射一个异常
  * distinct( ) — 过滤掉重复数据
  * distinctUntilChanged( ) — 过滤掉连续重复的数据
  * ofType( ) — 只发射指定类型的数据
  * ignoreElements( ) — 丢弃所有的正常数据，只发射错误或完成通知

##  Debounce 
仅在过了一段指定的时间还没发射数据时才发射一个数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-101956.png)
Debounce操作符会过滤掉发射速率过快的数据项。
RxJava将这个操作符实现为throttleWithTimeout和debounce。
注意：这个操作符会会接着最后一项数据发射原始Observable的onCompleted通知，即使这个通知发生在你指定的时间窗口内（从最后一项数据的发射算起）。也就是说，onCompleted通知不会触发限流。
throtleWithTimeout/debounce的一个变体根据你指定的时间间隔进行限流，时间单位通过TimeUnit参数指定。
这种操作符默认在computation调度器上执行，但是你可以通过第三个参数指定。
Javadoc: throttleWithTimeout(long,TimeUnit)) and debounce(long,TimeUnit))
Javadoc: throttleWithTimeout(long,TimeUnit,Scheduler)) and debounce(long,TimeUnit,Scheduler))
##  Distinct 
抑制（过滤掉）重复的数据项
![](/data/dokuwiki/opensourcelearn/pasted/20160311-102220.png)
Distinct的过滤规则是：只允许还没有发射过的数据项通过。
在某些实现中，有一些变体允许你调整判定两个数据不同(distinct)的标准。还有一些实现只比较一项数据和它的直接前驱，因此只会从序列中过滤掉连续重复的数据。
RxJava将这个操作符实现为distinct函数。
示例代码
```

Observable.just(1, 2, 1, 1, 2, 3)
          .distinct()
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
输出：
Next: 1
Next: 2
Next: 3
Sequence complete.
这个操作符有一个变体distinct(Func1)接受一个函数函数。这个函数根据原始Observable发射的数据项产生一个Key，然后，比较这些Key而不是数据本身，来判定两个数据是否是不同的。
###  distinctUntilChanged 
RxJava还是实现了一个distinctUntilChanged操作符。它只判定一个数据和它的直接前驱是否是不同的。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-102437.png)
和distinct(Func1)一样，根据一个函数产生的Key判定两个相邻的数据项是不是不同的。
Javadoc: distinctUntilChanged(Func1))
**distinct和distinctUntilChanged默认不在任何特定的调度器上执行。**

##  ElementAt 
只发射第N项数据。ElementAt操作符获取原始Observable发射的数据序列指定索引位置的数据项，**然后当做自己的唯一数据发射。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-102551.png)
RxJava将这个操作符实现为elementAt，给它传递一个**基于0的索引值**，它会发射原始Observable数据序列对应索引位置的值，如果你传递给elementAt的值为5，那么它会发射第六项的数据。
如果你传递的是一个负数，或者原始Observable的数据项数小于index+1，将会抛出一个IndexOutOfBoundsException异常。
Javadoc: elementAt(int))
###  elementAtOrDefault 
RxJava还实现了elementAtOrDefault操作符。与elementAt的区别是，如果索引值大于数据项数，它会发射一个默认值（通过额外的参数指定），而不是抛出异常。但是如果你传递一个负数索引值，它仍然会抛出一个IndexOutOfBoundsException异常。
Javadoc: elementAtOrDefault(int,T))
**elementAt和elementAtOrDefault默认不在任何特定的调度器上执行。**

##  Filter 
只发射通过了谓词测试的数据项
![](/data/dokuwiki/opensourcelearn/pasted/20160311-102731.png)
RxJava将这个操作符实现为filter函数。
```

Observable.just(1, 2, 3, 4, 5)
          .filter(new Func1<Integer, Boolean>() {
              @Override
              public Boolean call(Integer item) {
                return( item < 4 );
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
Next: 2
Next: 3
Sequence complete.
**filter默认不在任何特定的调度器上执行。**
Javadoc: filter(Func1))

###  ofType 
**ofType是filter操作符的一个特殊形式。它过滤一个Observable只返回指定类型的数据。**
ofType默认不在任何特定的调度器上指定。
Javadoc: ofType(Class))

##  First 
只发射第一项（或者满足某个条件的第一项）数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-102920.png)
**在RxJava中，这个操作符被实现为first，firstOrDefault和takeFirst。**
可能容易混淆，BlockingObservable也有名叫first和firstOrDefault的操作符，**它们会阻塞并返回值，不是立即返回一个Observable。**
还有几个其它的操作符执行类似的功能。
```

Observable.just(1, 2, 3)
          .first()
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
Sequence complete.
传递一个谓词函数给first，然后发射这个函数判定为true的第一项数据。
first(Func1)
###  firstOrDefault 
firstOrDefault与first类似，但是在Observagle没有发射任何数据时发射一个你在参数中指定的默认值。
firstOrDefault(T, Func1))传递一个谓词函数给firstOrDefault，然后发射这个函数判定为true的第一项数据，如果没有数据通过了谓词测试就发射一个默认值。
###  takeFirst 
takeFirst与first类似，除了这一点：如果原始Observable没有发射任何满足条件的数据，first会抛出一个NoSuchElementException，takeFist会返回一个空的Observable（不调用onNext()但是会调用onCompleted）。
Javadoc: takeFirst(Func1))
###  single 
![](/data/dokuwiki/opensourcelearn/pasted/20160311-103646.png)
single操作符也与first类似，但是如果原始Observable在完成之前不是正好发射一次数据，它会抛出一个NoSuchElementException。
first系列的这几个操作符默认不在任何特定的调度器上执行。
##  IgnoreElements 
**不发射任何数据，只发射Observable的终止通知**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-103754.png)
IgnoreElements操作符**抑制原始Observable发射的所有数据**，**只允许它的终止通知（onError或onCompleted）通过。**
如果你不关心一个Observable发射的数据，但是希望在它完成时或遇到错误终止时收到通知，你可以对Observable使用ignoreElements操作符，它会确保永远不会调用观察者的onNext()方法。
RxJava将这个操作符实现为ignoreElements。
Javadoc: ignoreElements())
ignoreElements默认不在任何特定的调度器上执行。
##  Last 
只发射最后一项（或者满足某个条件的最后一项）数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-103921.png)
在RxJava中的实现是last和lastOrDefault。
##  Sample 
**定期发射Observable最近发射的数据项**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-104135.png)
Sample操作符定时查看一个Observable，然后发射自上次采样以来它最近发射的数据。
**RxJava将这个操作符实现为sample和throttleLast。**
注意：如果自上次采样以来，原始Observable没有发射任何数据，这个操作返回的Observable在那段时间内也不会发射任何数据
sample(别名throttleLast)的一个变体按照你参数中**指定的时间间隔定时采样**（TimeUnit指定时间单位）。
sample的这个变体默认在computation调度器上执行，但是你可以使用第三个参数指定其它的调度器。
Javadoc: sample(long,TimeUnit))和throttleLast(long,TimeUnit))
Javadoc: sample(long,TimeUnit,Scheduler))和throttleLast(long,TimeUnit,Scheduler))
![](/data/dokuwiki/opensourcelearn/pasted/20160311-104350.png)
**sample的这个变体每当第二个Observable发射一个数据（或者当它终止）时就对原始Observable进行采样。**第二个Observable通过参数传递给sample。
sample的这个变体默认不在任何特定的调度器上执行。
Javadoc: sample(Observable))
![](/data/dokuwiki/opensourcelearn/pasted/20160311-104343.png)

##  Skip 
**抑制Observable发射的前N项数据**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-104429.png)
RxJava中这个操作符叫skip。skip的这个变体默认不在任何特定的调度器上执行。
###  SkipLast 
抑制Observable发射的后N项数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-104536.png)
##  Take 
**只发射前面的N项数据**，然后发射完成通知，忽略剩余的数据。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-104630.png)
RxJava将这个操作符实现为take函数。
如果你对一个Observable使用take(n)（或它的同义词limit(n)）操作符，而那个Observable发射的数据少于N项，那么take操作生成的Observable不会抛异常或发射onError通知，在完成前它只会发射相同的少量数据。
```

Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
          .take(4)
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
Next: 2
Next: 3
Next: 4
Sequence complete.
take(int)默认不任何特定的调度器上执行。

###  TakeLast 
发射Observable发射的最后N项数据
