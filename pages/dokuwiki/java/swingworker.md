title: swingworker 

#  SwingWorker 
参考：http://blog.csdn.net/vking_wang/article/details/8994882
还可以参考http://blog.csdn.net/xfilson/article/details/6527866
使用SwingWorker，程序能启动一个任务线程来异步查询，并马上返回EDT线程，允许EDT继续执行后续的UI事件。
SwingWoker实现了java.util.concurrent.RunnableFuture接口。RunnableFuture接口是Runnable和Future两个接口的简单封装。
SwingWorker有两个类型参数：**T及V**。T是doInBackground和get方法的返回类型；V是publish和process方法要处理的数据类型
` **SwingWorker实例不可复用，每次执行任务必须生成新的实例。** `

` doInBackground `方法作为任务线程的一部分执行，它负责完成线程的基本任务，并以返回值来作为线程的执行结果。继承类须覆盖该方法并确保包含或代理任务线程的基本任务。不要直接调用该方法，应使用任务对象的` execute `方法来调度执行。

 在获得执行结果后应使用` SwingWorker `的` get `方法获取` doInBackground `方法的结果。可以在EDT上调用get方法，但该方法将一直处于阻塞状态，直到任务线程完成。最好只有在知道结果时才调用get方法，这样用户便不用等待。为防止阻塞，可以使用` isDone `方法来检验doInBackground是否完成。另外调用方法` get(longtimeout, TimeUnitunit) `将会一直阻塞直到任务线程结束或超时。get获取任务结果的最好地方是在done方法内。
SwingWorker在EDT上激活done方法，因此可以在此方法内安全地和任何GUI组件交互。

**publish和process**
SwingWorker在doInBackground方法结束后才产生最后结果，但任务线程也可以产生和公布中间数据。有时没必要等到线程完成就可以获得中间结果。
中间结果是任务线程在产生最后结果之前就能产生的数据。当任务线程执行时，
**通过在doInBackground内使用publish方法来发布要处理的中间数据它可以发布类型为V的中间结果，通过覆盖process方法来处理中间结果。**
任务对象的**父类会在EDT线程上激活process方法，因此在process方法中程序可以安全的更新UI组件。**

**cancel和isCancelled**
如果想允许程序用户取消任务，实现代码要在SwingWorker子类中周期性地检查取消请求。调用isCancelled方法来检查是否有取消请求。检查的时机主要是：
  * doInBackground方法的
  * process方法中
  * done方法中


