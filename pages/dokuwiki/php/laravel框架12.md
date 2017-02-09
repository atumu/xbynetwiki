title: laravel框架12 

#  Laravel之核心接口(Contracts)和静态代理(Facades) 
##  Contracts 
Laravel 的 Contracts 是一组定义了**框架核心服务的接口**（ php class interfaces ）。例如，Illuminate\Contracts\Queue\Queue contract 定义了队列任务所需要的方法，而 Illuminate\Contracts\Mail\Mailer contract 定义了寄送 e-mail 需要的方法。
框架对于每个 contract 都有提供对应的实现。
Laravel 所有的 contracts 都放在 各自的[GitHub 保存库](https://github.com/illuminate/contracts)。除了提供给所有可用的 contracts 一个快速的参考，也可以单独作为一个低耦合的扩展包来让其他扩展包开发者使用。

` Contracts其实就是倡导面向接口编程，来达到解耦的目的。而这些通用的接口已经由Laravel为你设计好了。就是这些Contracts. `

```

<?php

namespace App\Orders;

use Illuminate\Contracts\Cache\Repository as Cache;

class Repository
{
    /**
     * 缓存实例。
     */
    protected $cache;

    /**
     * 创建一个新的仓库实例。
     *
     * @param  Cache  $cache
     * @return void
     */
    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }
}

```

Contracts其实就是倡导面向接口编程，来达到解耦的目的。而这些通用的接口已经由Laravel为你设计好了。就是这些Contracts.
**那么Laravel如何知道我们需要使用哪个实现呢？**
在Laravel默认的Contracts绑定中，在` 'Illuminate/Foundation/Application.php `'有这样的定义:
```

   /**
     * Register the core class aliases in the container.
     *
     * @return void
     */
    public function registerCoreContainerAliases()
    {
        $aliases = [
            'app'                  => ['Illuminate\Foundation\Application', 'Illuminate\Contracts\Container\Container', 'Illuminate\Contracts\Foundation\Application'],
            'auth'                 => 'Illuminate\Auth\AuthManager',
            'auth.driver'          => ['Illuminate\Auth\Guard', 'Illuminate\Contracts\Auth\Guard'],
            'auth.password.tokens' => 'Illuminate\Auth\Passwords\TokenRepositoryInterface',
            'blade.compiler'       => 'Illuminate\View\Compilers\BladeCompiler',
            'cache'                => ['Illuminate\Cache\CacheManager', 'Illuminate\Contracts\Cache\Factory'],
            'cache.store'          => ['Illuminate\Cache\Repository', 'Illuminate\Contracts\Cache\Repository'],
            'config'               => ['Illuminate\Config\Repository', 'Illuminate\Contracts\Config\Repository'],
            'cookie'               => ['Illuminate\Cookie\CookieJar', 'Illuminate\Contracts\Cookie\Factory', 'Illuminate\Contracts\Cookie\QueueingFactory'],
            'encrypter'            => ['Illuminate\Encryption\Encrypter', 'Illuminate\Contracts\Encryption\Encrypter'],
            'db'                   => 'Illuminate\Database\DatabaseManager',
            'db.connection'        => ['Illuminate\Database\Connection', 'Illuminate\Database\ConnectionInterface'],
            'events'               => ['Illuminate\Events\Dispatcher', 'Illuminate\Contracts\Events\Dispatcher'],
            'files'                => 'Illuminate\Filesystem\Filesystem',
            'filesystem'           => ['Illuminate\Filesystem\FilesystemManager', 'Illuminate\Contracts\Filesystem\Factory'],
            'filesystem.disk'      => 'Illuminate\Contracts\Filesystem\Filesystem',
            'filesystem.cloud'     => 'Illuminate\Contracts\Filesystem\Cloud',
            'hash'                 => 'Illuminate\Contracts\Hashing\Hasher',
            'translator'           => ['Illuminate\Translation\Translator', 'Symfony\Component\Translation\TranslatorInterface'],
            'log'                  => ['Illuminate\Log\Writer', 'Illuminate\Contracts\Logging\Log', 'Psr\Log\LoggerInterface'],
            'mailer'               => ['Illuminate\Mail\Mailer', 'Illuminate\Contracts\Mail\Mailer', 'Illuminate\Contracts\Mail\MailQueue'],
            'auth.password'        => ['Illuminate\Auth\Passwords\PasswordBroker', 'Illuminate\Contracts\Auth\PasswordBroker'],
            'queue'                => ['Illuminate\Queue\QueueManager', 'Illuminate\Contracts\Queue\Factory', 'Illuminate\Contracts\Queue\Monitor'],
            'queue.connection'     => 'Illuminate\Contracts\Queue\Queue',
            'redirect'             => 'Illuminate\Routing\Redirector',
            'redis'                => ['Illuminate\Redis\Database', 'Illuminate\Contracts\Redis\Database'],
            'request'              => 'Illuminate\Http\Request',
            'router'               => ['Illuminate\Routing\Router', 'Illuminate\Contracts\Routing\Registrar'],
            'session'              => 'Illuminate\Session\SessionManager',
            'session.store'        => ['Illuminate\Session\Store', 'Symfony\Component\HttpFoundation\Session\SessionInterface'],
            'url'                  => ['Illuminate\Routing\UrlGenerator', 'Illuminate\Contracts\Routing\UrlGenerator'],
            'validator'            => ['Illuminate\Validation\Factory', 'Illuminate\Contracts\Validation\Factory'],
            'view'                 => ['Illuminate\View\Factory', 'Illuminate\Contracts\View\Factory'],
        ];

```,这就是绑定了默认的接口实现.
**在我们自定义的接口实现时，我们可以在ServiceProvider中使用进行绑定:**
```

$this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

```

###  Contract 参考: 
http://laravel-china.org/docs/5.1/contracts
![](/data/dokuwiki/php/pasted/20160415-142054.png)![](/data/dokuwiki/php/pasted/20160415-142115.png)![](/data/dokuwiki/php/pasted/20160415-142128.png)
##  Facades 
**Facades 为应用程序的服务容器中可用的类提供了一个「静态」接口**。Laravel 本身附带许多的 facades，甚至你可能在不知情的状况下已经在使用他们！**Laravel 「facades」作为在服务容器内基类的「静态代理」**，提供了一个简洁、易表达的语法优点，同时维持着比传统静态方法更高的可测试性和弹性。

**在 Laravel 应用程序环境（Context）中，facade 是个提供从容器访问对象的类。**
Laravel 的 facades，以及任何你创建的自定义 facades，会继承基底 ` Illuminate\Support\Facades\Facade ` 类。
facade 类只需要去实现一个方法：` getFacadeAccessor `。getFacadeAccessor 方法的工作定义是从容器中解析出什么。
Facade 基类利用 ` _ _callStatic() 魔术方法 `从你的 facade 延迟调用来解析对象。
在下面的例子，调用了 Laravel 的缓存系统。看了一下这个代码，或许有人认为静态方法 get 是被 Cache 类调用的(其实是通过_ _callStatic魔术方法实现的)：
```

<?php
namespace App\Http\Controllers;

use Cache;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示指定用户的个人数据。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}

```
**注意在文件的上方，我们「导入」Cache facade。这个 facade 做为访问底层实现 ` Illuminate\Contracts\Cache\Factory ` 接口的代理。**
我们使用 facade 的任何调用将会发送给 Laravel 缓存服务的底层实例。
**如果我们查看 Illuminate\Support\Facades\Cache 类，你会发现没有静态方法 get：**
```

class Cache extends Facade
{
    /**
     * 获取组件的注册名称。
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'cache'; }
}

```
相反的，Cache facade 继承了基底 Facade 类以及定义了 getFacadeAccessor() 方法。记住，这个方法的工作是返回服务容器绑定的名称。
**当用户在 Cache facade 上参考任何的静态方法，Laravel 会从服务容器解析被绑定的 cache 以及针对对象运行请求的方法（在这个例子中是 get）。**
以上摘自中文网。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。

貌似说了半天还是不知到为何物？
我们打开项目目录下的config/app.php，然后找到
```

/*
    |--------------------------------------------------------------------------
    | Class Aliases
    |--------------------------------------------------------------------------
    |
    | This array of class aliases will be registered when this application
    | is started. However, feel free to register as many as you wish as
    | the aliases are "lazy" loaded so they don't hinder performance.
    |
    */

    'aliases' => [

        'App'       => Illuminate\Support\Facades\App::class,
        'Artisan'   => Illuminate\Support\Facades\Artisan::class,
        'Auth'      => Illuminate\Support\Facades\Auth::class,
        'Blade'     => Illuminate\Support\Facades\Blade::class,
        'Bus'       => Illuminate\Support\Facades\Bus::class,
        'Cache'     => Illuminate\Support\Facades\Cache::class,
        'Config'    => Illuminate\Support\Facades\Config::class,
        'Cookie'    => Illuminate\Support\Facades\Cookie::class,
        'Crypt'     => Illuminate\Support\Facades\Crypt::class,
        'DB'        => Illuminate\Support\Facades\DB::class,
        'Eloquent'  => Illuminate\Database\Eloquent\Model::class,
        'Event'     => Illuminate\Support\Facades\Event::class,
        'File'      => Illuminate\Support\Facades\File::class,
        'Gate'      => Illuminate\Support\Facades\Gate::class,
        'Hash'      => Illuminate\Support\Facades\Hash::class,
        'Input'     => Illuminate\Support\Facades\Input::class,
        'Lang'      => Illuminate\Support\Facades\Lang::class,
        'Log'       => Illuminate\Support\Facades\Log::class,
        'Mail'      => Illuminate\Support\Facades\Mail::class,
        'Password'  => Illuminate\Support\Facades\Password::class,
        'Queue'     => Illuminate\Support\Facades\Queue::class,
        'Redirect'  => Illuminate\Support\Facades\Redirect::class,
        'Redis'     => Illuminate\Support\Facades\Redis::class,
        'Request'   => Illuminate\Support\Facades\Request::class,
        'Response'  => Illuminate\Support\Facades\Response::class,
        'Route'     => Illuminate\Support\Facades\Route::class,
        'Schema'    => Illuminate\Support\Facades\Schema::class,
        'Session'   => Illuminate\Support\Facades\Session::class,
        'Storage'   => Illuminate\Support\Facades\Storage::class,
        'URL'       => Illuminate\Support\Facades\URL::class,
        'Validator' => Illuminate\Support\Facades\Validator::class,
        'View'      => Illuminate\Support\Facades\View::class,

    ],

```
你是不是发现了什么？对，Facades其实就是在config/app.php中定义的一系列类的别名。只不过这些类都具有一个共同的特点，那就是继承基底 ` Illuminate\Support\Facades\Facade ` 类并实现一个方法：` getFacadeAccessor `返回名称。

###  Facade 类参考 
http://laravel-china.org/docs/5.1/facades
以下表格三列分别表示：config/app.php中定义的别名，别名对应的类，getFacadeAccessor()方法返回的容器绑定注册名称。
![](/data/dokuwiki/php/pasted/20160415-141843.png)![](/data/dokuwiki/php/pasted/20160415-141901.png)![](/data/dokuwiki/php/pasted/20160415-141927.png)

###  自定义Facade 
http://www.tutorialspoint.com/laravel/laravel_facades.htm
http://www.cnblogs.com/hanyouchun/p/5341264.html
Step 1 −创建一个名为 TestFacadesServiceProvider的ServiceProvider ,使用如下命令即可:
```

php artisan make:provider TestFacadesServiceProvider

```
Step 2 − 创建一个底层代理类，命名为“TestFacades.php” at “App/Test”.
App/Test/TestFacades.php
```

<?php
namespace App\Test;

class TestFacades{
   public function testingFacades(){
      echo "Testing the Facades in Laravel.";
   }
}
?>

```
Step 3 − 创建一个 Facade 类 called “TestFacades.php” at “App/Test/Facades”.
App/Test/Facades/TestFacades.php
```

<?php
namespace app\Test\Facades;
use Illuminate\Support\Facades\Facade;

class TestFacades extends Facade{
   protected static function getFacadeAccessor() { return 'test'; }
}

```
Step 4 −创建一个ServiceProviders类，名为“TestFacadesServiceProviders.php” at “App/Test/Facades”.
App/Providers/TestFacadesServiceProviders.php
```

<?php
namespace App\Providers;
use App;
use Illuminate\Support\ServiceProvider;

class TestFacadesServiceProvider extends ServiceProvider {
   public function boot() {
      //
   }
   public function register() {
     //可以这么绑定,这需要use App;
    //  App::bind('test',function() {
    //     return new \App\Test\TestFacades;
    //  });
     
     //也可以这么绑定，推荐。这个test对应于Facade的getFacadeAccessor返回值
        $this->app->bind("test", function(){
            return new MyFoo(); //给这个Facade返回一个代理实例。所有对Facade的调用都会被转发到该类对象下。
        });
   }
}

```
Step 5 − 在config/app.php注册ServiceProvider类
Step 6 − 在config/app.php注册自定义Facade的别名

使用测试：
Add the following lines in app/Http/routes.php.
app/Http/routes.php
```

Route::get('/facadeex', function(){
   return TestFacades::testingFacades();
});

```
Step 9 − Visit the following URL to test the Facade.
http://localhost:8000/facadeex去查看输出