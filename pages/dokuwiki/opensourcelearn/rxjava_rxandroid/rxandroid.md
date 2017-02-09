title: rxandroid 

#  RxAndroid使用介绍 
项目地址：https://github.com/ReactiveX/RxAndroid/
文档：https://github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module

特点：
  * provides a Scheduler that schedules an Observable on a given Android Handler thread, particularly the main UI thread
  * provides base Observer implementations that make guarantees w.r.t. to reliable and thread-safe use throughout Fragment and Activity life-cycle callbacks (coming soon)
  * provides reusable, self-contained reactive components for common Android use cases and UI concerns (coming soon)

安装：
```

Maven:
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxandroid</artifactId>
    <version>0.23.0</version>
</dependency>
Gradle:
compile 'io.reactivex:rxandroid:0.23.0'

```
##  使用介绍： 
###  Observing on the UI thread 
```

public class ReactiveFragment extends Fragment {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Observable.from("one", "two", "three", "four", "five")
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(/* an Observer */);
    }

```
With RxJava instead you would declare your Observable to be observed on the main thread:
This will execute the Observable on a new thread, and emit results through onNext on the main UI thread.
###  Observing on arbitrary threads 
```

new Thread(new Runnable() {
    @Override
    public void run() {
        final Handler handler = new Handler(); // bound to this thread
        Observable.from("one", "two", "three", "four", "five")
                .subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.handlerThread(handler))
                .subscribe(/* an Observer */)

        // perform work, ...
    }
}, "custom-thread-1").start();

```
This will execute the Observable on a new thread and emit results through onNext on "custom-thread-1". (This example is contrived since you could as well call observeOn(Schedulers.currentThread()) but it shall suffice to illustrate the idea.)
###  Fragment and Activity life-cycle 
```

// MyActivity
private Subscription subscription;

protected void onCreate(Bundle savedInstanceState) {
    this.subscription = observable.subscribe(this);
}

...

protected void onDestroy() {
    this.subscription.unsubscribe();
    super.onDestroy();
}

```
**This ensures that all references to the subscriber (the Activity) will be released as soon as possible, and no more notifications will arrive at the subscriber through onNext.**

One problem with this is that if the Activity is destroyed because of a change in screen orientation, the Observable will fire again in onCreate. You can prevent this by using the cache or replay Observable operators, while making sure the Observable somehow survives the Activity life-cycle (for instance, by storing it in a global cache, in a Fragment, etc.) You can use either operator to ensure that when the subscriber subscribes to an Observable that’s already “running,” items emitted by the Observable during the span when it was detached from the Activity will be “played back,” and any in-flight notifications from the Observable will be delivered as usual.
