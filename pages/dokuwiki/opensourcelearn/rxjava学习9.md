title: rxjava学习9 

#  RxJava学习之操作符学习六错误处理 
很多操作符**可用于对Observable发射的onError通知做出响应或者从错误中恢复**，例如，你可以：
  * 吞掉这个错误，切换到一个备用的Observable继续发射数据
  * 吞掉这个错误然后发射默认值
  * 吞掉这个错误并立即尝试重启这个Observable
  * 吞掉这个错误，在一些回退间隔后重启这个Observable
这是操作符列表：
  * onErrorResumeNext( ) — 指示Observable在遇到错误时发射一个数据序列
  * onErrorReturn( ) — 指示Observable在遇到错误时发射一个特定的数据
  * onExceptionResumeNext( ) — instructs an Observable to continue emitting items after it encounters an exception (but not another variety of throwable)指示Observable遇到错误时继续发射数据
  * retry( ) — 指示Observable遇到错误时重试
  * retryWhen( ) — 指示Observable遇到错误时，将错误传递给另一个Observable来决定是否要重新给订阅这个Observable
##  Catch 
从onError通知中恢复发射数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-110935.png)
Catch操作符拦截原始Observable的onError通知，将它替换为其它的数据项或数据序列，让产生的Observable能够正常终止或者根本不终止。
在某些ReactiveX的实现中，有一个叫onErrorResumeNext的操作符，它的行为与Catch相似。
**RxJava将Catch实现为三个不同的操作符：**
  * onErrorReturn让Observable遇到错误时发射一个特殊的项并且正常终止。
  * onErrorResumeNext让Observable在遇到错误时开始发射第二个Observable的数据序列。
  * onExceptionResumeNext让Observable在遇到错误时继续发射后面的数据项

##  Retry 
如果原始Observable遇到错误，重新订阅它期望它能正常终止
![](/data/dokuwiki/opensourcelearn/pasted/20160311-111119.png)
Retry操作符不会将原始Observable的onError通知传递给观察者，它会订阅这个Observable，再给它一次机会无错误地完成它的数据序列。Retry总是传递onNext通知给观察者，由于重新订阅，可能会造成数据项重复，如上图所示。
**RxJava中的实现为retry和retryWhen。**
  * 无论收到多少次onError通知，无参数版本的retry都会继续订阅并发射原始Observable。
  * 接受单个count参数的retry会最多重新订阅指定的次数，如果次数超了，它不会尝试再次订阅，它会把最新的一个onError通知传递给它的观察者。
  * 还有一个版本的retry接受一个谓词函数作为参数，这个函数的两个参数是：重试次数和导致发射onError通知的Throwable。这个函数返回一个布尔值，如果返回true，retry应该再次订阅和镜像原始的Observable，如果返回false，retry会将最新的一个onError通知传递给它的观察者。
**retry操作符默认在trampoline调度器上执行。**
