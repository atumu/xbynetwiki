title: laravel框架30 

#  Laravel之数据库操作入门 
Laravel 对主流的数据库系统的连接和查询都提供了很好的支持，无论是原始的 SQL，还是流畅的查询语句构造器，或是强大的 Eloquent ORM。
目前，Laravel 支持以下四种数据库系统：
  * MySQL
  * Postgres
  * SQLite
  * SQL Server
Summer： Mongo DB 的支持可以使用这个项目 [laravel-mongodb](https://github.com/jenssegers/laravel-mongodb)

##  配置信息# 
Laravel 应用程序的数据库配置文件放置在 ` config/database.php `。
在这个配置文件内你可以**定义所有的数据库连接，以及指定默认使用哪个连接**。在此文件内提供了所有支持的数据库系统示例。
###  数据库读写分离# 
有时候你也许会希望使用一个数据库作为只读数据库，而另一个数据库则负责写入、更新以及删除。
如何设置读取与写入的连接，让我们看下这个例子：
```

'mysql' => [
    'read' => [
        'host' => '192.168.1.1',
    ],
    'write' => [
        'host' => '196.168.1.2'
    ],
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
],

```
注意，有两个键加入了这个配置文件数组内` ：read 及 write `。

##  运行原始 SQL 
` DB facade ` 提供每个类型的查找方法：` select、update、insert、delete、statement。 `
运行一个 Select 查找#
```

<?php
namespace App\Http\Controllers;

use DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示应用程序中所有用户的列表。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::select('select * from users where active = ?', [1]);
        return view('user.index', ['users' => $users]);
    }
}

```
传递给 select 方法的第一个参数是原始的 SQL 查找，而第二个参数是任何查找所需要的参数绑定。
select 方法总会返回结果的数组数据。**数组中的每个结果都是一个 PHP StdClass 对象**，这使你能够访问到结果的值：
```

foreach ($users as $user) {
    echo $user->name;
}

```
**使用命名绑定#**
除了使用 ? 来表示你的参数绑定外，你也可以使用命名绑定运行查找：
```

$results = DB::select('select * from users where id = :id', ['id' => 1]);

```
**运行 Insert#**
```

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

```
**运行 Update#**
update 方法用于更新已经存在于数据库的记录。该方法会返回此声明所影响的行数：
```

$affectedCount = DB::update('update users set votes = 100 where name = ?', ['John']);

```
**运行 Delete#**
delete 方法用于删除已经存在于数据库的记录。如同 update 一样，删除的行数将会被返回：
```

$deletedCount = DB::delete('delete from users');

```
**运行一般声明#**
有时候一些数据库操作不应该返回任何参数。对于这种类型的操作，你可以在 DB facade 使用 statement 方法：
```

DB::statement('drop table users');

```

###  监听SQL事件# 
如果你希望能够收到来自于应用程序的每一条 SQL 查找，则可以使用 listen 方法。这个方法对于纪录查找跟调试将非常有用。你可以在服务容器中注册你的查找侦听器：
```

<?php
namespace App\Providers;

use DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 启动任何应用程序的服务。
     *
     * @return void
     */
    public function boot()
    {
        DB::listen(function($sql, $bindings, $time) {
            //
        });
    }

    /**
     * 注册一个服务提供者。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}

```

##  数据库事务# 
在 ` DB facade 中使用 transaction 方法 `。如果在事务的闭包内抛出异常，事务将会被自动还原。如果闭包运行成功，事务将被自动提交。你不需要担心在使用 transaction 方法时还需要亲自去手动还原或提交事务：
```

DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});

```
**手动操作事务#**
如果你想手动处理事务并对还原或提交操作进行完全控制，则可以在 DB facade 使用 beginTransaction 方法：
```

DB::beginTransaction();
你也可以通过 rollBack 方法来还原事务：
DB::rollBack();
最后，可以通过 commit 方法来提交这个事务：
DB::commit();

```
**注意： DB facade 的事务方法也可以用来控制查询语句构造器及 Eloquent ORM 的事务。**

##  使用多数据库连接# 
当你使用了多个连接时，则可以通过 DB facade 的 connection 方法来访问每个连接。**传递给 connection 方法的 name 必须对应至 config/database.php 配置文件中的连接列表的其中一个**：
```

$users = DB::connection('foo')->select(...);

```
你也可以在连接的实例中使用 getPdo 方法**访问原始的底层 PDO 实例**：
```

$pdo = DB::connection()->getPdo();

```