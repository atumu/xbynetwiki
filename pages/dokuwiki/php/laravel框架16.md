title: laravel框架16 

#  Laravel之缓存与memcached 
Laravel 给各种不同的缓存系统提供了统一的 API，缓存的配置文件都放在`  config/cache.php ` 中，在这个文件中，你可以指定默认想用哪个缓存驱动，Laravel 支持当前流行的缓存后端，**如 Memcached 和 Redis。**
Laravel 默认采用的缓存驱动是 file，这个驱动在文件系统中保存了序列化的缓存对象，对于大型应用程序而言，Laravel 比较建议你使用内存缓存，例如 Memcached 或 APC。
config/cache.php文件内容如下:
```

<?php
return [
    'default' => env('CACHE_DRIVER', 'file'),

    'stores' => [

        'apc' => [
            'driver' => 'apc',
        ],

        'array' => [
            'driver' => 'array',
        ],

        'database' => [
            'driver' => 'database',
            'table'  => 'cache',
            'connection' => null,
        ],

        'file' => [
            'driver' => 'file',
            'path'   => storage_path('framework/cache'),
        ],

        'memcached' => [
            'driver'  => 'memcached',
            'servers' => [
                [
                    'host' => '127.0.0.1', 'port' => 11211, 'weight' => 100,
                ],
            ],
        ],

        'redis' => [
            'driver' => 'redis',
            'connection' => 'default',
        ],

    ],

    'prefix' => 'laravel',

];


```
数据库
当使用`  database ` 这个缓存驱动时，你需要配置一个数据库表来放置缓存项目，下面是表结构：
```

Schema::create('cache', function($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});

```
**Memcached#**
使用 Memcached 做缓存需要先安装 Memcached PECL 扩展包。` sudo apt-get install php5-memcached `
默认的配置文件采用以 Memcached::addServer 为基础的 TCP/IP：
```

'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],

```
**Redis#**
在你选择使用 Redis 作为 Laravel 的缓存之前，你需要通过 Composer 预先安装 ` predis/predis ` 扩展包 (~1.0)。

##  缓存的使用 
**获取一个缓存的实例**
` Illuminate\Contracts\Cache\Factory ` 和 ` Illuminate\Contracts\Cache\Repository ` contracts 提供了访问 Laravel 缓存服务的机制， 而 Factory contract 则为你的应用程序提供了访问所有缓存驱动的机制，Repository contract 是典型的缓存驱动实现，它会依照你的缓存配置文件的变化而变化。
你也需要使用`  Cache facade `，我们将会在此文档中介绍，Cache facade 提供了方便又简洁的方法访问缓存实例。
例如，我们试着在一个控制器中引用 Cache facade：
```

<?php
namespace App\Http\Controllers;

use Cache;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
     * 显示应用程序中所有用户列表。
     *
     * @return Response
     */
    public function index()
    {
        $value = Cache::get('key');

        //
    }
}

```
**访问多个缓存仓库**
使用 Cache facade 可通过`  store 方法 `来访问缓存仓库，**传入 store 方法的键对应一个缓存配置文件中的 stores 配置值**：
```

$value = Cache::store('file')->get('foo');
Cache::store('redis')->put('bar', 'baz', 10);

```
**从缓存中获取项目#**
```

$value = Cache::get('key');
$value = Cache::get('key', 'default');
$value = Cache::get('key', function() {
    return DB::table(...)->get();
});

```
**确认项目是否存在#**
```

if (Cache::has('key')) {
    //
}

```
**递增与递减值#**
```

Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);

```
**取出或更新#**
有时候，你可能会想从缓存中取出一个项目，但也想在当取出的项目不存在时存入一个默认值，例如，你可能会想从缓存中取出所有用户，当找不到用户时，从数据库中将这些用户取出并放入缓存中，你可以使用 Cache::remember 方法达到目的：
```

$value = Cache::remember('users', $minutes, function() {
    return DB::table('users')->get();
});

```
` 如果那个项目不存在缓存中，则返回给 remember 方法的闭包将会被运行，而且闭包的运行结果将会被存放在缓存中。 `
你也可以结合 remember 和 forever 这两个方法来 ”永久“ 存储缓存：
```

$value = Cache::rememberForever('users', function() {
    return DB::table('users')->get();
});

```
**取出与删除#**
如果你需要从缓存中**取出一个项目并删除它**，你可能会使用 pull 方法，与 get 相似，如果对象不存在缓存中，pull 方法将会返回 null：
```

$value = Cache::pull('key');

```
**存放项目到缓存中#**
```

Cache::put('key', 'value', $minutes);

```
如果要指定一个缓存项目过期的分钟数，你也可能需要**传递一个 PHP 的 DateTime 实例**来表示该缓存项目过期的时间点：(注意这里使用了Carbon组件)
```

$expiresAt = Carbon::now()->addMinutes(10);
Cache::put('key', 'value', $expiresAt);

```
**add 方法只会把暂时不存在缓存中的项目放入缓存**，如果成功存放，会返回 true，否则返回 false：
```

Cache::add('key', 'value', $minutes);

```
forever 方法可以用来存放永久的项目到缓存中，这些值必须被手动的删除，这可以通过 forget 方法实现：
```

Cache::forever('key', 'value');

```
**删除缓存中的项目#**
```

Cache::forget('key');

```
也使用 flush 方法清空所有缓存：
```

Cache::flush();

```
清空缓存 并不会 遵从缓存的前缀，并会将缓存中所有的项目删除。在清除与其它应用程序共用的缓存时应谨慎考虑这一点。

##  加入自定义的缓存驱动
我们可以在 Cache facade 中使用 ` extend 方法 `自定义缓存驱动来扩充 Laravel 缓存，它被用来绑定一个自定义驱动的解析器到管理者上，通常这可以通过服务容器来完成。
例如，要注册一个名为「mongo」的缓存驱动：
```

<?php
namespace App\Providers;

use Cache;
use App\Extensions\MongoStore;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Cache::extend('mongo', function($app) {
            return Cache::repository(new MongoStore);
        });
    }
    public function register()
    {
        //
    }
}

```
第一个传给 extend 方法的参数是**驱动的名称，这个名称要与你在 config/cache.php 配置文件中，driver 选项指定的名称相同**，第二个参数是一个应返回一个 ` Illuminate\Cache\Repository ` 实例的闭包。（请别忘了在 config/app.php 中的服务提供者数组注册这个提供者）。
为了创建我们的自定义缓存驱动，首先需要实现 ` Illuminate\Contracts\Cache\Store ` contract。因此我们的 MongoDB 缓存实现大概会长这样子：
```

<?php
namespace App\Extensions;

class MongoStore implements \Illuminate\Contracts\Cache\Store
{
    public function get($key) {}
    public function put($key, $value, $minutes) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}

```
##  缓存标签 
**注意： 缓存标签并不支持使用 file 或 dababase 的缓存驱动**。此外，当在缓存使用多个标签并「永久」写入时，类似 memcached 的驱动性能会是最佳的，且会自动清除旧的纪录。
**写入被标记的缓存项目**
缓存标签允许你在缓存中标记关联的项目，并清空所有已分配指定标签的缓存值。你可以通过传递一组标签名称的有序数组，以访问被标记的缓存。举例来说，让我们访问一个被标记的缓存并 put 值给它：
```

Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

```
当然，你不必限制于 put 方法。你可以在使用标签时使用任何缓存保存系统的方法。

**获取被标记的缓存项目#**
若要获取一个被标记的缓存项目，只要传递一样的标签串行表至 tags 方法：
```

$john = Cache::tags(['people', 'artists'])->get('John');
$anne = Cache::tags(['people', 'authors'])->get('Anne');

```
你可以清空已分配的单个标签或是一组标签列表中的所有项目。例如，下方的语法会把被标记 people、authors，或两者的缓存都给移除。所以，Anne 与 John 都从缓存中被移除：
```

Cache::tags(['people', 'authors'])->flush();

```
相反的，下方的语法只会删除被标示为 authors 的缓存，所以 Anne 会被移除，但 John 不会。
```

Cache::tags('authors')->flush();

```

##  缓存事件# 
你可以监听到缓存做每一次操作的触发事件。一般来说，你必须将事件侦听器放置在`  EventServiceProvider ` 的 boot 方法中：
```

/**
 * 为你的应用程序注册任何其它事件。
 *
 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
 * @return void
 */
public function boot(DispatcherContract $events)
{
    parent::boot($events);

    $events->listen('cache.hit', function ($key, $value) {
        //
    });

    $events->listen('cache.missed', function ($key) {
        //
    });

    $events->listen('cache.write', function ($key, $value, $minutes) {
        //
    });

    $events->listen('cache.delete', function ($key) {
        //
    });
}

```