title: laravel框架32 

#  Laravel之数据库模拟测试数据的填充 
Laravel 可以简单的使用 seed 类来给数据库填充测试数据。所有的 seed 类都放在`  database/seeders ` 目录下。 Laravle 默认为你定义了一个 ` DatabaseSeeder ` 类。你可以在这个类中**使用 call 方法来运行其它的 seed 类**，以借此控制数据填充的顺序。
##  编写数据填充# 
你可以通过 make:seeder Artisan 命令 来生成一个 Seeder。所有通过框架生成的 Seeder 都将被放置在 database/seeders 路径：
```

php artisan make:seeder UsersTableSeeder

```
**在 seeder 类里只有一个默认方法：run。当运行 db:seed Artisan 命令 时就会调用此方法。**你可以在 run 方法中给数据库添加任何数据。你可使用 查询语句构造器 或 Eloquent 模型工厂 来手动添加数据。
如下所示，我们将修改 Laravel 预先生成好的 DatabaseSeeder 类来给 run 方法添加一段可在数据库添加数据的语法：
```

<?php
use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder
{
    /**
     * 运行数据库填充。
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => str_random(10),
            'email' => str_random(10).'@gmail.com',
            'password' => bcrypt('secret'),
        ]);
    }
}

```
###  使用模型工厂# 
**手动为每一个 seed 模型一一指定属性是很麻烦的一件事。**作为替代方案，你**可以使用 模型工厂 来帮助你更便捷的生成大量数据库数据。**一旦工厂被定义，就能使用 factory 这个辅助方法函数来添加数据到数据库。
让我们来创建 50 个用户并为每个用户创建一个关联：
```

/**
 * 运行数据库填充。
 *
 * @return void
 */
public function run()
{
    factory(App\User::class, 50)->create()->each(function($u) {
        $u->posts()->save(factory(App\Post::class)->make());
    });
}

```
**模型工厂#**
**测试时，常常需要在运行测试之前写入一些数据到数据库中**。创建测试数据时，除了手动的来设置每个字段的值，还可以使用 Eloquent 模型的「工厂」来设置每个属性的默认值。在开始之前，你可以先查看下应用程序的 ` database/factories/ModelFactory.php ` 文件。此文件包含一个现成的工厂定义：
```

$factory->define(App\User::class, function (Faker\Generator $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->email,
        'password' => bcrypt(str_random(10)),
        'remember_token' => str_random(10),
    ];
});

```
闭包内为工厂的定义，你可以返回模型中所有属性的默认测试值。在该闭包内会接收到` [Faker PHP 函数库](https://github.com/fzaninotto/Faker) `的实例，它可以让你很方便的生成各种随机数据以进行测试。
当然，你也可以随意将自己额外的工厂增加至 ModelFactory.php 文件。
**多个工厂类型#**
有时你可能希望针对同一个 Eloquent 模型类来创建多个工厂。例如，除了一般用户的工厂之外，还有「管理员」工厂。你可以使用 defineAs 方法来定义这个工厂：
```

$factory->defineAs(App\User::class, 'admin', function ($faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->email,
        'password' => str_random(10),
        'remember_token' => str_random(10),
        'admin' => true,
    ];
});

```
除了从一般用户工厂复制所有基本属性，你也可以使用 raw 方法来获取所有基本属性。一旦你获取到这些属性，就可以轻松的**为其增加任何额外值**：
```

$factory->defineAs(App\User::class, 'admin', function ($faker) use ($factory) {
    $user = $factory->raw(App\User::class);

    return array_merge($user, ['admin' => true]);
});

```
**在测试中使用工厂#**
在工厂定义后，就可以在测试或是数据库的填充文件中，` 通过全局的 factory 函数来生成模型实例 `。接着让我们先来看看几个创建模型的例子。首先我们会使用 make 方法创建模型，但不将它们保存至数据库：
```

public function testDatabase()
{
    $user = factory(App\User::class)->make();

    // 在测试中使用模型...
}

```
**如果你想重写模型中的某些默认值**：
```

$user = factory(App\User::class)->make([
    'name' => 'Abigail',
   ]);

```
你也可以创建一个含有多个模型的集合，或创建一个指定类型的模型：
```

// 创建三个 App\User 实例...
$users = factory(App\User::class, 3)->make();
// 创建一个 App\User「管理员」实例...
$user = factory(App\User::class, 'admin')->make();
// 创建三个 App\User「管理员」实例...
$users = factory(App\User::class, 'admin', 3)->make();

```
**保存多个模型，增加关联至模型#**
你甚至可以保存多个模型到数据库上。在本例中，我们还会增加关联至我们所创建的模型。当使用 create 方法创建多个模型时，它会返回一个 Eloquent 集合实例，让你能够使用集合所提供的便利函数，像是 each：
```

$users = factory(App\User::class, 3)
           ->create()
           ->each(function($u) {
                $u->posts()->save(factory(App\Post::class)->make());
            });

```
###  调用其它的 Seeders# 
在 DatabaseSeeder 类中，你可以**使用 call 方法来运行其它的 seed 类**。
为避免发生单个 seeder 类变得太大的情况，可使用 call方法来将数据填充拆分成多个文件。只需简单传递你想要运行的 seeder 类名称即可：
```

public function run()
{
    Model::unguard();
    $this->call(UsersTableSeeder::class);
    $this->call(PostsTableSeeder::class);
    $this->call(CommentsTableSeeder::class);
    Model::reguard();
}

```

##  运行数据填充# 
在默认的情况下，db:seed 命令将运行 DatabaseSeeder 类，并通过它来调用其它的 seed 类。但是，你也可以使用 --class 选项来单独运行一个特定的 seeder 类：
```

php artisan db:seed
php artisan db:seed --class=UserTableSeeder

```
你也可以使用 migrate:refresh 命令来对数据库进行数据填充，它会回滚并重新运行所有迁移。这在对数据库进行重构时非常有用：
```

php artisan migrate:refresh --seed

```