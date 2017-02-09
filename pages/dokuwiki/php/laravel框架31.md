title: laravel框架31 

#  Laravel查询构造器 
数据库查询构造器提供了方便、流畅的接口，以用来创建及运行数据库查找。
注意：**Laravel 的查询构造器使用 PDO 参数绑定，以保护你的应用程序不受数据库注入攻击**。在传入字符串作为绑定前不需要先清理它们。
##  获取结果# 
**从数据表中获取所有的数据列#**
若要开始进行查找，可在`  DB facade 上使用 table 方法 `。table 方法会针对指定的数据表返回一个查询构造器实例，允许你在查询时**链式调用**更多约束，并得到最终结果。在这个例子中，我们将会从一个数据表中来获取所有的记录：
```

<?php
namespace App\Http\Controllers;

use DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * 显示应用程序的所有用户列表。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}

```
就像原始查找一样，get 方法会返回一个数组结果，其中每一个结果都是 PHP StdClass 对象的实例。你可以将列作为对象的属性来访问每个列的值：
```

foreach ($users as $user) {
    echo $user->name;
}

```
**从数据表中获取单个列或行#**
**若你只需从数据表中取出单行数据，则可以使用 first 方法**。这个方法会返回单个 StdClass 对象：
```

$user = DB::table('users')->where('name', 'John')->first();
echo $user->name;

```
若你不想取出完整的一行，则可以使用 value 方法来**从单条记录中取出单个值**。这个方法会直接返回字段的值：
```

$email = DB::table('users')->where('name', 'John')->value('email');

```
###  从数据表中将结果切块# 
**若你需要操作数千条数据库记录，则可考虑使用 chunk 方法。这个方法一次只取出一小「块」结果，并会将每个区块传给一个闭包进行处理。**这个方法对于要编写处理数千条记录的 Artisan 命令非常有用。例如，让我们将整个 user 数据表进行切块，每次处理 100 条记录：
```

DB::table('users')->chunk(100, function($users) {
    foreach ($users as $user) {
        //
    }
});

```
**你可以从闭包中返回 false，以停止对后续切块的处理：**
```

DB::table('users')->chunk(100, function($users) {
    // 处理记录…
    return false;
});

```
**获取字段值列表#**
若你想要获取一个包含单个字段值的数组，你可以使用 ` lists 方法 `。在这个例子中，我们将取出 role 数据表 title 字段的数组：
```

$titles = DB::table('roles')->lists('title');
foreach ($titles as $title) {
    echo $title;
}

```
你也可以在返回的数组中指定自定义的键值字段：
```

$roles = DB::table('roles')->lists('title', 'name');
foreach ($roles as $name => $title) {
    echo $title;
}

```
###  聚合# 
查询构造器也提供了各种聚合方法，例如 count、max、min、avg、以及 sum。你可以在创建查找后调用其中的任意一个方法：
```

$users = DB::table('users')->count();
$price = DB::table('orders')->max('price');

```
当然，你也可以将这些方法合并到其它的子句上来构建查找：
```

$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');

```
##  Selects# 
指定一个 Select 子句#
当然，你并不会总是想从数据表中选出所有的字段。这时可以使用 select 方法为查找指定一个自定义的 select 子句：
```

$users = DB::table('users')->select('name', 'email as user_email')->get();
$users = DB::table('users')->distinct()->get();

```
若你已有一个查询构造器实例，且希望在其现存的 select 子句中加入一个字段，则可以使用 addSelect 方法：
```

$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();

```
**原始表达式#**
有时你可能需要在查找中使用原始表达式。这些表达式会被当作字符串注入到查找中，因此要小心避免造成数据库注入攻击！要创建一个原始表达式，可以使用 DB::raw 方法：
```

$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();

```
##  Joins# 
**Inner Join 语法#**
```

$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();

```
**Left Join 语法#**
```

$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

```
**高级的 Join 语法#**
你也可以指定更高级的 join 子句。让我们以传入一个闭包当作 join 方法的第二参数来作为开始。` 此闭包会接收 JoinClause 对象 `，让你可以在 join 子句上指定约束：
```

DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();

```
**若你想要在连接中使用「where」风格的子句，则可以在连接中使用 where 和 orWhere 方法**。这些方法会比较字段和一个值，来代替两个字段的比较.

##  Unions# 
你可以先创建一个初始查找，然后使用 union 方法将它与第二个查找进行合并：
```

$first = DB::table('users')
            ->whereNull('first_name');
$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();

```
也可使用 unionAll 方法，它和 union 有着相同的方法签名。

##  Where 子句# 
**简单的 Where 子句#**
```

$users = DB::table('users')->where('votes', '=', 100)->get();
$users = DB::table('users')->where('votes', 100)->get();

```
当然，在编写 where 子句时，你**可以使用数据库所支持的任何运算符**：
```

$users = DB::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = DB::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = DB::table('users')
                ->where('name', 'like', 'T%')
                ->get();

```
###  OrWhere 语法# 
` orWhere ` 方法和 where 方法接受相同的参数：
```

$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();

```
###  whereBetween、whereNotBetween 
```

$users = DB::table('users')
                    ->whereBetween('votes', [1, 100])->get();
$users = DB::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();

```
###  whereIn 与 whereNotIn 
```

$users = DB::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
$users = DB::table('users')
                    ->whereNotIn('id', [1, 2, 3])
                    ->get();

```
###  whereNull 与 whereNotNull 
```

$users = DB::table('users')
                    ->whereNull('updated_at')
                    ->get();
$users = DB::table('users')
                    ->whereNotNull('updated_at')
                    ->get();

```

##  高级的 Where 子句# 
###  参数分组# 
有时你可能会需要创建更高级的 where 子句，例如「where exists」或者嵌套的参数分组。Laravel 的查询语句构造器也能处理这些。让我们先来看下一个在括号中将约束分组的例子：
```

DB::table('users')
            ->where('name', '=', 'John')
            ->orWhere(function ($query) {
                $query->where('votes', '>', 100)
                      ->where('title', '<>', 'Admin');
            })
            ->get();

```
如你所见，上面例子会**将闭包传入 orWhere 方法，以告诉查询语句构造器开始一个约束分组**。此闭包接收一个查询语句构造器的实例，你可用它来设置应包含在括号分组内的约束。
上面的例子会生成以下 SQL：
```

select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

```
###  Exists 语法# 
**whereExists 方法允许你编写 where exists SQL 子句。**
```

DB::table('users')
            ->whereExists(function ($query) {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();

```
上述的查找会生成以下的 SQL：
```

select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)

```
##  Ordering、Grouping、Limit 及 Offset# 
**orderBy#**
```

$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();

```
**groupBy、having 与 havingRaw、skip、take#**
groupBy 和 having 方法可用来将查找结果进行分组。having 方法的签名和 where 方法的类似：
```

$users = DB::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();

```
**havingRaw 方法**可用来将原始字符串设置为 having 子句的值。例如，我们可以找出所有销售额大于 2,500 元的部门：
```

$users = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > 2500')
                ->get();

```
**skip 与 take#**
**要限制查找所返回的结果数量，或略过指定数量的查找结果（偏移），则可使用 skip 和 take 方法**：
```

$users = DB::table('users')->skip(10)->take(5)->get();

```

##  Insert、update、delete 
**insert 方法**接收一个数组，包含要插入的字段名称及值：
```

DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);

```
你甚至可以**在 insert 调用中传入一个嵌套数组，来一次性插入多条记录到数据表中**。每个数组代表要插入数据表中的一列记录：
```

DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);

```
**若数据表有自动递增的 id，则可使用 insertGetId 方法来插入记录并获取其 ID：**
```

$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0],'user_id'
);

```
注意：当使用 PostgreSQL 时，insertGetId 方法将预测自动递增字段的名称为 id。**若你要从不同「顺序」来获取 ID，则可以将顺序名称作为第二个参数传给 insertGetId 方法。**

**Updates#**
update 方法和 insert 方法一样，接收含有一对字段及值的数组，其中包含了要被更新的字段。可使用 where 子句来约束 update 查找：
```

DB::table('users')
            ->where('id', 1)
            ->update(['votes' => 1]);

```
**查询语句构造器也提供了便利的方法来递增或递减指定字段的值。**
这两个方法都必须接收至少一个参数（要修改的字段）。也可选择性地传入第二个参数，用来控制字段应递增／递减的量。
```

DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);

```
你也可以指定要在操作中**更新其它字段**：
```

DB::table('users')->increment('votes', 1, ['name' => 'John']);

```

**Deletes#**
```

DB::table('users')->delete();
DB::table('users')->where('votes', '<', 100)->delete();

```
若你希望截去整个数据表的所有数据列，并将自动递增 ID 重设为零，则可以使用 truncate 方法：
```

DB::table('users')->truncate();

```

##  悲观锁定# 
查询语句构造器也包含一些可用以协助你在 select 语法上作「悲观锁定」的函数。若要以「共享锁」来运行语句，则可在查找上使用 sharedLock 方法。共享锁可避免选择的数据列被更改，直到事务被提交为止：
```

DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

```
此外，你也可以使用 lockForUpdate 方法。「用以更新」锁可避免数据列被其它共享锁修改或选取：
```

DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

```