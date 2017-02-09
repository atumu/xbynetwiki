title: laravel框架15 

#  Laravel之Artisan命令行 
Artisan 是 Laravel 的命令行接口的名称，它提供了许多实用的命令来帮助你开发 Laravel 应用，它是由强大的 Symfony Console 组件所驱动的。你可以使用 list 命令来列出所有可用的 Artisan 命令：
```

php artisan list

```
每个命令也包含了「帮助」界面，它会显示并概述命令可以使用的参数及选项。只要在命令前面加上 help 即可显示帮助界面：
```

php artisan help migrate

```
##  编写自定义命令 
很有用，需要时强烈建议参考。
略。。。请参考：http://laravel-china.org/docs/5.1/artisan
##  使用代码调用命令 
有时候你想在命令行接口外运行 Artisan 命令。例如，你希望在路由或控制器触发 Artisan 命令。**只需利用 Artisan facade 的 call 方法即可做到**。call 方法的第一个参数为命令的名称，第二个参数为数组型态的命令输入。退出码将会被返回：
```

Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1,
        '--queue' => 'default'
    ]);

    //
});

```
在 Artisan facade 使用 **queue 方法，可以将一堆 Artisan 命令放进队列**，好让它们能在后台被你的队列服务器 运行：
```

Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1,
        '--queue' => 'default'
    ]);

    //
});

```
