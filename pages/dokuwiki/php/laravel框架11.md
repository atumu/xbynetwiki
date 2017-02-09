title: laravel框架11 

#  Laravel之服务容器和服务提供者 
##  服务容器 
Laravel 服务容器是管理类依赖与运行依赖注入的强力工具。
###  绑定 
` 几乎所有的服务容器绑定都会注册至服务提供者中 `，所以下方所有的例子将示范在该情境中使用容器。
` 不过，如果它们没有依赖任何的接口，那么就没有将类绑定至容器中的必要。并不需要为容器指定如何建构这些对象，因为它会通过 PHP 的反射服务自动解析「实际」的对象。 `
**在服务提供者中，你总是可以通过 $this->app 实例变量访问容器。**我们可以` 使用 bind 方法注册一个绑定 `，传递我们希望注册的类或接口名称，并连同返回该类实例的闭包：
```

$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app['HttpClient']);
});

```
注意，我们获取到的容器本身作为参数传递给解析器。我们可以使用容器来解析我们绑定对象的次要依赖。

**绑定一个单例**
```

$this->app->singleton('FooBar', function ($app) {
    return new FooBar($app['SomethingElse']);
});

```
**绑定实例**
```

$fooBar = new FooBar(new SomethingElse);
$this->app->instance('FooBar', $fooBar);

```

**绑定接口至实现**
```

$this->app->bind('App\Contracts\EventPusher', 'App\Services\RedisEventPusher');

```
这么做会告知容器当有个类需要 EventPusher 的实现时，必须注入 RedisEventPusher。现在我们可以在构造器中对 EventPusher 接口使用类型提示，或任何需要通过服务容器注入依赖的其他位置：
```

use App\Contracts\EventPusher;
public function __construct(EventPusher $pusher)
{
    $this->pusher = $pusher;
}

```

**情境绑定**
**有时候，你可能有两个类使用到相同接口，但你希望每个类都能注入不同实现。**例如，当系统收到新订单时，我们可能想通过 PubNub 来发送事件，而不是 Pusher。Laravel 提供一个简单流畅的接口来定义此行为：
```

$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give('App\Services\PubNubEventPusher');

```
你甚至可以传递一个闭包至 give 方法：
```

$this->app->when('App\Handlers\Commands\CreateOrderHandler')
          ->needs('App\Contracts\EventPusher')
          ->give(function () {
                  // Resolve dependency...
              });

```
**标记(名称)**
有些时候，可能需要解析某个「分类」下的所有绑定。例如，你正在构建一个能接收多个不同 Report 接口实现数组的报表汇整器。注册完 Report 实现后，可以使用 tag 方法为它们赋予一个标签：
```

$this->app->bind('SpeedReport', function () {
    //
});
$this->app->bind('MemoryReport', function () {
    //
});
$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

```
一旦服务被标记之后，你可以通过 tagged 方法将它们全部解析：
```

$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});

```
###  解析 
有几种方式可以从容器中解析一些东西。首先，你可以使用`  make 方法 `，它会接收你希望解析的类或是接口的名称：
```

$fooBar = $this->app->make('FooBar');

```
或者，你可以像数组一样从容器中进行访问，**因为他实现了 PHP 的 ArrayAccess 接口**：
```

$fooBar = $this->app['FooBar'];

```
举个例子，你可以在控制器的构造器中对应用程序定义的保存库进行类型提示。保存库会自动被解析及注入至类中：
```

<?php
namespace App\Http\Controllers;

use Illuminate\Routing\Controller;
use App\Users\Repository as UserRepository;

class UserController extends Controller
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
    public function show($id)
    {
        //
    }
}

```
###  容器事件 
每当服务容器解析一个对象时就会触发事件。你可以使用 ` resolving 方法 `监听这个事件：
```

$this->app->resolving(function ($object, $app) {
    // 当容器解析任何类型的对象时会被调用...
});
$this->app->resolving(FooBar::class, function (FooBar $fooBar, $app) {
    // 当容器解析「FooBar」类型的对象时会被调用...
});

```
如你所见，**被解析的对象会被传递至回调中，让你可以在传递前设置任何额外属性到对象上**。

##  服务提供者 
http://laravel-china.org/docs/5.1/providers
**服务提供者是所有 Laravel 应用程序启动的中心所在。**包括你自己的应用程序，以及所有的 Laravel 核心服务，都是通过服务提供者启动的。
**但我们所说的「启动」指的是什么？一般而言，我们指的是注册事物**，包括**注册服务容器绑定、事件侦听器、中间件，甚至路由**。服务提供者是设置你的应用程序的中心所在。
若你打开 Laravel 的 ` config/app.php ` 文件，你将会看到 ` providers 数组 `。这些都是你的应用程序会加载到的所有服务提供者类。当然，它们之中有很多属于**「延迟」提供者**，意味着除非真正需要它们所提供的服务，否则它们并不会在每一个请求中都被加载。
###  编写服务提供者 
所有的服务提供者都继承了 ` Illuminate\Support\ServiceProvider ` 类。这个抽象类要求你在你的提供者上**定义至少一个方法：register**。
` 在 register 方法中，你应该只将事物绑定至服务容器之中。永远不要试图在 register 方法中注册任何事件侦听器、路由或任何其他功能(这些功能放到boot方法中)。 `
```

Artisan 命令行接口可以很容易地通过 make:provider 命令生成新的提供者：
php artisan make:provider RiakServiceProvider

```
###  注册方法 
如同之前提到的，在 register 方法中，你应该只将事物绑定至服务容器中。永远不要尝试在 register 方法中注册任何事件侦听器、路由或任何其他功能。否则的话，你可能会意外地使用到由尚未加载的服务提供者所提供的服务。
现在，让我们来看看基本的服务提供者：
```

<?php
namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton('Riak\Contracts\Connection', function ($app) {
            return new Connection(config('riak'));
        });
    }
}

```
###  启动方法 
因此，若我们需要在我们的服务提供者中注册一个视图组件则应该在 boot 方法中完成。此方法会在所有其他的服务提供者被注册后才被调用，意味着你能访问已经被框架注册的所有其他服务：
```

<?php
namespace App\Providers;
use Illuminate\Support\ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * 运行注册后的启动服务。
     *
     * @return void
     */
    public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }

    /**
     * 在容器中注册绑定。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}

```
**启动方法依赖注入**
我们可以为我们 boot 方法中的依赖作类型提式。服务容器会自动注入你所需要的任何依赖：
```

use Illuminate\Contracts\Routing\ResponseFactory;
public function boot(ResponseFactory $factory)
{
    $factory->macro('caps', function ($value) {
        //
    });
}

```

###  注册提供者 
**所有的服务提供者都在 config/app.php 配置文件中被注册。**这个文件包含了一个** providers 数组**，你可以在其中列出你所有服务提供者的名称。此数组默认会列出一组 Laravel 的核心服务提供者。这些提供者启动了 Laravel 的核心组件，例如邮件寄送者、队列、缓存及其他等等。
欲注册你的提供者，只需将它加入此数组：
```

'providers' => [
    // 其他的服务提供者

    App\Providers\AppServiceProvider::class,
],

```
###  延迟提供者 
若你的提供者仅在服务容器中注册绑定，你可以选择延缓其注册，直到真正需要其中已注册的绑定。延缓像这样的提供者加载可增进你的应用程序的性能，因为这样就毋须每个请求都从文件系统将其加载。
**要延缓提供者加载，可将 defer 属性设置为 true，并定义一个 provides 方法。**provides 方法会返回提供者所注册的服务容器绑定：
```

<?php
namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * 指定提供者加载是否延缓。
     *
     * @var bool
     */
    protected $defer = true;

    /**
     * 注册服务提供者。
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton('Riak\Contracts\Connection', function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * 获取提供者所提供的服务。
     *
     * @return array
     */
    public function provides()
    {
        return ['Riak\Contracts\Connection'];
    }

}

```
Laravel 编译并保存了一份清单，包括所有由延缓服务提供者所提供的服务，以及其服务提供者类的名称。因此，**只有在当你企图解析其中的服务时，Laravel 才会加载该服务提供者。**