title: phplearn31 

#  PHP学习之多线程 
http://php.net/manual/zh/book.pthreads.php
**PHP 5.3 以上版本，使用pthreads PHP扩展，可以使PHP真正地支持多线程**。多线程在处理重复性的循环任务，能够大大缩短程序执行时间。
不过可以看出，它是参照java的线程模型实现的。
PHP扩展下载：https://github.com/krakjoe/pthreads
PHP手册文档：http://php.net/manual/zh/book.pthreads.php
**pthreads 是一组允许用户在 PHP 中使用多线程技术的面向对象的 API。** 它提供了创建多线程应用所需的全套工具，无论是 Web 应用还是控制台应用。 
通过使用`  Thread， Worker 以及 Threaded ` 对象，PHP 应用可以创建、读取、写入以及执行多线程应用，并可以在多个线程之间进行同步控制。

  * Threaded 对象： Threaded 对象提供支持 pthreads 操作的基本功能，包括同步方法以及其他对程序员很有帮助的接口。
  * Thread 对象： 通过继承 pthreads 中提供的 Thread 对象并实现 run 方法，用户可以创建自己的 Thread 对象。 只要线程上下文中持有某个 Thread 对象的引用，就可以读/写该对象的属性，也可以调用该对象的公有（public）或者受保护（protected）的方法。 当在创建 Thread 对象的进程或线程上下文中调用该对象的 start 方法时，pthreads 扩展会在另外的独立线程中执行该对象的 run 方法。 仅有创建 Thread 对象的线程/进程方可开始（start）或者加入（join）这个 Thread 对象。
  * Worker 对象： Worker 是有状态的线程对象，它在线程开始之后就可用，除非代码中显式地关闭线程，否则该对象在线程对象超出作用范围之后才会失效。 持有 Worker 对象引用的线程上下文可以向 Worker 中入栈其他线程对象，Worker 对象将在独立线程中执行入栈对象的代码。 Woker 对象的 run 方法会在它的栈中入栈对象之前执行，这样就可以进行一些必需的资源初始化工作。

  * Pool 对象： **Pool 对象是 Worker 线程对象池，可以用来在多个 Worker 对象之间分发 Threaded 对象，它可以很好的处理对象应用。** Pool 对象从 1.0.0 版本开始引入，现在已经成为最易用且高效多线程编程方式。(Pool 是标准 PHP 对象，所以**不可以在多个线程上下文中共享同一个 Pool 对象。**)

**线程间同步**： **由 pthreads 扩展创建的所有对象拥有内置的线程间同步机制，和 Java 语言很类似，使用 ::wait 和 :: notify 方法。** 调用某一个对象的 ::wait 方法会导致当前线程上下文进入等待状态，等待另外一个线程上下文调用同一个对象的 ::notify 方法。 为 PHP Threaded 对象提供了强有力的线程间同步控制机制。
**应用中会用在多线程场景中的对象都应该从 Threaded 类继承。**

方法修饰符： Threaded 对象中的受保护方法（protected）是被 pthreads 保护的，也就是说，在同一时间，只有一个线程可以访问该方法。 在执行过程中，私有方法（private）只能被该对象本身调用。

##  安装 
pthreads 扩展由 PECL 主持，使用 » github 管理源代码。 步骤如下：
1、下载:使用标准的 PECL 包安装方式就可以完成安装：» http://pecl.php.net/package/pthreads。
Windows 用户可以从 » [PECL](http://windows.php.net/downloads/pecl/releases/pthreads/) 下载已经构建的二进制发行包。
**注意：新版pthreads2.0重构之后只支持php7了，PHP5要下1.0版**
2、：Extract the zip.Move php_pthreads.dll to the php\ext\ directory.
3、：（Windows 用户需要将 pthreadVC2.dll （包含在 Windows 版二进制发行包中）所在路径加入到 PATH 环境变量中。如Extract the zip ，Move pthreadVC2.dll to the 'C:\windows\system32' directory.）
4、然后启用Open php\php.ini and add
extension=php_pthreads.dll

##  快速入门 
主要类和接口:
  * Threaded 类
  * Thread 类
  * Worker 类
  * Pool 类
  * Mutex 类
  * Cond 类
```

<?php
class My extends Thread{
    function run(){
        for($i=1;$i<10;$i++){
            echo Thread::getCurrentThreadId() .  "\n";
            sleep(2);     // <------
        }
    }
}

for($i=0;$i<2;$i++){
    $pool[] = new My(); 
}

foreach($pool as $worker){
    $worker->start();
}
foreach($pool as $worker){
    $worker->join();
}
?>

```

给出一段PHP多线程、与For循环，抓取百度搜索页面的PHP代码示例：
```

<?php
  class test_thread_run extends Thread 
  {
      public $url;
      public $data;

      public function __construct($url)
      {
          $this->url = $url;
      }

      public function run()
      {
          if(($url = $this->url))
          {
              $this->data = model_http_curl_get($url);
          }
      }
  }

  function model_thread_result_get($urls_array) 
  {
      foreach ($urls_array as $key => $value) 
      {
          $thread_array[$key] = new test_thread_run($value["url"]);
          $thread_array[$key]->start();
      }

      foreach ($thread_array as $thread_array_key => $thread_array_value) 
      {
          while($thread_array[$thread_array_key]->isRunning())
          {
              usleep(10);
          }
          if($thread_array[$thread_array_key]->join())
          {
              $variable_data[$thread_array_key] = $thread_array[$thread_array_key]->data;
          }
      }
      return $variable_data;
  }

  function model_http_curl_get($url,$userAgent="") 
  {
      $userAgent = $userAgent ? $userAgent : 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.2)'; 
      $curl = curl_init();
      curl_setopt($curl, CURLOPT_URL, $url);
      curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
      curl_setopt($curl, CURLOPT_TIMEOUT, 5);
      curl_setopt($curl, CURLOPT_USERAGENT, $userAgent);
      $result = curl_exec($curl);
      curl_close($curl);
      return $result;
  }

  for ($i=0; $i < 100; $i++) 
  { 
      $urls_array[] = array("name" => "baidu", "url" => "http://www.baidu.com/s?wd=".mt_rand(10000,20000));
  }

  $t = microtime(true);
  $result = model_thread_result_get($urls_array);
  $e = microtime(true);
  echo "多线程：".($e-$t)."
";

  $t = microtime(true);
  foreach ($urls_array as $key => $value) 
  {
      $result_new[$key] = model_http_curl_get($value["url"]);
  }
  $e = microtime(true);
  echo "For循环：".($e-$t)."
";
?>

```
参考：http://www.jb51.net/article/49701.htm