title: laravel框架34 

#  Laravel之EloquentORM入门 
Laravel 的 Eloquent ORM 提供了漂亮、简洁的 ActiveRecord 实现来和数据库进行交互。
**每个数据库表都有一个对应的「模型」可用来跟数据表进行交互。你可以通过模型查找数据表内的数据，以及将记录添加到数据表中。**
在开始之前，请确认你已在`  config/database.php ` 文件中设置好了数据库连接。
##  定义模型# 
开始之前，让我们先来创建一个 Eloquent 模型。**模型通常放在 app 目录中，不过你可以将他们随意放在任何可通过 composer.json 自动加载的地方**。所有的 Eloquent 模型都继承自 ` Illuminate\Database\Eloquent\Model ` 类。
**Summer： 建议在 app 目录下创建 Models 来统一存放。**
创建模型实例的最简单方法是使用 make:model Artisan 命令：
```

php artisan make:model User

```
当你生成一个模型时想要顺便生成一个数据库迁移，可以使用 - -migration 或 -m 选项：
```

php artisan make:model User --migration
php artisan make:model User -m

```
###  Eloquent 模型约定# 
现在，让我们来看一个 Flight 模型类的例子，我们将会用它从 flights 数据表中取回与保存信息：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}

```
###  数据表名称# 
**请注意，我们并没有告诉 Eloquent Flight 模型该使用哪一个数据表。**
除非数据表明确地指定了其它名称，否则将使用类的「蛇形名称」、复数形式名称来作为数据表的名称。因此在此例子中，**Eloquent 将会假设 Flight 模型被存储记录在 flights 数据表中**。你可以在模型上**定义一个 table 属性，用来指定自定义的数据表名称**：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 与模型关联的数据表。
     *
     * @var string
     */
    protected $table = 'my_flights';
}

```
**主键#**
Eloquent 也会假设每个数据表都有一个叫做 ` id ` 的主键字段。你也可以定义一个 ` $primaryKey ` 属性来重写这个约定。
**时间戳#**
默认情况下，Eloquent 会预计你的数据表中有 ` created_at 和 updated_at ` 字段。如果你不希望让 Eloquent 来自动维护这两个字段，**可在模型内将 $timestamps 属性设置为 false**：
如果你需要自定义自己的时间戳格式，**可在模型内设置 $dateFormat 属性。**这个属性决定了日期应如何在数据库中存储，以及当模型被序列化成数组或 JSON 时的格式：
```

protected $dateFormat = 'U';

```
**数据库连接#**
默认情况下，所有的 Eloquent 模型会使用应用程序中默认的数据库连接设置。**如果你想为模型指定不同的连接，可以使用 $connection 属性**：
```

protected $connection = 'connection-name';

```

##  取回多个模型# 
一旦你**创建并 关联了一个模型到数据表** 上，那么**你就可以从数据库中获取数据**。可**把每个 Eloquent 模型想像成强大的查询构造器，它让你可以流畅地查找与模型关联的数据表**。例如：
```

<?php
namespace App\Http\Controllers;

use App\Flight;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
     * 显示所有可以的航班清单。
     *
     * @return Response
     */
    public function index()
    {
        $flights = Flight::all();

        return view('flight.index', ['flights' => $flights]);
    }
}

```
**访问字段的值#**
假设你有一个 Eloquent 模型的实例，则可以通过相对应的属性来访问模型的字段值。例如，让我们遍历查找所返回的每个 Flight 实例，并且输出 name 字段的值：
```

foreach ($flights as $flight) {
    echo $flight->name;
}

```
**增加额外的限制#**
Eloquent 的 all 方法会返回在模型数据表中的所有结果。**由于每个 Eloquent 模型都可以当作一个查询构造器**，所以你可以在查找中增加规则，然后使用 get 方法来获取结果：
```

$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();

```
` 注意： 由于 Eloquent 模型是查询构造器，因此你应当去阅读所有 查询构造器 中可用的方法。你可在 Eloquent 查找中使用这其中的任何方法。 `

###  集合# 
**像是 all 以及 get 之类的可以取回多个结果的 Eloquent 方法，将会返回一个 Illuminate\Database\Eloquent\Collection 实例。**Collection 类提供 多种辅助方法 来处理你的 Eloquent 结果。当然，你也可以简单地像数组一样来遍历集合：
```

foreach ($flights as $flight) {
    echo $flight->name;
}

```
###  分块结果# 
如果你需要处理上千笔 Eloquent 查找结果，则可以使用 chunk 命令。chunk 方法将会获取一个 Eloquent 模型的「分块」，并将它们送到指定的 闭包 (Closure) 中进行处理。当你在处理大量结果时，使用 chunk 方法可节省内存：
```

Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});

```
##  取回单个模型／集合# 
当然，除了从指定的数据表取回所有记录，你也可以通过 ` find 和 first ` 方法来取回单条记录。这些方法返回的是单个模型的实例，而不是返回模型的集合：
```

// 通过主键取回一个模型...
$flight = App\Flight::find(1);
// 取回符合查找限制的第一个模型 ...
$flight = App\Flight::where('active', 1)->first();

```
**「未找到」异常#**
**有时候你可能希望在找不到模型时抛出一个异常，这在路由或是控制器内特别有用。**` findOrFail 以及 firstOrFail ` 方法会取回查找的第一个结果。如果没有找到相应结果，则会抛出一个 ` Illuminate\Database\Eloquent\ModelNotFoundException `：
```

$model = App\Flight::findOrFail(1);
$model = App\Flight::where('legs', '>', 100)->firstOrFail();

```
**如果没有捕捉到异常，则会自动地送回 HTTP 404 响应给用户**，因此当使用这些方法时，你没有必要明确的编写检查来返回 404 响应：
```

Route::get('/api/flights/{id}', function ($id) {
    return App\Flight::findOrFail($id);
});

```
当然，你也可以使用 count、sum、max，和其它 查询构造器 提供的 **聚合**函数。这些方法会返回适当的标量值，而不是一个完整的模型实例：
```

$count = App\Flight::where('active', 1)->count();
$max = App\Flight::where('active', 1)->max('price');

```

##  添加和更新模型# 
**基本添加#**
要在数据库中创建一条新记录，只需创建一个新模型实例，并在模型上设置属性和调用 save 方法即可：
```

<?php
namespace App\Http\Controllers;

use App\Flight;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
     * 创建一个新的航班实例。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 验证请求...
        $flight = new Flight;
        $flight->name = $request->name;
        $flight->save();
    }
}

```
在这个例子中，我们把进来的 HTTP 请求的 name 参数简单地指定给 App\Flight 模型实例的 name 属性**。当我们调用 save 方法，就会添加一条记录到数据库中。当 save 方法被调用时，created_at 以及 updated_at 时间戳将会被自动设置，因此我们不需要去手动设置它们。**

**基本更新#**
save 方法也可以用于更新数据库中已经存在的模型。要更新模型，则须先取回模型，再设置任何你希望更新的属性，接着调用 save 方法。同样的，updated_at 时间戳将会被自动更新，所以我们不需要手动设置它的值：
```

$flight = App\Flight::find(1);
$flight->name = 'New Flight Name';
$flight->save();

```
也可以针对符合指定查找的任意数量模型进行更新。在这个例子中，所有 active 并且 destination 为 San Diego 的航班，都将会被标识为延迟：
```

App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);

```
update 方法会预计收到一个字段与值成对的数组，来代表应该被更新的字段。

###  批量赋值# 
你也可以使用 create 方法在一行代码上保存一个新模型。被添加的模型实例将会从你的方法中返回。
然而，` 在这样做之前，你需要先在你的模型上指定一个 fillable 或 guarded 属性，因为所有的 Eloquent 模型都有针对批量赋值（Mass-Assignment）做保护。 `
你可以在模型上使用 $fillable 属性来实现。例如，让我们使 Flight 模型的 name 属性可以被批量赋值：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;
class Flight extends Model
{
    /**
     * 可以被批量赋值的属性。
     *
     * @var array
     */
    protected $fillable = ['name'];
}

```
一旦我们已经设置好可以被批量赋值的属性，便能通过 create 方法来添加一条新记录到数据库。create 方法返回已经被保存的模型实例：
```

$flight = App\Flight::create(['name' => 'Flight 10']);

```
**$fillable 作为一个可以被批量赋值的属性「白名单」**。另外你也可以选择使用 $guarded。
$guarded 属性应该包含一个你不想要被批量赋值的属性数组。所有不在数组里面的其它属性都可以被批量赋值。
因此，**$guarded 的功能更像是一个「黑名单」**。使用的时候应该**只选择 $fillable 或 $guarded 中的其中一个**：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 不可被批量赋值的属性。
     *
     * @var array
     */
    protected $guarded = ['price'];
}

```
在上面的例子当中，所有属性 除了 price 以外，都可以被批量赋值。

**其它创建的方法#**
` firstOrCreate 和 firstOrNew `将会使用指定的字段／值对，来尝试寻找数据库中的记录。如果在数据库中找不到模型，则会使用指定的属性来添加一条记录，并返回一个创建的模型实例。
**请注意 firstOrnew 返回的模型还尚未保存到数据库。你需要通过手动调用 save 方法来保存它：**
```

// 用属性取回航班，当结果不存在时创建它...
$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);
// 用属性取回航班，当结果不存在时实例化一个新实例...
$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

```

##  删除模型# 
要删除模型，必须在模型实例上调用 delete 方法：
```

$flight = App\Flight::find(1);
$flight->delete();

```
**通过键来删除现有的模型#**
然而，如果你已知道了模型中的主键，则可以不用取回模型就能直接删除它。若要这么做，请调用 destroy 方法：
```

App\Flight::destroy(1);
App\Flight::destroy([1, 2, 3]);
App\Flight::destroy(1, 2, 3);

```
**通过查找来删除模型#**
```

$deletedRows = App\Flight::where('active', 0)->delete();

```

###  软删除# 
除了从数据库中移除实际记录，Eloquent 也可以「软删除」模型。当模型被软删除时，它们并不会真的从数据库中被移除。而是会在模型上设置一个`  deleted_at ` 属性并将其添加到数据库。如果模型有一个非空值 deleted_at，代表模型已经被软删除了。**要在模型上启动软删除，则必须在模型上使用 Illuminate\Database\Eloquent\SoftDeletes trait 并添加 deleted_at 字段到你的 $dates 属性上：**
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;

    /**
     * 需要被转换成日期的属性。
     *
     * @var array
     */
    protected $dates = ['deleted_at'];
}

```
当然，你也应该添加 deleted_at 字段到数据表中。Laravel 结构生成器 包含了一个用来创建此字段的辅助方法：
```

Schema::table('flights', function ($table) {
    $table->softDeletes();
});

```
现在，当你在模型上调用 delete 方法时，deleted_at 字段将会被设置成目前的日期和时间。而且，当查找有启用软删除的模型时，被软删除的模型将会自动从所有查找结果中排除。
**要确认指定的模型实例是否已经被软删除，可以使用 trashed 方法：**
```

if ($flight->trashed()) {
    //
}

```
**查找被软删除的模型#**
如上所述，被软删除的模型将会自动从所有的查找结果中排除。然而，你可以通过在查找中**调用 withTrashed 方法来强制查找已被软删除的模型：**
```

$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();

```
withTrashed 方法也可以被用在 关联 查找：
```

$flight->history()->withTrashed()->get();

```
**Where 子句注意事项#**
` 当在你的软删除模型查找增加了 orWhere 子句时，请使用 高级 where 子句 来对 WHERE 子句的逻辑进行分组。 `例如：
```

User::where(function($query) {
        $query->where('name', '=', 'John')
              ->orWhere('votes', '>', 100);
        })
        ->get();

```
这将生成以下 SQL：
```

select * from `users` where `users`.`deleted_at` is null and (`name` = 'John' or `votes` > 100)

```
如果 orWhere 子句没有被分组，它将会在包含软删除的纪录中生成以下 SQL：
```

select * from `users` where `users`.`deleted_at` is null and `name` = 'John' or `votes` > 100

```
**只获取被软删除的模型#**
onlyTrashed 方法会 只 获取已被软删除的模型：
```

$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();

```
**恢复被软删除的模型#**
```

$flight->restore();

```
你也可以在查找上使用 restore 方法来快速地恢复多个模型：
```

App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();

```
**永久地删除模型#**
```

// 强制删除单个模型实例...
$flight->forceDelete();

// 强制删除所有相关模型...
$flight->history()->forceDelete();

```

##  查找范围# 
**范围（Scopes）可让你定义有限制的共用集合**，它可以轻松地在你的应用程序中被重复使用。例如，你可能需要频繁地获取所有被认为是「受欢迎的」用户。
要定义此范围，则可以简单地在 Eloquent 模型方法前面加上` 前缀 scope `：
**范围应总是返回查询构造器的实例**：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 限制查找只包括受欢迎的用户。
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    /**
     * 限制查找只包括活跃的用户。
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
}

```
**利用查找范围#**
一旦定义了范围，则可以在查找模型时调用范围方法。**在进行方法调用时不需要加上 scope 前缀**。你甚至可以链式调用不同的范围，如：
```

$users = App\User::popular()->active()->orderBy('created_at')->get();

```
**动态范围#**
有时候，你可能希望定义一个可接受参数的范围。这时只需给你的范围加**上额外的参数**即可。范围参数应该被定义在 $query 参数之后：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 限制查找只包括指定类型的用户。
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}

```
现在，你可以在范围调用时传递参数：
```

$users = App\User::ofType('admin')->get();

```
##  事件# 
**Eloquent 模型会触发许多事件，让你可以借助以下的方法在模型的生命周期的多个时间点进行操作：creating、created、updating、updated、saving、saved、deleting、deleted、restoring、restored。**事件可以让你每当有特定的模型类在数据库中被保存或更新时简单地运行代码。
当一个新模型被初次保存将会触发 creating 以及 created 事件。如果一个模型已经存在于数据库且调用了 save 方法，将会触发 updating 和 updated 事件。在这两种情况下都会触发 saving 和 saved 事件。
让我们在 服务提供者 中定义一个 Eloquent 事件侦听器来作为示例。在我们的事件侦听器中，我们会在指定的模型上调用 isValid 方法，并在模型无效时返回 false。**从 Eloquent 事件侦听器返回 false 会取消 save 和 update 的操作：**
```

<?php

namespace App\Providers;

use App\User;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 启动所有应用程序服务。
     *
     * @return void
     */
    public function boot()
    {
        User::creating(function ($user) {
            if ( ! $user->isValid()) {
                return false;
            }
        });
    }

    /**
     * 注册服务提供者。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}

```