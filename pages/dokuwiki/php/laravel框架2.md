title: laravel框架2 

#  Laravel之HTTP路由 
http://laravel-china.org/docs/5.1/routing
你可以在`  app/Http/routes.php ` 文件中**定义应用程序的大多数路由**，该文件将会被 ` App\Providers\RouteServiceProvider ` 类加载。最基本的 Laravel 路由仅接受 URI 和一个闭包：
```

Route::get('/', function () {
    return 'Hello World';
});
Route::post('foo/bar', function () {
    return 'Hello World';
});
Route::put('foo/bar', function () {
    //
});
Route::delete('foo/bar', function () {
    //
});

```
**为多重动作注册路由**
有时候你可能需要注册一个可响应多个 HTTP 动作的路由。这时可通过 Route facade 的`  match ` 方法来实现：
```

Route::match(['get', 'post'], '/', function () {
    return 'Hello World';
});

```
或者，你甚至可以通过`  any 方法 `来使用注册路由并响应所有的 HTTP 动作：
```

Route::any('foo', function () {
    return 'Hello World';
});

```
**产生 URLs 路由**
你可以通过 url 辅助函数产生应用程序路由：
$url = url('foo');

##  路由参数 
```

Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});

```
注意：路由参数不能包含 - 字符。请用底线 (_) 替换。
**可选的路由参数**
有时候你可能需要指定路由参数，但是让路由参数的存在是可选的。这时可以在参数名称后面加上 ? 来实现：
```

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});

```
**正则表达式限制参数**
你可以在路由实例上使用 ` where 方法 `来限制路由参数格式。where 方法接受参数的名称和定义参数应该如何被限制的正则表达式：
```

Route::get('user/{name}', function ($name) {
    //
})
->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})
->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})
->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

```

**全局限制**
如果你希望路由参数可以总是遵循正则表达式，则可以使用 ` pattern 方法 `。你应该在 ` RouteServiceProvider 的 boot 方法 `里定义这些模式：
```

/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @param  \Illuminate\Routing\Router  $router
 * @return void
 */
public function boot(Router $router)
{
    $router->pattern('id', '[0-9]+');
    parent::boot($router);
}

```
模式一旦被定义，便会**自动应用**到所有使用该参数名称的路由上：
```

Route::get('user/{id}', function ($id) {
    // Only called if {id} is numeric.
});

```

##  命名路由 
命名路由让你可以更方便的**为特定路由产生 URL 或进行重定向**。你可以使用`  as 数组键 `指定名称到路由上：
```

Route::get('user/profile', ['as' => 'profile', function () {
    //
}]);

```
还可以指定路由名称到控制器动作：
```

Route::get('user/profile', [
    'as' => 'profile', 'uses' => 'UserController@showProfile'
]);

```
除了可以在路由的数组定义中指定路由名称外，你也可以在路由定义后方链式调用`  name 方法 `：
```

Route::get('user/profile', 'UserController@showProfile')->name('profile');

```
**路由群组和命名路由**
如果你使用了路由群组，那么你可以在路由群组的属性数组中指定一个 as 关键字，这将允许你为路由群组中的所有路由设置相同的**前缀名称**：
```

Route::group(['as' => 'admin::'], function () {
    Route::get('dashboard', ['as' => 'dashboard', function () {
        // 路由名称为「admin::dashboard」
    }]);
});

```
**对命名路由产生 URLs**
一旦你在指定的路由中分配了名称，则可通过 ` route 函数 `来使用路由名称产生 URLs 或重定位：
```

$url = route('profile');
$redirect = redirect()->route('profile');

```
如果路由定义了参数，那么你**可以把参数作为第二个参数传递给 route 方法**。指定的参数将自动加入到 URL 中：
```

Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
    //
}]);
$url = route('profile', ['id' => 1]);

```
##  路由群组 
**路由群组允许你共用路由属性，例如：中间件、命名空间，**你可以利用路由群组到多个路由套用这些属性，而不需在每个路由都设置一次。**共用属性被指定为数组格式**，当作`  Route::group ` 方法的第一个参数：
**中间件**
要想指定中间件到所有群组内的路由中，你可以在群组属性数组里使用`  middleware 参数 `。中间件将会依照列表内指定的顺序运行：
```

Route::group(['middleware' => 'auth'], function () {
    Route::get('/', function ()    {
        // 使用 Auth 中间件
    });
    Route::get('user/profile', function () {
        // 使用 Auth 中间件
    });
});

```

**命名空间**
另一个常见的例子是，**指定相同的 PHP 命名空间给控制器群组**。可以使用 ` namespace 参数 `来指定群组内所有控制器的命名空间：
```

Route::group(['namespace' => 'Admin'], function()
{
    // 控制器在「App\Http\Controllers\Admin」命名空间

    Route::group(['namespace' => 'User'], function()
    {
        // 控制器在「App\Http\Controllers\Admin\User」命名空间
    });
});

```
请记住，**默认 RouteServiceProvider 会在命名空间群组内导入你的 routes.php 文件，让你不用指定完整的 App\Http\Controllers 命名空间前缀就能注册控制器路由。**所以，我们只需要指定在基底 ` App\Http\Controllers ` 根命名空间之后的部分命名空间。

**子域名路由**
路由群组也可以被用来做**处理通配符的子域名**。子域名可以像路由 URIs 分配路由参数，让你在路由或控制器中获取子域名参数。使用路由群组属性数组上的 domain 指定子域名变量名称：
```

Route::group(['domain' => '{account}.myapp.com'], function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});

```
**路由前缀**
通过路由群组数组属性中的`  prefix `，在路由群组内**为每个路由指定的 URI 加上前缀**。例如，你可能想要在路由群组中将所有的路由 URIs 加上前缀 admin：
```

Route::group(['prefix' => 'admin'], function () {
    Route::get('users', function ()    {
        // 符合「/admin/users」URL
    });
});

```
你也可以**使用 prefix 参数去指定路由群组中共用的参数**：
```

Route::group(['prefix' => 'accounts/{account_id}'], function () {
    Route::get('detail', function ($account_id)    {
        // 符合 accounts/{account_id}/detail URL
    });
});

```

##  CSRF 保护 
Laravel 提供简单的方法保护你的应用程序不受到 跨网站请求伪造 攻击。跨网站请求伪造是一种恶意的攻击，借以通过经过身份验证的用户身份运行未经授权的命令。
Laravel 会自动产生一个** CSRF token** 给每个受应用程序管理的活动用户的 Session。该 token 用来验证用户是否为实际发出请求至应用程序的用户。若要产生一个包含 CSRF token 的 ` _token 隐藏输入字段 `，可以使用 csrf_field 辅助函数：
```

<?php echo csrf_field(); ?>

```
csrf_field 辅助函数产生以下的 HTML：
<input type="hidden" name="` _token `" value="<?php echo csrf_token(); ?>">
当然，也可以在** Blade 模板引擎** 中使用：
```

![](/data/dokuwiki csrf_field() )

```
你不需要手动验证 POST、PUT 或 DELETE 请求的 CSRF token。**在 VerifyCsrfToken HTTP 中间件 将自动验证请求与 session 中的 token 是否相符。**

**不受 CSRF 保护的 URIs**
**有时候你可能会希望一组 URIs 不要被 CSRF 保护。**例如，你如果使用 Stripe 处理付款，并且利用他们的 webhook 系统，你需要从 Laravel CSRF 保护中排除 webhook 的处理路由。
**可以在 VerifyCsrfToken 中间件中增加`  $except 属性 `来排除 URIs**：
```

<?php
namespace App\Http\Middleware;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

class VerifyCsrfToken extends BaseVerifier
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
    ];
}


```
**X-CSRF-TOKEN**
除了检查 CSRF token 是否被当作 POST 参数之外，在 Laravel VerifyCsrfToken 中间件也会检查**请求标头中的 X-CSRF-TOKEN**。例如，你可以将其保存在 meta 标签中：
```

<meta name="csrf-token" content="![](/data/dokuwiki csrf_token() )">

```
一旦你创建了 meta 标签，你就可以使用 **jQuery** 之类的函数库将 token 加入到所有的请求标头。**基于 AJAX 的应用，提供了简单、方便的 CSRF 保护**：
```

$.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
});

```
**Laravel 也会在 XSRF-TOKEN cookie 中保存 CSRF token。你也可以使用 cookie 的值来设置 ` X-XSRF-TOKEN ` 请求标头。**一些 JavaScript 框架会自动帮你处理，例如：Angular。你不大可能会需要手动去设置这个值。

其余HTTP方法支持：
HTML 表单没有支持 PUT、PATCH 或 DELETE 动作。所以在从 HTML 表单中调用被定义的 PUT、PATCH 或 DELETE 路由时，你将需要在表单中增加` 隐藏的 _method 字段 `。跟随 _method 字段送出的值将被作为 HTTP 的请求方法使用：
```

<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="![](/data/dokuwiki csrf_token() )">
</form>

```
你也可以使用 methid_field 辅助函数来产生隐藏的输入字段 _method：
```

<?php echo method_field('PUT'); ?>

```
当然，也可以使用 Blade 模板引擎：
```

![](/data/dokuwiki method_field('PUT') )

```
##  路由模型绑定 
Laravel 路由模型绑定提供了一个方便的方式来注入类实例到你的路由中。例如，除了注入一个用户的 ID，你也可以注入与指定 ID 相符的完整 User 类实例。
首先，使用路由的`  model 方法 `为指定参数指定类。必须在`  RouteServiceProvider::boot 方法 `中**定义你的模型绑定**：
绑定参数至模型
```

public function boot(Router $router)
{
    parent::boot($router);
    $router->model('user', 'App\User');
}

```
接着，定义包含 {user} 参数的路由：
```

$router->get('profile/{user}', function(App\User $user) {
    //
});

```
因为我们已经绑定 {user} 参数至 App\User 模型，**所以 User 实例会被注入至该路由。**所以，举个例子，一个至 profile/1 的请求会注入 ID 为 1 的 User 实例。
` 注意：如果符合的模型不存在于数据库中，就会自动抛出一个 404 异常。 `
如果你希望**指定你自己的「不存在」行为**，只需传递一个闭包作为 model 方法的第三个参数：
```

$router->model('user', 'App\User', function() {
    throw new NotFoundHttpException;
});

```
如果**你希望使用你自己的解析逻辑**，那么你必须使用 ` Route::bind 方法 `。你传递至 bind 方法的闭包会获取 URI 的部分值，且会返回你想注入至路由的类实例：
```

$router->bind('user', function($value) {
    return App\User::where('name', $value)->first();
});

```


##  抛出 404 错误 
这里有两种方法从路由手动触发 404 错误。首先，你可以使用 ` abort 辅助函数 `。abort 辅助函数只是简单的抛出一个带有指定状态代码的 ` Symfony\Component\HttpFoundation\Exception\HttpException `：
```

abort(404);

```
其次，你也可以手动抛出 Symfony\Component\HttpKernel\Exception\NotFoundHttpException 的实例。
更多有关如何操作 404 异常和自定义响应，可以到[错误](http://laravel-china.org/docs/5.1/errors#http-exceptions)章节内参考文档。