title: phplearn30 

#  PHP学习之使用协程实现多任务调度 
**PHP5.5一个比较好的新功能是加入了对迭代生成器和协程的支持.**
这篇文章将尝试通过介绍如何使用协程来实施任务调度, 来解释在PHP中的协程.
##  协程 
http://www.laruence.com/2015/05/28/3038.html
**协程的支持是在迭代生成器的基础上**, 增加了可以回送数据给生成器的功能(调用者发送数据给被调用的生成器函数). **这就把生成器到调用者的单向通信转变为两者之间的双向通信.**
**传递数据的功能是通过迭代器的send()方法实现的.** 下面的logger()协程是这种通信如何运行的例子：
```

<?php
function logger($fileName) {
    $fileHandle = fopen($fileName, 'a');
    while (true) {
        fwrite($fileHandle, yield . "\n");
    }
}
 
$logger = logger(__DIR__ . '/log');
$logger->send('Foo');
$logger->send('Bar')
?>

```
**正如你能看到,这儿yield没有作为一个语句来使用, 而是用作一个表达式, 即它能被演化成一个值. 这个值就是调用者传递给send()方法的值. 在这个例子里, yield表达式将首先被”Foo”替代写入Log, 然后被”Bar”替代写入Log.**
上面的例子里演示了yield作为接受者, **接下来我们看如何同时进行接收和发送的例子：**
` 主要类:Task与Scheduler `
```

<?php
function gen() {
    $ret = (yield 'yield1');//$ret得到的是yield收到send的值。
    var_dump($ret);
    $ret = (yield 'yield2');
    var_dump($ret);
}
$gen = gen();
var_dump($gen->current());    // string(6) "yield1"
var_dump($gen->send('ret1')); // string(4) "ret1"   (the first var_dump in gen)
                              // string(6) "yield2" (the var_dump of the ->send() return value)
var_dump($gen->send('ret2')); // string(4) "ret2"   (again from within gen)
                              // NULL               (the return value of ->send())
?>

```
即使没有调用生成器如$gen->next()方法，send()也会导致指针向下移动。

要很快的理解输出的精确顺序可能稍微有点困难, 但你确定要搞清楚为什按照这种方式输出. 以便后续继续阅读.
另外, 我要特别指出的有两点：
第一点,yield表达式两边的括号在PHP7以前不是可选的, 也就是说在PHP5.5和PHP5.6中圆括号是必须的.
第二点,你可能已经注意到调用current()之前没有调用rewind().这是因为生成迭代对象的时候已经隐含地执行了rewind操作.

让我们看看下面具有两个简单（没有什么意义）任务的调度器：
```

<?php
function task1() {
    for ($i = 1; $i <= 10; ++$i) {
        echo "This is task 1 iteration $i.\n";
        yield;
    }
}
 
function task2() {
    for ($i = 1; $i <= 5; ++$i) {
        echo "This is task 2 iteration $i.\n";
        yield;
    }
}
 
$scheduler = new Scheduler();
 
$scheduler->new Task(task1());
$scheduler->new Task(task2());
 
$scheduler->run();

```
两个任务都仅仅回显一条信息,然后使用yield把控制回传给调度器.输出结果如下：

This is task 1 iteration 1.
This is task 2 iteration 1.
This is task 1 iteration 2.
This is task 2 iteration 2.
This is task 1 iteration 3.
This is task 2 iteration 3.
This is task 1 iteration 4.
This is task 2 iteration 4.
This is task 1 iteration 5.
This is task 2 iteration 5.
This is task 1 iteration 6.
This is task 1 iteration 7.
This is task 1 iteration 8.
This is task 1 iteration 9.
This is task 1 iteration 10.
输出确实如我们所期望的：对前五个迭代来说,两个任务是交替运行的, 而在第二个任务结束后, 只有第一个任务继续运行.

更多请参考：http://www.laruence.com/2015/05/28/3038.html