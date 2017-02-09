title: laravel框架36 

#  Laravel之EloquentORM集合操作 
http://laravel-china.org/docs/5.1/eloquent-collections
默认情况下 Eloquent 返回的都是一个 ` Illuminate\Database\Eloquent\Collection ` 对象的实例，包含通过 get 方法或是访问一个关联来获取到的结果。Eloquent 集合对象继承了 Laravel 集合基类，因此它自然也继承了许多可用于与 Eloquent 模型交互的方法。
当然，所有集合都可以作为迭代器，来让你像遍历一个 PHP 数组一样来遍历一个集合：
```

$users = App\User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}

```
然而，集合比数组更强大的地方是其使用了各种 map / reduce 的直观操作。例如，我们移除所有未激活的用户模型和收集其余各个用户的名字：
```

$users = App\User::where('active', 1)->get();
$names = $users->reject(function ($user) {
    return $user->active === false;
})
->map(function ($user) {
    return $user->name;
});

```
