title: rxjava学习15 

#  RxJava学习之操作符学习十二阻塞操作 
这一节解释 ` BlockingObservable ` 的子类.** 一个阻塞的Observable 继承普通的Observable类，增加了一些可用于阻塞Observable发射的数据的操作符。**
要将普通的Observable 转换为 BlockingObservable，可以使用`  Observable.toBlocking( )) 方法或者BlockingObservable.from( )) ` 方法。
  * forEach( ) — 对Observable发射的每一项数据调用一个方法，会阻塞直到Observable完成
  * first( ) — 阻塞直到Observable发射了一个数据，然后返回第一项数据
  * firstOrDefault( ) — 阻塞直到Observable发射了一个数据或者终止，返回第一项数据，或者返回默认值
  * last( ) — 阻塞直到Observable终止，然后返回最后一项数据
  * lastOrDefault( ) — 阻塞直到Observable终止，然后返回最后一项的数据，或者返回默认值
  * mostRecent( ) — 返回一个总是返回Observable最近发射的数据的iterable
  * next( ) — 返回一个Iterable，会阻塞直到Observable发射了另一个值，然后返回那个值
  * latest( ) — 返回一个iterable，会阻塞直到或者除非Observable发射了一个iterable没有返回的值，然后返回这个值
  * single( ) — 如果Observable终止时只发射了一个值，返回那个值，否则抛出异常
  * singleOrDefault( ) — 如果Observable终止时只发射了一个值，返回那个值，否则否好默认值
  * toFuture( ) — 将Observable转换为一个Future
  * toIterable( ) — 将一个发射数据序列的Observable转换为一个Iterable
  * getIterator( ) — 将一个发射数据序列的Observable转换为一个Iterator

![](/data/dokuwiki/opensourcelearn/pasted/20160311-142203.png)
BlockingObservable的方法
BlockingObservable的方法不是将一个Observable变换为另一个，也不是过滤Observables，**它们会打断Observable的调用链，会阻塞等待直到Observable发射了想要的数据，然后返回这个数据（而不是一个Observable）。**
要将一个Observable转换为一个BlockingObservable，你可以使用Observable.toBlocking或BlockingObservable.from方法。
