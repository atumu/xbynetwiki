title: phplearn09 

#  PHP学习之日志记录 
##  Monolog：PHP 日志记录工具 
http://hao.jobbole.com/monolog/
http://monolog.ow2.org/
https://github.com/Seldaek/monolog
Monolog 是PHP的一个日志类库。相比于其他的日志类库，它有以下的特点：
  * 功能强大。可以把日志发送到文件、socket、邮箱、数据库和各种web services。
  * 遵循 PSR3 的接口规范。可以很轻易的替换成其他遵循同一规范的日志类库。
  * 良好的扩展性。通过 Handler 、 Formatter 和 Processor 这几个接口，可以对Monolog类库进行各种扩展和自定义。
###  安装 
```

composer require monolog/monolog

```
```

<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
// create a log channel
$log = new Logger('name');
$log->pushHandler(new StreamHandler('path/to/your.log', Logger::WARNING));
 
// add records to the log
$log->addWarning('Foo');
$log->addError('Bar');

```
###  核心概念 
**每一个 Logger 实例都包含一个通道名(channel)和handler的堆栈。**当你添加一条记录时，记录会依次通过handler堆栈的处理。而每个handler也可以决定是否把记录传递到下一个堆栈里的下一个handler。
通过handler，我们可以实现一些复杂的日志操作。例如我们把 StreamHandler 放在堆栈的最下面，那么所有的日志记录最终都会写到硬盘文件里。同时我们把 MailHandler 放在堆栈的最上面，通过设置日志等级把错误日志通过邮件发送出去。**Handler里有个 $bubble 属性，这个属性定义了handler是否拦截记录不让它流到下一个handler。所以如果我们把 MailHandler 的 $bubble 参数设置为 false ，则出现错误日志时，日志会通过 MailHandler 发送出去，而不会经过 StreamHandler 写到硬盘上。**

**Logger 可以创建多个，每个都可以定义自己的频道名和handler堆栈。handler可以在多个 Logger 中共享。频道名会反映在日志里，方便我们查看和过滤日志记录。**
如果没有指定日志格式（Formatter），Handler会使用默认的Formatter。

**日志等级**
  * DEBUG (100): 详细的debug信息。
  * INFO (200): 关键事件。
  * NOTICE (250): 普通但是重要的事件。
  * WARNING (300): 出现非错误的异常。
  * ERROR (400): 运行时错误，但是不需要立刻处理。
  * CRITICA (500): 严重错误。
  * EMERGENCY (600): 系统不可用。
###  自定义格式 
```

// the default date format is "Y-m-d H:i:s"
$dateFormat = "Y n j, g:i a";
// the default output format is "[%datetime%] %channel%.%level_name%: %message% %context% %extra%\n"
$output = "%datetime% > %level_name% > %message% %context% %extra%\n";
// finally, create a formatter
$formatter = new LineFormatter($output, $dateFormat);

// Create a handler
$stream = new StreamHandler(__DIR__.'/my_app.log', Logger::DEBUG);
$stream->setFormatter($formatter);
// bind it to a logger object
$securityLogger = new Logger('security');
$securityLogger->pushHandler($stream);

```
###  多个handler 
```

<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\FirePHPHandler;

// 创建Logger实例
$logger = new Logger('my_logger');
// 添加handler
$logger->pushHandler(new StreamHandler(__DIR__.'/my_app.log', Logger::DEBUG));
$logger->pushHandler(new FilePHPHandler());

// 开始使用
$logger->addInfo('My logger is now ready');

```
###  添加额外的数据 
Monolog有两种方式对日志添加额外的信息。
**第一个方法是使用$context参数，传入一个数组：**
```

<?php
$logger->addInfo('Adding a new user', array('username' => 'Seldaek'));

```

**第二个方法是使用processor。**processor可以是任何可调用的方法，这些方法把日志记录作为参数，然后经过处理修改 extra 部分后返回。
```

<?php

$logger->pushProcessor(function ($record) {
    $record['extra']['dummy'] = 'Hello world!';

    return $record;
});

```
Processor不一定要绑定在Logger实例上，也可以绑定到某个具体的handler上。使用handler实例的 pushProcessor 方法进行绑定。

###  channel的使用 
使用频道名可以对日志进行分类，这在大型的应用上是很有用的。通过频道名，可以很容易的对日志记录进行刷选。
例如我们想在同一个日志文件里记录不同模块的日志，我们可以把相同的handler绑定到不同的Logger实例上，这些实例使用不同的频道名：
```

<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\FirePHPHandler;

// 创建handler
$stream = new StreamHandler(__DIR__.'/my_app.log', Logger::DEBUG);
$firephp = new FirePHPHandler();

// 创建应用的主要logger
$logger = new Logger('my_logger');
$logger->pushHandler($stream);
$logger->pushHandler($firephp);

// 通过不同的频道名创建一个用于安全相关的logger
$securityLogger = new Logger('security');
$securityLogger->pushHandler($stream);
$securityLogger->pushHandler($firephp);

```
###  Handler 
Monolog内置很多很实用的handler，它们几乎囊括了各种的使用场景，这里介绍一些使用的：
  * **StreamHandler ：把记录写进PHP流，主要用于日志文件。**
  * **SyslogHandler ：把记录写进syslog。**
  * ErrorLogHandler ：把记录写进PHP错误日志。
  * **NativeMailerHandler ：使用PHP的 mail() 函数发送日志记录。**
  * SocketHandler ：通过socket写日志。
  * AmqpHandler ：把记录写进兼容 amqp 协议的服务。
  * **BrowserConsoleHandler ：把日志记录写到浏览器的控制台。**由于是使用浏览器的 console 对象，需要看浏览器是否支持。
  * **RedisHandler ：把记录写进Redis。**
  * **MongoDBHandler ：把记录写进Mongo。**
  * ElasticSearchHandler ：把记录写到ElasticSearch服务。
  * BufferHandler ：允许我们把日志记录缓存起来一次性进行处理。
更多的Handler请看 https://github.com/Seldaek/monolog#handlers 。
```

<?php

use Monolog\Logger;
use Monolog\Handler\SocketHandler;

// Create the logger
$logger = new Logger('my_logger');

// Create the handler
$handler = new SocketHandler('unix:///var/log/httpd_app_log.socket');
$handler->setPersistent(true);

// Now add the handler
$logger->pushHandler($handler, Logger::DEBUG);

// You can now use your logger
$logger->addInfo('My logger is now ready');

```
###  Formatter 
同样的，这里介绍几个自带的Formatter：
  * **LineFormatter ：把日志记录格式化成一行字符串。**
  * HtmlFormatter ：把日志记录格式化成HTML表格，主要用于邮件。
  * JsonFormatter ：把日志记录编码成JSON格式。
  * LogstashFormatter ：把日志记录格式化成logstash的事件JSON格式。
  * ElasticaFormatter ：把日志记录格式化成ElasticSearch使用的数据格式。
更多的Formatter请看 https://github.com/Seldaek/monolog#formatters 。
###  Processor 
前面说过，**Processor可以为日志记录添加额外的信息**，Monolog也提供了一些很实用的processor：
  * IntrospectionProcessor ：增加当前脚本的文件名和类名等信息。
  * WebProcessor ：增加当前请求的URI、请求方法和访问IP等信息。
  * MemoryUsageProcessor ：增加当前内存使用情况信息。
  * MemoryPeakUsageProcessor ：增加内存使用高峰时的信息。
更多的Processor请看 https://github.com/Seldaek/monolog#processors 。

###  扩展handler 
Monolog内置了很多handler，但是并不是所有场景都能覆盖到，有时需要自己去定制handler。
写一个handler并不难，**只需要实现 Monolog\Handler\HandlerInterface 这个接口即可。**
下面这个例子实现了把日志记录写到数据库里。我们不需要把接口里的方法全部实现一次，可以直接使用Monolog提供的**抽象类 AbstractProcessingHandler** 进行继承，实现里面的 write 方法即可。
```

<?php
use Monolog\Logger;
use Monolog\Handler\AbstractProcessingHandler;
class PDOHandler extends AbstractProcessingHandler
{
  private $initialized = false;
  private $pdo;
  private $statement;
  public function __construct(PDO $pdo, $level = Logger::DEBUG, $bubble = true)
  {
    $this->pdo = $pdo;
    parent::__construct($level, $bubble);
  }
  protected function write(array $record)
  {
    if (!$this->initialized) {
      $this->initialize();
    }
    $this->statement->execute(array(
      'channel' => $record['channel'],
      'level' => $record['level'],
      'message' => $record['formatted'],
      'time' => $record['datetime']->format('U'),
    ));
  }
  private function initialize()
  {
    $this->pdo->exec(
      'CREATE TABLE IF NOT EXISTS monolog '
      .'(channel VARCHAR(255), level INTEGER, message LONGTEXT, time INTEGER UNSIGNED)'
    );
    $this->statement = $this->pdo->prepare(
      'INSERT INTO monolog (channel, level, message, time) VALUES (:channel, :level, :message, :time)'
    );
  }
}

```
然后我们就可以使用它了：
```

<?php
$logger->pushHandler(new PDOHandler(new PDO('sqlite:logs.sqlite'));
// You can now use your logger
$logger->addInfo('My logger is now ready');

```
                     
##  示例 
```

<?php
// Use Composer autoloader
require 'vendor/autoload.php';

// Import Monolog namespaces
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// Setup Monolog logger
$log = new Logger('my-app-name');
$log->pushHandler(new StreamHandler('logs/development.log', Logger::WARNING));

// Use logger
$log->warning('This is a warning!');


```
```

<?php
// Use Composer autoloader
require 'vendor/autoload.php';

// Import Monolog namespaces
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\SwiftMailerHandler;

date_default_timezone_set('America/New_York');

// Setup Monolog and basic handler
$log = new Logger('my-app-name');
$log->pushHandler(new StreamHandler('logs/production.log', Logger::WARNING));

// Add SwiftMailer handler for critical errors
$transport = \Swift_SmtpTransport::newInstance('smtp.example.com', 587)
             ->setUsername('USERNAME')
             ->setPassword('PASSWORD');
$mailer = \Swift_Mailer::newInstance($transport);
$message = \Swift_Message::newInstance()
           ->setSubject('Website error!')
           ->setFrom(array('daemon@example.com' => 'John Doe'))
           ->setTo(array('admin@example.com'));
$log->pushHandler(new SwiftMailerHandler($mailer, $message, Logger::CRITICAL));

// Use logger
$log->critical('The server is on fire!');


```
参考：
http://www.tuicool.com/articles/eiIbYjJ
https://github.com/Seldaek/monolog/blob/master/doc/01-usage.md