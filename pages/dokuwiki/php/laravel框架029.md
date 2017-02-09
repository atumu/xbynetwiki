title: laravel框架029 

#  Laravel之表单验证 
Laravel 提供了多种不同的处理方法来对应用程序传入的数据进行验证。默认情况下，**Laravel 的基底控制器类使用了 ` ValidatesRequests ` trait**，其提供了一种便利的方法来使用各种强大的验证规则验证传入的 HTTP 请求。
##  验证快速上手# 
先让我们来看看一个完整的**表单验证示例以及返回错误消息**给用户。
**定义路由#**
首先，我们假设在 ` app/Http/routes.php ` 文件中定义了以下路由：
```

// 显示一个创建博客文章的表单...
Route::get('post/create', 'PostController@create');
// 保存一个新的博客文章...
Route::post('post', 'PostController@store');

```
**创建控制器#**
接下来，让我们来看下操作这些路由的控制器。

**编写验证逻辑#**
检查应用程序的基底控制器 (App\Http\Controllers\Controller) 类你会看到这个类使用了 ValidatesRequests trait。这个 trait 在你所有的控制器里提供了方便的 validate 验证方法。
` validate 方法 `会接收 HTTP 传入的请求以及验证的规则。如果验证通过，你的代码就可以正常的运行。**若验证失败，则会抛出异常错误消息并自动将其返回给用户。在一般的 HTTP 请求下，都会生成一个重定向响应，对于 AJAX 请求则会发送 JSON 响应。**
让我们接着回到 store 方法来深入理解 validate 方法：
```

public function store(Request $request)
{
    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // 博客文章成功发表，将其保存到数据库...
}

```
**如果验证失败，将会自动生成一个对应的响应。**如果验证通过，那我们的控制器将会继续正常运行。

**对于嵌套属性的提醒#**
如果你的 HTTP 请求中包含了「嵌套」参数，则可以在验证规则中使用「点」语法来指定他们：
```

$this->validate($request, [
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);

```
###  显示验证错误# 
如果本次请求的参数未通过我们指定的验证规则呢？正如前面所提到的，**Laravel 会自动把用户重定向到先前的位置。另外，所有的验证错误会被自动闪存至 session。**
请注意我们并不需要在 GET 路由中明确的将错误消息绑定到视图上。这是因为** Laravel 会自动检查 session 内的错误数据，如果错误存在的话，它会自动将这些错误消息绑定到视图上**。因此需要的注意一点是** $errors 变量在每次请求的所有视图中都可以被使用**，你可以很方便的假设 $errors 变量已被定义且进行安全地使用。**$errors 变量是 Illuminate\Support\MessageBag 的实例**。
所以，在我们的例子中，当验证失败时，用户将被重定向到我们的控制器 create 方法，让我们在视图中显示错误的消息：
```

<!-- /resources/views/post/create.blade.php -->
<h1>创建文章</h1>

@if (count($errors) > 0)
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>![](/data/dokuwiki $error )</li>
            @endforeach
        </ul>
    </div>
@endif

```
**自定义闪存的错误消息格式#**
当验证失败时，如果你想要在闪存上自定义验证的错误格式，则需**在控制器中重写 formatValidationErrors**。别忘了将 ` Illuminate\Contracts\Validation\Validator ` 类引入到文件上方：
```

<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Bus\DispatchesJobs;
use Illuminate\Contracts\Validation\Validator;
use Illuminate\Routing\Controller as BaseController;
use Illuminate\Foundation\Validation\ValidatesRequests;

abstract class Controller extends BaseController
{
    use DispatchesJobs, ValidatesRequests;

    /**
     * {@inheritdoc}
     */
    protected function formatValidationErrors(Validator $validator)
    {
        return $validator->errors()->all();
    }
}

```
###  AJAX 请求和验证# 
在这个例子中，我们使用一种传统的方式来将数据发送到应用程序上。**当我们在 AJAX 的请求中使用 validate 方法时，Laravel 并不会生成一个重定向响应，而是会生成一个包含所有错误验证的 JSON 响应。这个 JSON 响应会发送一个 422 HTTP 状态码。**

##  表单请求验证# 
**表单请求是一个自定义的请求类，里面包含着验证逻辑。**要创建一个表单请求类，可使用 Artisan 命令行命令 make:request ：
```

php artisan make:request StoreBlogPostRequest

```
新生成的类文件会被放在 ` app/Http/Requests ` 目录下。让我们将一些验证规则加入到 ` rules 方法 `中：
```

/**
 * 获取适用于请求的验证规则。
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}

```
怎样才能较好的运行验证规则呢？你所需要做的就是**在控制器方法中利用类型提示传入请求**。**传入的请求会在控制器方法被调用前进行验证，意思就是说你不会因为验证逻辑而把控制器弄得一团糟**：
```

/**
 * 保存传入的博客文章。
 *
 * @param  StoreBlogPostRequest  $request
 * @return Response
 */
public function store(StoreBlogPostRequest $request)
{
    // 传入的请求是有效的...
}

```
###  授权表单请求# 
表单的请求类内包含了 ` authorize 方法 `。在这个方法中，**你可以确认用户是否真的通过了授权**，以便更新指定数据。比方说，有一个用户想试图去更新一篇文章的评论，你能保证他确实是这篇评论的拥有者吗？具体代码如下：
```

/**
 * 判断用户是否有权限做出此请求。
 *
 * @return bool
 */
public function authorize()
{
    $commentId = $this->route('comment');

    return Comment::where('id', $commentId)
                  ->where('user_id', Auth::id())->exists();
}

```
请注意，在上面例子中**调用 route 方法。该方法可以帮助你获取路由被调用时传入的 URI 参数，如示例中的 {comment} 参数**：
Route::post('comment/{comment}');
**如果 authorize 方法返回 false，则会自动返回一个 HTTP 响应，其中包含 403 状态码，而你的控制器方法也将不会被运行。**
##  自定义闪存的错误消息格式# 
如果你想要自定义验证失败时闪存到 session 的验证错误格式，可在你的基底请求 **(App\Http\Requests\Request) 中重写 formatErrors**。别忘了文件上方引入 Illuminate\Contracts\Validation\Validator 类：
```

protected function formatErrors(Validator $validator)
{
    return $validator->errors()->all();
}

```
###  自定义错误消息# 
你可以通过**重写表单请求的 messages 方法来自定义错误消息**。此方法必须返回一个数组，其中含有成对的属性或规则以及对应的错误消息：
```

/**
 * 获取已定义验证规则的错误消息。
 * @return array
 */
public function messages()
{
    return [
        'title.required' => '标题是必填的',
        'body.required'  => '消息是必填的',
    ];
}

```
###  在语言包中自定义指定消息# 
在许多情况下，你可能希望在语言包中被指定的特定属性自定义消息不被直接传到 Validator 上。**因此你可以把消息加入到 resources/lang/xx/validation.php 语言包中的 custom 数组**。
```

'custom' => [
    'email' => [
        'required' => '我们需要知道你的 e-mail 地址！',
    ],
],

```
##  可用的验证规则# 
![](/data/dokuwiki/php/pasted/20160422-145827.png)

##  自定义验证规则 
http://laravel-china.org/docs/5.1/validation