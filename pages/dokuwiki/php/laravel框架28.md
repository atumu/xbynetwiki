title: laravel框架28 

#  Laravel之Session 
由于 HTTP 协议是无状态的，所以 session 提供了一种保存用户数据的方法。Laravel 附带支持了多种 session 后端驱动，并通过统一的 API 进行使用。也内置支持像是 ` Memcached、Redis ` 和数据库这样的后端驱动。
Session 的配置文件在 ` config/session.php `。请务必看一下此配置文件中可用的设置选项及注释。**Laravel 默认使用 file 的 session 驱动**，它在大多数应用中可以良好运作。在上线的应用程序中，你可能会考虑使用更快的 memcached 或 redis 等驱动。
Session driver 定义数据将由什么样的方式进行存储。Laravel 附带了几个不错且可立即使用的驱动：
  * file - 将 sessions 保存在 storage/framework/sessions 中。
  * cookie - 将 sessions 安全的保存在加密后的 cookies 中。
  * database - 将 sessions 保存在应用程序使用的数据库中。
  * memcached / redis - 将 sessions 保存在其中一个快速且基于缓存的存储系统中。
  * array - 将 sessions 保存在简单的 PHP 数组中，并只存在于本次请求。

**数据库#**
当使用 database session 驱动时，你必需先构建保存 session 项目的数据表。以下例子使用 Schema 语法建表：
```

Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->text('payload');
    $table->integer('last_activity');
});

```
你也可使用 session:table Artisan 命令生成迁移文件！
```

php artisan session:table
composer dump-autoload
php artisan migrate

```

**Redis#**
在 Laravel 使用 Redis Session 之前，你需要先通过 Composer 安装 predis/predis(~1.0) 扩展包。

**其它的 Session 使用注意事项#**
Laravel 框架在内部使用了 flash 作为 session 的键，所以应该避免 session 使用此名称。
如果你的 session 数据需要加密，可将配置文件中的 encrypt 选项设为 true。

##  访问 Session# 
我们可在控制器方法内通过对 HTTP 请求使用类型提示访问 session 实例。请记住，控制器方法的依赖是通过 Laravel 的服务容器注入的：
```

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示指定用户的个人文件。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function showProfile(Request $request, $id)
    {
        $value = $request->session()->get('key');

        //
    }
}

```
你可以在 get 方法中的第二个参数内设置一个默认值，当指定的键名不存在时，将会返回设置的默认值。如果你传入一个 闭包 作为 get 方法的默认值，该 闭包 将被运行并返回它的结果：
```

$value = $request->session()->get('key', 'default');
$value = $request->session()->get('key', function() {
    return 'default';
});

```
如果你想从 session 中获取所有数据，则可以使用 all 方法：
```

$data = $request->session()->all();

```
你也可使用全局的 session PHP 函数来获取 session 中保存的数据：
```

Route::get('home', function () {
    // 获取 session 中的一条数据...
    $value = session('key');

    // 写入一条数据至 session 中...
    session(['key' => 'value']);
});

```
**判断项目在 Session 中是否存在#**
has 方法被用于检查项目是否存在于 session 内。如果存在将会返回 true：
```

if ($request->session()->has('users')) {
    //
}

```
**保存数据到 Session 中#**
put 方法(相当于add)能将一个**新的数据**加入现有的 session 内。
```

$request->session()->put('key', 'value');

```
**保存数据进 Session 数组值中#**
push 方法(相当于set)可以将一个**新的值**加入至一个 session 数组内：
```

$request->session()->push('user.teams', 'developers');

```
**从 Session 中取出并删除数据#**
```

$value = $request->session()->pull('key', 'default');

```
**从 Session 中移除数据#**
```

$request->session()->forget('key');
$request->session()->flush();

```
**重新生成 Session ID#**
```

$request->session()->regenerate();

```

##  闪存数据# 
有时候**你想存入一条缓存的数据，让它只在下一次的请求内有效**，则可以使用`  flash 方法 `。使用这个方法保存 session，只能将数据保留到下个 HTTP 请求，然后就会被自动删除。闪存数据在短期的状态消息中很有用：
```

$request->session()->flash('status', 'Task was successful!');

```
**如果需要保留闪存数据给更多请求，可以使用 reflash 方法**，这将会将所有的闪存数据保留给额外的请求。如果想保留特定的闪存数据，则可以使用 keep 方法：
```

$request->session()->reflash();
$request->session()->keep(['username', 'email']);

```

##  加入自定义的 Session 驱动# 
若要加入额外驱动至 Laravel 的 session 中，则可以使用`  Session facade 的 extend ` 方法。在服务提供者 的 boot 方法内调用 extend 方法：
```

<?php
namespace App\Providers;

use Session;
use App\Extensions\MongoSessionStore;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Session::extend('mongo', function($app) {
            // 返回 SessionHandlerInterface 的实现...
            return new MongoSessionStore;
        });
    }
    public function register()
    {
        //
    }
}

```
请注意，` 你自定义的 session 驱动必须实现 SessionHandlerInterface 接口 `。这个接口包含了一些需要实现的方法。一个基本的 MongoDB 实现应该看起来像这样：
```

<?php
namespace App\Extensions;

class MongoHandler implements SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}

```
让我们来快速了解每个方法的作用：
  * open 方法通常用在基于文件的 session 存储系统中。像 Larvel 就附带了一个 file 的驱动，所以你不用把任何东西放到这个方法内。你可以把这方法看做是空的也没关系。
  * close 方法跟 open 方法很相似，通常也被忽略了。对大多数的驱动而言，此方法并不是需要的。
  * read 方法必须根据给予的 $sessionId 返回关联的 session 数据的字符串版本。这在驱动中并不需要做任何的编码跟序列化动作，在 Laravel 内已经会自动运行。
  * write 方法必须在大部分存储系统内写入 $data 字符串时关联至 $sessionId，如 MongoDB、Dynamo 等等。
  * destroy 方法能删除与 $sessionId 相关联的数据。
  * gc 方法能删除 $lifetime 之前的所有数据，$lifetime 是一个 UNIX 的时间戳。但在一些如 Memcached 和 Redis 这样的系统中，使用这个方法可能会留下一段空白的存储数据。
一旦 session 驱动被注册，则必须在 ` config/session.php ` 的配置文件内使用 mongo 驱动。

