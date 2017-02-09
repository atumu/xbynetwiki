title: rxjava学习17 

#  RxJava学习之自定义操作符 
将你的自定义操作符定义为实现了`  Operator 接口 `的一个公开类, 就像这样：
```

public class MyOperator<T> implements Operator<T> {
  public MyOperator( /* any necessary params here */ ) {
    /* 这里添加必要的初始化代码 */
  }

  @Override
  public Subscriber<? super T> call(final Subscriber<? super T> s) {
    return new Subscriber<t>(s) {
      @Override
      public void onCompleted() {
        /* 这里添加你自己的onCompleted行为，或者仅仅传递完成通知： */
        if(!s.isUnsubscribed()) {
          s.onCompleted();
        }
      }

      @Override
      public void onError(Throwable t) {
        /* 这里添加你自己的onError行为, 或者仅仅传递错误通知：*/
        if(!s.isUnsubscribed()) {
          s.onError(t);
        }
      }

      @Override
      public void onNext(T item) {
        /* 这个例子对结果的每一项执行排序操作，然后返回这个结果 */
        if(!s.isUnsubscribed()) {
          transformedItem = myOperatorTransformOperation(item);
          s.onNext(transformedItem);
        }
      }
    };
  }
}

```
实现你的变换器
将你的自定义操作符定义为实现了 ` Transformer 接口 `的一个公开类，就像这样：
```

public class MyTransformer<Integer,String> implements Transformer<Integer,String> {
  public MyTransformer( /* any necessary params here */ ) {
    /* 这里添加必要的初始化代码 */
  }

  @Override
  public Observable<String> call(Observable<Integer> source) {
    /* 
     * 这个简单的例子Transformer应用一个map操作，
     * 这个map操作将发射整数变换为发射整数的字符串表示。
     */
    return source.map( new Func1<Integer,String>() {
      @Override
      public String call(Integer t1) {
        return String.valueOf(t1);
      }
    } );
  }
}

```
具体请参考：https://mcxiaoke.gitbooks.io/rxdocs/content/topics/Implementing-Your-Own-Operators.html