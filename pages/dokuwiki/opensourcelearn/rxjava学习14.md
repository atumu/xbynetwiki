title: rxjava学习14 

#  RxJava学习之操作符学习十一连接操作 
这一节解释` ConnectableObservable ` 和它的子类以及它们的操作符：
  * ConnectableObservable.connect( ) — 指示一个**可连接的Observable**开始发射数据
  * Observable.publish( ) — 将一个Observable转换为一个可连接的Observable
  * Observable.replay( ) — 确保所有的订阅者看到相同的数据序列，即使它们在Observable开始发射数据之后才订阅
  * ConnectableObservable.refCount( ) — 让一个可连接的Observable表现得像一个普通的Observable
一个可连接的Observable与普通的Observable差不多，除了这一点：**可连接的Observable在被订阅时并不开始发射数据，只有在它的connect()被调用时才开始。用这种方法，你可以等所有的潜在订阅者都订阅了这个Observable之后才开始发射数据。**
![](/data/dokuwiki/opensourcelearn/pasted/20160311-141609.png)
RxJava中connect是` ConnectableObservable `接口的一个方法，使用publish操作符可以将一个普通的Observable转换为一个ConnectableObservable。
调用ConnectableObservable的connect方法会让它后面的Observable开始给发射数据给订阅者。
connect方法返回一个` Subscription对象 `，可以调用它的unsubscribe方法让Observable停止发射数据给观察者。
即使没有任何订阅者订阅它，你也可以使用connect方法让一个Observable开始发射数据（或者开始生成待发射的数据）。这样，你可以将一个"冷"的Observable变为"热"的。
