title: rxjava 

#  RxJava使用介绍 
项目地址：https://github.com/ReactiveX/RxJava
Wiki:https://github.com/ReactiveX/RxJava/wiki
JavaDoc:http://reactivex.io/RxJava/javadoc/
##  安装： 

```

Maven:
<dependency>
    <groupId>io.reactivex</groupId>
    <artifactId>rxjava</artifactId>
    <version>1.0.10</version>
</dependency>
Gradle:
compile 'io.reactivex:rxjava:1.0.10'
  
如果你想拷贝Jars：可以通过maven：mvn -f pom.xml dependency:copy-dependencies
  pom.xml
  <?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.netflix.rxjava.download</groupId>
    <artifactId>rxjava-download</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Simple POM to download rxjava and dependencies</name>
    <url>http://github.com/ReactiveX/RxJava</url>
    <dependencies>
        <dependency>
            <groupId>io.reactivex</groupId>
            <artifactId>rxjava</artifactId>
            <version>1.0.10</version>
            <scope/>
        </dependency>
    </dependencies>
</project>

```
##  使用介绍： 
```

public static void hello(String... names) {
    Observable.from(names).subscribe(new Action1<String>() {

        @Override
        public void call(String s) {
            System.out.println("Hello " + s + "!");
        }

    });
}

```
Observable.from可以对集合数组等参数**拆分为单个元素一一分派**
打印结果：
hello("Ben", "George");
Hello Ben!
Hello George!
##  Observables 
![](/data/dokuwiki/opensourcelearn/rxjava_rxandroid/pasted/20150531-070103.png)
###  Creating an Observable from an Existing Data Structure 
使用just()或者from()等从现有数据结构构造Observable
```

Observable<String> o = Observable.from("a", "b", "c");
list = [5, 6, 7, 8]
Observable<Integer> o = Observable.from(list);
Observable<String> o = Observable.just("one object");

```
然后Observable会同步地调用Subcribe的onNext()方法,完成时调用onCompleted()，发生错误时调用onError()

##  schedulers调度器 
参考：http://reactivex.io/documentation/scheduler.html
Schedulers工厂类可以返回多种Scheduler
![](/data/dokuwiki/opensourcelearn/rxjava_rxandroid/pasted/20150531-064449.png)
###  Default Schedulers for RxJava Observable Operators 
![](/data/dokuwiki/opensourcelearn/rxjava_rxandroid/pasted/20150531-064821.png)![](/data/dokuwiki/opensourcelearn/rxjava_rxandroid/pasted/20150531-064836.png)
###  Using Schedulers 
推荐方式:使用RxJava，你可以使用subscribeOn()指定观察者代码运行的线程，使用observerOn()指定订阅者运行的线程：
```

myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));

```

第二种方式：
```

worker = Schedulers.newThread().createWorker();
worker.schedule(new Action0() {

    @Override
    public void call() {
        yourWork();
    }

});
// some time later...
worker.unsubscribe();

someScheduler.schedule(someAction, 500, TimeUnit.MILLISECONDS);

```

#  subject 
![](/data/dokuwiki/opensourcelearn/rxjava_rxandroid/pasted/20150531-070416.png)
其余略。。。。
