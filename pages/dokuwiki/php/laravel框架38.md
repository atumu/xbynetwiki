title: laravel框架38 

#  Laravel之EloquentORM之序列化与json 
当你在创建 JSON API 的时候，经常会需要将模型和关联转换成数组或 JSON。Eloquent 提供了一些便捷的方法来让我们可以完成这些转换，以及控制哪些属性需要被包括在序列化中。
**将模型转换成一个数组#**
如果要将模型还有其加载的关联转换成一个数组，则可以使用`  toArray 方法 `。**这个方法是递归的**，因此，所有属性和关联（包含关联中的关联）都会被转换成数组：
```

$user = App\User::with('roles')->first();
return $user->toArray();

```
你也可以将 集合 转换成数组：
```

$users = App\User::all();
return $users->toArray();

```
**将模型转换成 JSON#**
如果要将模型转换成 JSON，则可以使用 ` toJson 方法 `。如同 toArray 方法一样，**toJson 方法也是递归的。**因此，所有的属性以及关联都会被转换成 JSON：
```

$user = App\User::find(1);
return $user->toJson();

```
或者，你也可以强制把一个模型或集合转型成一个字符串，它将会自动调用 toJson 方法：
$user = App\User::find(1);
return (string) $user;
` 当模型或集合被转型成字符串时，模型或集合便会被转换成 JSON 格式，因此你可以直接从应用程序的路由或者控制器中返回 Eloquent 对象 `：
```

Route::get('users', function () {
    return App\User::all();
});

```

##  隐藏来自 JSON 的属性# 
有时候你可能会想要限制包含在模型数组或 JSON 表示中的属性，**比如说密码**。则可以通过在模型中增加`  $hidden 属性 `定义来实现：
```

<?php
namespace App;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 在数组中隐藏的属性。
     *
     * @var array
     */
    protected $hidden = ['password'];
}

```
另外，你也可以使用`  visible 属性 `来定义应该包含在你的模型数组和 JSON 表示中的**属性白名单**：
```

/**
     * 在数组中可见的属性。
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];

```
##  添加一些值到 JSON# 
有时候，你可能需要**添加在数据库中没有对应字段的数组属性**。首先你需要为这个值定义一个 访问器：
```

<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 为用户获取管理者的标记。
     *
     * @return bool
     */
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }
}

```
一旦访问器创建成功，就可以将属性名称添加到模型的 ` appends 属性 `：
```

    /**
     * 访问器被附加到模型数组的形式。
     *
     * @var array
     */
    protected $appends = ['is_admin'];

```
一旦属性被添加到 appends 清单，便会将模型中的数组和 JSON 这两种形式都包含进去。在 appends 数组中的属性也遵循模型中 visible 和 hidden 设置。
