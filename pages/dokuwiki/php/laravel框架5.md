title: laravel框架5 

#  Laravel之HTTP请求与响应 
##  HTTP 请求 
获取请求
  * 基本请求信息
  * PSR-7 请求
获取输入数据
  * 旧输入数据
  * Cookies
  * 上传文件
**获取请求**
要**通过依赖注入的方式获取 HTTP 请求的实例**，就必须在控制器的构造器或方法中，使用 ` Illuminate\Http\Request ` 类型提示。当前的请求实例便会自动由服务容器注入：
```

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    /**
     * 保存新的用户。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}

```
**基本请求信息**
Illuminate\Http\Request 的实例提供了多种方法来用于检查应用程序的 HTTP 请求。` Larevel 的 Illuminate\Http\Request 继承了 Symfony\Component\HttpFoundation\Request 类 `。下方是该类的几个有用的方法：
**获取请求的 URI**
path 方法会返回请求的 URI。所以，如果接收到的请求目标是 http://domain.com/foo/bar，那么 path 方法就会返回 foo/bar：
```

$uri = $request->path();

```
is 方法可以验证接收到的**请求 URI 与指定的规则是否相匹配**。使用此方法时你可以**将 * 符号作为通配符**：
```

if ($request->is('admin/*')) {
    //
}

```
若要**获取完整的网址**，而不是仅有路径信息，则可以对请求实例使用 url 方法：
```

$url = $request->url();

```
**获取请求的方法**
```

$method = $request->method();
if ($request->isMethod('post')) {
    //
}

```

###  PSR-7 请求 
**PSR-7 标准制定的 HTTP 消息接口包含了请求及响应。**
如果你想获得一个 PSR-7 的请求实例，你就必须先安装几个函数库。Laravel 使用 Symfony 的 HTTP 消息桥接组件，将原 Laravel 的请求及响应转换至 PSR-7 所支持的实现：
```

composer require symfony/psr-http-message-bridge
composer require zendframework/zend-diactoros

```
**只要你安装完这些函数库，就可以在路由或控制器中，简单的对请求类型使用类型提示来获取 PSR-7 的请求**：
```

use Psr\Http\Message\ServerRequestInterface;
Route::get('/', function (ServerRequestInterface $request) {
    //
});

```
如果你从路由或控制器返回一个 PSR-7 的响应实例，那么它会被框架自动转换回 Laravel 的响应实例并显示。

###  获取输入数据 
获取特定输入值
```

$name = $request->input('name');
$name = $request->name;
//带默认值：
$name = $request->input('name', 'Sally');

```
如果是**「数组」形式**的输入数据，则可以使用「点」语法来获取数组：
```

$input = $request->input('products.0.name');

```
**确认是否有输入值**
当该数据存在并且字符串不为空时，has 方法就会传回 true：
```

if ($request->has('name')) {
    //
}

```
**获取所有输入数据**
```

$input = $request->all();

```
**获取部分输入数据**
如果你想获取输入数据的子集，你可以使用`  only 及 except 方法 `。这两个方法**都接受单个数组或是动态列表作为参数**：
```

$input = $request->only(['username', 'password']);
$input = $request->only('username', 'password');
$input = $request->except(['credit_card']);
$input = $request->except('credit_card');

```
###  旧输入数据 
**Laravel 可以让你将本次的输入数据保留到下一次请求发送前。**对于在表单验证失败后重新填入表单值相当有用。当然，如果你使用 Laravel 的验证服务，你就不需要再手动使用这些方法，因为 Laravel 的一些内置验证功能会自动调用它们。
**将输入数据闪存至 Session**
Illuminate\Http\Request 实例的 **flash 方法会将当前的输入数据存进 Session 中**，所以下次用户发出请求至应用程序时就可以使用它们：
```

$request->flash();

```
你也可以使用`  flashOnly 及 flashExcept 方法 `将请求数据的子集保存至 Session：
```

$request->flashOnly('username', 'email');
$request->flashExcept('password');

```
**闪存输入数据至 Session 后重定向**
你可能常常需要将输入数据闪存并重定向至前一页，这时只要在重定向方法后加上`  withInput ` 就行了：
```

return redirect('form')->withInput();
return redirect('form')->withInput($request->except('password'));

```
**获取旧输入数据**
若要获取上一次请求后所闪存的输入数据，你可以使用 Request 实例中的`  old 方法 `。old 方法提供一个简便的方式从 Session 取出被闪存的输入数据：
```

$username = $request->old('username');

```
Laravel 也提供了全局辅助方法 old。如果你要**在 Blade 模板中显示旧输入数据**，可以使用更加方便的 old 辅助方法：
```

![](/data/dokuwiki old('username') )

```

###  Cookies 
**从请求取出 Cookie 值**
**Laravel 框架创建的每个 cookie 都会被加密并且加上认证标识，这代表着用户擅自更改的 cookie 都会失效。**若要从此次请求获取 cookie 值，你可以使用 Illuminate\Http\Request 实例中的 cookie 方法：
```

$value = $request->cookie('name');

```
**将新的 Cookie 附加到响应**
Laravel 提供了全局辅助方法 cookie，可通过简易的工厂产生新的 ` Symfony\Component\HttpFoundation\Cookie ` 实例。可以在 Illuminate\Http\Response 实例之后加上 ` withCookie 方法 `来把 cookie 附加至响应：
```

$response = new Illuminate\Http\Response('Hello World');
$response->withCookie(cookie('name', 'value', $minutes));
return $response;

```

##  HTTP响应 
基本响应
  * 附加标头至响应
  * 附加 Cookies 至响应
其它响应类型
  * 视图响应
  * JSON 响应
  * 文件下载
重定向
  * 重定向至命名路由
  * 重定向至控制器行为
  * 重定向并加上 Session 闪存数据
响应宏

###  基本响应 
最基本的响应就是从路由或控制器简单的返回一个字符串：
```

Route::get('/', function () {
    return 'Hello World';
});

```
指定的字符串会被框架自动转换成 HTTP 响应。
但是对大部分的路由及控制器所运行的动作来说，**一般需要的是返回完整的 Illuminate\Http\Response 实例或是一个视图。**返回一个完整的 Response 实例时，就能够自定义响应的 HTTP 状态码以及标头。Response 实例继承了 ` Symfony\Component\HttpFoundation\Response ` 类，其提供了很多创建 HTTP 响应的方法：
```

use Illuminate\Http\Response;

Route::get('home', function () {
    return (new Response($content, $status))
                  ->header('Content-Type', $value);
});

```
为了方便起见，你可以使用辅助方法 response：
```

Route::get('home', function () {
    return response($content, $status)
                  ->header('Content-Type', $value);
});

```
注意：有关 Response 方法的完整列表可以参照 [API](http://laravel-china.org/api/master/Illuminate/Http/Response.html) 文档以及[Symfony API](http://laravel-china.org/api/master/Illuminate/Http/Response.html) 文档。
**附加标头至响应**
请记得，**大部份的响应方法是可链式调用的，这让你可以顺畅的创建响应**。
```

return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');

```
**附加 Cookies 至响应**
通过响应实例的`  withCookie ` 辅助方法可以让你轻松的附加 cookies 至响应。：
```

return response($content)->header('Content-Type', $type)
                 ->withCookie('name', 'value');

```
withCookie 方法可以接受额外的可选参数，让你进一步自定义 cookies 的属性：
```

->withCookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

```
**默认情况下，所有 Laravel 产生的 cookies 都会被加密并加上认证标识，因此无法被用户读取及修改。**如果你想在应用程序产生的 cookies 中的某个子集进行加密停用，则可以使用 ` App\Http\Middleware\EncryptCookies ` 中间件的 **$except 属性**：
```

/**
 * 无需被加密的 cookies 名称。
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];

```
使用辅助方法 response 可以轻松的产生其他类型的响应实例。**当你调用辅助方法 response 且不带任何参数时，将会返回 Illuminate\Contracts\Routing\ResponseFactory的实现。**此 Contract 提供了一些有用的方法来产生响应。
###  视图响应 
如果你想要控制响应状态码及标头，同时也想要返回一个视图作为返回的内容时，则可以使用`  view 方法 `：
```

return response()->view('hello', $data)->header('Content-Type', $type);

```
当然，**如果你没有自定义 HTTP 状态码及标头的需求，则可以简单的使用全局的 view 辅助方法。**

###  JSON 响应 
json 方法会自动将标头的 Content-Type 设置为 application/json，并通过 PHP 的 json_encode 函数将指定的数组转换为 JSON：
```

return response()->json(['name' => 'Abigail', 'state' => 'CA']);

```
**如果你想创建一个 JSONP 响应**，则可以使用 json 方法并加上 setCallback：
```

return response()->json(['name' => 'Abigail', 'state' => 'CA'])
                 ->setCallback($request->input('callback'));

```
###  重定向 
重定向响应是类 ` Illuminate\Http\RedirectResponse ` 的实例，并且包含用户要重定向至另一个 URL 所需的标头。有几种方法可以产生 RedirectResponse 的实例。
最简单的方式就是通过全局的`  redirect 辅助 `方法：
```

Route::get('dashboard', function () {
    return redirect('home/dashboard');
});

```
有时你可能希望**将用户重定向至前一个位置**，例如当提交一个无效的表单之后。这时可以使用全局的`  back 辅助 `方法来达成这个目的：
```

Route::post('user/profile', function () {
    // 验证该请求...
    return back()->withInput();
});

```

**重定向至命名路由**
**当你调用 redirect 辅助方法且不带任何参数时，将会返回 Illuminate\Routing\Redirector 的实例**，你可以对该 Redirector 的实例调用任何方法。举个例子，要产生一个 RedirectResponse 到一个命名路由，你可以使用 route 方法：
```

return redirect()->route('login');

```
如果你的路由有参数，则可以将参数放进 route 方法的第二个参数：
```

// For a route with the following URI: profile/{id}
return redirect()->route('profile', [1]);

```
如果你要重定向至路由且路由的参数为 Eloquent 模型的「ID」，则可以直接将模型传入，ID 将会自动被提取：
return redirect()->route('profile', [$user]);

**重定向至控制器行为**
```

return redirect()->action('HomeController@index');

```
当然，如果你的控制器路由需要参数的话，你可以传递它们至 action 方法的第二个参数：
```

return redirect()->action('UserController@profile', [1]);

```

**重定向并加上 Session 闪存数据**
```

Route::post('user/profile', function () {
    // 更新用户的个人数据...
    return redirect('dashboard')->with('status', 'Profile updated!');
});

```
**当然，在用户重定向至新的页面后，你可以获取并显示 session 的闪存数据。**举个例子，使用 Blade 的语法：
```

@if (session('status'))
    <div class="alert alert-success">
        ![](/data/dokuwiki session('status') )
    </div>
@endif

```
      
###  响应宏 
如果你想要自定义可以在很多路由和控制器重复使用的响应，可以使用 ` Illuminate\Contracts\Routing\ResponseFactory ` 实现的` 方法 macro `。
举个例子，来自服务提供者的 boot 方法：
```

<?php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Contracts\Routing\ResponseFactory;

class ResponseMacroServiceProvider extends ServiceProvider
{
    /**
     * 提供注册后运行的服务。
     *
     * @param  ResponseFactory  $factory
     * @return void
     */
    public function boot(ResponseFactory $factory)
    {
        $factory->macro('caps', function ($value) use ($factory) {
            return $factory->make(strtoupper($value));
        });
    }
}

```
macro 函数第一个参数为宏名称，第二个参数为闭包函数。宏的闭包函数会在 ResponseFactory 的实现或者辅助方法 response **调用宏名称的时候被运行**：
```

return response()->caps('foo');

```      