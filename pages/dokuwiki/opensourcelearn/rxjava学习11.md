title: rxjava学习11 

#  RxJava学习之操作符学习八条件和布尔操作 
这个页面的操作符可用于根据条件发射或变换Observables，或者对它们做布尔运算：
##  条件操作符 
  * amb( ) — 给定多个Observable，只让第一个发射数据的Observable发射全部数据
  * defaultIfEmpty( ) — 发射来自原始Observable的数据，如果原始Observable没有发射数据，就发射一个默认数据
  * (rxjava-computation-expressions) doWhile( ) — 发射原始Observable的数据序列，然后重复发射这个序列直到不满足这个条件为止
  * (rxjava-computation-expressions) ifThen( ) — 只有当某个条件为真时才发射原始Observable的数据序列，否则发射一个空的或默认的序列
  * skipUntil( ) — 丢弃原始Observable发射的数据，直到第二个Observable发射了一个数据，然后发射原始Observable的剩余数据
  * skipWhile( ) — 丢弃原始Observable发射的数据，直到一个特定的条件为假，然后发射原始Observable剩余的数据
  * (rxjava-computation-expressions) switchCase( ) — 基于一个计算结果，发射一个指定Observable的数据序列
  * takeUntil( ) — 发射来自原始Observable的数据，直到第二个Observable发射了一个数据或一个通知
  * takeWhile( ) and takeWhileWithIndex( ) — 发射原始Observable的数据，直到一个特定的条件为真，然后跳过剩余的数据
  * (rxjava-computation-expressions) whileDo( ) — if a condition is true, emit the source Observable's sequence and then repeat the sequence as long as the condition remains true如果满足一个条件，发射原始Observable的数据，然后重复发射直到不满足这个条件为止
  * (rxjava-computation-expressions) — 表示这个操作符当前是可选包 rxjava-computation-expressions 的一部分，还没有包含在标准RxJava的操作符集合里
##  布尔操作符 
  * all( ) — 判断是否所有的数据项都满足某个条件
  * contains( ) — 判断Observable是否会发射一个指定的值
  * exists( ) and isEmpty( ) — 判断Observable是否发射了一个值
  * sequenceEqual( ) — test the equality of the sequences emitted by two Observables判断两个Observables发射的序列是否相等
##  All 
判定是否Observable发射的所有数据都满足某个条件
![](/data/dokuwiki/opensourcelearn/pasted/20160311-134952.png)
传递一个谓词函数给All操作符，这个函数接受原始Observable发射的数据，根据计算返回一个布尔值。
**All返回一个只发射一个单个布尔值的Observable**，如果原始Observable正常终止并且每一项数据都满足条件，就返回true；如果原始Observable的任何一项数据不满足条件就返回False。
RxJava将这个操作符实现为all，它默认不在任何特定的调度器上执行。
##  Amb 
给定两个或多个Observables，它只发射首先发射数据(或通知)的那个Observable的所有数据
当你传递多个Observable给Amb时，**它只发射其中一个Observable的数据和通知：首先发送通知给Amb的那个，**不管发射的是一项数据还是一个onError或onCompleted通知。Amb将忽略和丢弃其它所有Observables的发射物。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-135147.png)
RxJava的实现是amb，有一个类似的对象方法ambWith。例如，Observable.amb(o1,o2)和o1.ambWith(o2)是等价的。
这个操作符默认不在任何特定的调度器上执行。
##  Contains、IsEmpty、exists、DefaultIfEmpty 
判定一个Observable是否发射一个特定的值
![](/data/dokuwiki/opensourcelearn/pasted/20160311-135300.png)
给Contains传一个指定的值，如果原始Observable发射了那个值，它返回的Observable将发射true，否则发射false。
相关的一个操作符**IsEmpty用于判定原始Observable是否没有发射任何数据。**isEmpty默认不在任何特定的调度器上执行。
contains默认不在任何特定的调度器上执行。
**RxJava中还有一个exists操作符，它通过一个谓词函数测试原始Observable发射的数据，只要任何一项满足条件就返回一个发射true的Observable，否则返回一个发射false的Observable。+**
exists默认不在任何特定的调度器上执行。
**DefaultIfEmpty**发射来自原始Observable的值，如果原始Observable没有发射任何值，就发射一个默认值。DefaultIfEmpty简单的精确地发射原始Observable的值，如果原始Observable没有发射任何数据正常终止（以onCompletedd的形式），DefaultIfEmpty返回的Observable就发射一个你提供的默认值。RxJava将这个操作符实现为defaultIfEmpty。它默认不在任何特定的调度器上执行。
##  SequenceEqual 
判定两个Observables是否发射相同的数据序列。
![](/data/dokuwiki/opensourcelearn/pasted/20160311-135621.png)
传递两个Observable给SequenceEqual操作符，它会比较两个Observable的发射物，**如果两个序列是相同的（相同的数据，相同的顺序，相同的终止状态），它就发射true**，否则发射false。
它还有一个版本接受第三个参数，可以传递一个函数用于比较两个数据项是否相同。
这个操作符默认不在任何特定的调度器上执行。
##  SkipUntil 
丢弃原始Observable发射的数据，直到第二个Observable发射了一项数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-135821.png)
SkipUntil订阅原始的Observable，但是忽略它的发射物，直到第二个Observable发射了一项数据那一刻，它开始发射原始Observable。
RxJava中对应的是skipUntil，它默认不在任何特定的调度器上执行。
###  SkipWhile 
丢弃Observable发射的数据，直到一个指定的条件不成立
![](/data/dokuwiki/opensourcelearn/pasted/20160311-135922.png)
SkipWhile订阅原始的Observable，但是忽略它的发射物，直到你指定的某个天剑变为false的那一刻，它开始发射原始Observable。+
skipWhile默认不在任何特定的调度器上执行。
##  TakeUntil 
与SkipUntil相反
当第二个Observable发射了一项数据或者终止时，丢弃原始Observable发射的任何数据
![](/data/dokuwiki/opensourcelearn/pasted/20160311-140015.png)
TakeUntil订阅并开始发射原始Observable，它还监视你提供的第二个Observable。如果第二个Observable发射了一项数据或者发射了一个终止通知，TakeUntil返回的Observable会停止发射原始Observable并终止。
RxJava中的实现是takeUntil。注意：第二个Observable发射一项数据或一个onError通知或一个onCompleted通知都会导致takeUntil停止发射数据。+
takeUntil默认不在任何特定的调度器上执行。
###  TakeWhile 
与SkipWhile相反
发射Observable发射的数据，直到一个指定的条件不成立
TakeWhile发射原始Observable，直到你指定的某个条件不成立的那一刻，它停止发射原始Observable，并终止自己的Observable。
RxJava中的takeWhile操作符返回一个镜像原始Observable行为的Observable，直到某一项数据你指定的函数返回false那一刻，这个新的Observable发射onCompleted终止通知。
takeWhile默认不在任何特定的调度器上执行。