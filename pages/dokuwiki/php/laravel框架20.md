title: laravel框架20 

#  Laravel之事件 
Laravel 事件提供了简单的侦听器实现，允许你订阅和监听事件，事件类通常被保存在`  app/Events ` 目录下，而它们的侦听器被保存在 ` app/Listeners ` 目录下。
##  注册事件或侦听器# 
你可以在 ` EventServiceProvider ` 注册所有的事件侦听器，` listen 属性 `是一个数组，包含所有事件（键）以及事件对应的侦听器（值），你也可以根据需求增加事件到这个数组，例如：让我们增加 PodcastWasPurchased 事件：
```

/**
 * 应用程序的事件侦听器映射。
 *
 * @var array
 */
protected $listen = [
    'App\Events\PodcastWasPurchased' => [
        'App\Listeners\EmailPurchaseConfirmation',
    ],
];

```
**生成事件或侦听器类#**
```

php artisan event:generate

```
**手动注册事件#**
一般来说，事件必须通过 EventServiceProvider 的 $listen 数组进行注册；不过，你也可以通过 Event facade 或是 Illuminate\Contracts\Events\Dispatcher contract 实现的事件发送器来手动注册事件：
```

/**
 * 注册你应用程序中的任何其它事件。
 *
 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
 * @return void
 */
public function boot(DispatcherContract $events)
{
    parent::boot($events);

    $events->listen('event.name', function ($foo, $bar) {
        //
    });
}

```
**事件侦听器的通配符#**
你也可以使用 * 通配符注册侦听器，让你可以在同个侦听器拦截多个事件。通配符侦听器会接收完整事件数据的数组作为唯一的参数：
```

$events->listen('event.*', function (array $data) {
    //
});

```

##  定义事件 
一个事件类只是一个包含了相关事件信息的数据容器。继承App\Events\Event
```

<?php

namespace App\Events;

use App\Podcast;
use App\Events\Event;
use Illuminate\Queue\SerializesModels;

class PodcastWasPurchased extends Event
{
	public $attr;
  	....
}

```
##  定义侦听器# 
接下来，让我们看一下例子事件的侦听器。事件侦听器的 ` handle 方法 `接收了事件实例。event:generate 命令将会在事件的 handle 方法自动加载正确的事件类和类型提示。在 handle 方法内，你可以运行任何必要响应该事件的逻辑。
```

<?php

namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation
{
    public function __construct()
    {
        //
    }

    /**
     * 处理事件。
     *
     * @param  PodcastWasPurchased  $event
     * @return void
     */
    public function handle(PodcastWasPurchased $event)
    {
        // 使用 $event->podcast 访问播客（podcast）...
    }
}

```
你的事件侦听器也可以在构造器内对任何依赖使用类型提示。所有事件侦听器经由 Laravel 服务容器做解析，所以依赖将会自动的被注入。

**停止一个事件的传播#**
有时候，你可能希望停止一个事件传播到其它的侦听器。你可以通过**在侦听器的 handle 方法中返回 false 来实现。**

###  可队列的事件侦听器# 
需要一个可 队列 的事件侦听器吗？那是再容易不过了。` 只要增加 ShouldQueue 接口到你的侦听器类 `。由 event:generate Artisan 命令生成的侦听器已经将此接口导入到命名空间了，因此可以像这样来立即使用它：
```

<?php
namespace App\Listeners;

use App\Events\PodcastWasPurchased;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class EmailPurchaseConfirmation implements ShouldQueue
{
    //
}

```
##  触发事件# 
如果要触发一个事件，你可以使用`  Event facade ` 来发送一个事件的实例到`  fire 方法 `。fire 方法将会发送事件到所有已经注册的侦听器上：
```

<?php
namespace App\Http\Controllers;

use Event;
use App\Podcast;
use App\Events\PodcastWasPurchased;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function purchasePodcast($userId, $podcastId)
    {
        $podcast = Podcast::findOrFail($podcastId);
        // 逻辑...
        Event::fire(new PodcastWasPurchased($podcast));
    }
}

```
另外，你也可以使用` 全局 event 辅助函数 `来触发事件：
```

event(new PodcastWasPurchased($podcast));

```
##  事件订阅器# 
事件订阅器是一个让你可以订阅多个事件的类，允许你在单个类内定义多个事件的操作。订阅器应该定义一个可以发送一个事件发送器实例的`  subscribe 方法 `，：
```

<?php
namespace App\Listeners;

class UserEventListener
{
    /**
     * 处理用户登录事件。
     */
    public function onUserLogin($event) {}

    /**
     * 处理用户注销事件。
     */
    public function onUserLogout($event) {}

    /**
     * 注册侦听器的订阅者。
     *
     * @param  Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'App\Events\UserLoggedIn',
            'App\Listeners\UserEventListener@onUserLogin'
        );

        $events->listen(
            'App\Events\UserLoggedOut',
            'App\Listeners\UserEventListener@onUserLogout'
        );
    }

}

```
**注册事件订阅器#**
一旦订阅器被定义，它就可以被注册到事件发送器中。你可以在`  EventServiceProvider 中使用 $subscribe 属性 `注册订阅器。例如，让我们增加 UserEventListener。
```

<?php
namespace App\Providers;

use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * 事件侦听器映射到应用程序。
     *
     * @var array
     */
    protected $listen = [
        //
    ];

    /**
     * 订阅者类进行注册。
     *
     * @var array
     */
    protected $subscribe = [
        'App\Listeners\UserEventListener',
    ];
}

```
##  框架事件# 
Laravel 为框架运行的行为提供了许多「核心」事件。你可以像订阅自己自定义事件一样的方式来订阅它们：
![](/data/dokuwiki/php/pasted/20160419-103812.png)![](/data/dokuwiki/php/pasted/20160419-103853.png)

##  广播事件 
在构建实时响应的 Web App 时，经常会使用到 Web Sockets。当在服务器上更新一些数据时，Web Socket 连接通常会发送一个消息来通知客户端处理。
为了协助你创建这些类型的应用程序，Laravel 让你可以简单的通过 Web Socket 连接来「广播」你的事件。广播 Laravel 事件让你能够在服务器端代码和客户端 JavaScript 框架间共享相同的事件名称。
具体请参考http://laravel-china.org/docs/5.1/events