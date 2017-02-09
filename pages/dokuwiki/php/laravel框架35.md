title: laravel框架35 

#  Laravel之EloquentORM关联 
数据表之间经常会互相进行关联。例如，一篇博客文章可能会有多条评论，或是一张订单可能对应一个下单客户。Eloquent 让管理和处理这些关联变得很容易，同时也支持多种类型的关联：
  * 一对一
  * 一对多
  * 多对多
  * 远层一对多
  * 多态关联
  * 多态多对多关联
**你可在 Eloquent 模型类内将 Eloquent 关联定义为函数。**因为关联像 Eloquent 模型一样也可以作为强大的 查询语句构造器，定义关联为函数提供了强而有力的链式调用及查找功能。例如：
```

$user->posts()->where('active', 1)->get();

```
##  一对一# 
一对一关联是很基本的关联。例如一个 User 模型也许会对应一个 Phone。要定义这种关联，我们必须将 phone 方法放置于 User 模型上。phone 方法应该要返回基类 Eloquent 上的 hasOne 方法的结果：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 获取与指定用户互相关联的电话纪录。
     */
    public function phone()
    {
        return $this->hasOne('App\Phone');
    }
}

```
定义好关联之后，我们就可以使用 **Eloquent 的动态属性**来获取关联纪录。**动态属性让你能够访问关联函数**，就像他们是在模型中定义的属性：
```

$phone = User::find(1)->phone;

```
Eloquent 会假设对应关联的**外键名称**是基于模型名称的。在这个例子里，它会自动假设 Phone 模型拥有 user_id 外键。如果你想要重写这个约定，则可以传入第二个参数到 hasOne 方法里。
```

return $this->hasOne('App\Phone', 'foreign_key');

```
此外，Eloquent 的默认外键在上层模型的 id 字段会有个对应值。换句话说，Eloquent 会寻找用户的 id 字段与 Phone 模型的 user_id 字段的值相同的纪录。如果你想让关联使用 id 以外的值，则可以传递第三个参数至 hasOne 方法来指定你自定义的**键**：
```

return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

```
**定义相对的关联#**
现在，让我们在 Phone 模型上定义一个关联，此关联能够让我们访问拥有此电话的 User。我们可以定义与 hasOne 关联相对应的 belongsTo 方法：
```

<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Phone extends Model
{
    /**
     * 获取拥有此电话的用户。
     */
    public function user()
    {
        return $this->belongsTo('App\User');
    }
}

```
在上述例子中，Eloquent 会尝试匹配 Phone 模型的 user_id 至 User 模型的 id。Eloquent 判断的默认外键名称参考自关联模型的方法名称，并会在方法名称后面加上 _id。当然，如果 Phone 模型的外键不是 user_id，则可以传递自定义键名作为 belongsTo 方法的第二个参数：
```

/**
 * 获取拥有此电话的用户。
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key');
}

```
如果你的上层模型不是使用 id 作为主键，或是希望以不同的字段来连接下层模型，则可以传递第三个参数至 belongsTo 方法来指定上层数据表的自定义键：

```

/**
 * 获取拥有此电话的用户。
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key', 'other_key');
}

```

##  一对多# 
一个「一对多」关联使用于定义单个模型拥有任意数量的其它关联模型。例如，一篇博客文章可能会有无限多个评论。就像其它的 Eloquent 关联一样，可以通过放置一个函数到 Eloquent 模型上来定义一对多关联：
```

<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * 获取博客文章的评论。
     */
    public function comments()
    {
        return $this->hasMany('App\Comment');
    }
}

```
```

$comments = App\Post::find(1)->comments;

foreach ($comments as $comment) {
    //
}

```
当然，因为所有的关联也都提供了查询语句构造器的功能，因此你可以对获取到的评论进一步增加条件，通过调用 comments 方法然后在该方法后面链式调用查询条件：
```

$comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

```
就像 hasOne 方法，你也可以通过传递额外的参数至 hasMany 方法来重写外键与本地键：
```

return $this->hasMany('App\Comment', 'foreign_key');
return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

```
**定义相对的关联#**
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 获取拥有此评论的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}

```
一旦关联被定义之后，则可以通过 post「动态属性」来获取 Comment 的 Post 模型：
```

$comment = App\Comment::find(1);
echo $comment->post->title;

```

##  多对多# 
多对多关联要稍微比 hasOne 及 hasMany 关联复杂。如一个用户可能拥有多种身份，而一种身份能同时被多个用户拥有。举例来说，很多用户都拥有「管理者」的身份。
要定义这种关联，**需要使用三个数据表**：users、roles 和 role_user。role_user 表命名是以相关联的两个模型数据表来依照字母顺序命名，并包含了 user_id 和 role_id 字段。
多对多关联通过编写一个在自身 Eloquent 类调用的 belongsToMany 的方法来定义。举个例子，让我们在 User 模型定义 roles 方法：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 属于该用户的身份。
     */
    public function roles()
    {
        return $this->belongsToMany('App\Role');
    }
}

```
一旦关联被定义，则可以使用 roles 动态属性来访问用户的身份：
```

$user = App\User::find(1);

foreach ($user->roles as $role) {
    //
}

```
当然，就如所有其它的关联类型一样，你也可以调用 roles 方法并在该关联之后链式调用查询条件：
```

$roles = App\User::find(1)->roles()->orderBy('name')->get();

```
如前文提到那样，Eloquent 会合并两个关联模型的名称并依照字母顺序命名。当然你也可以随意重写这个约定。可通过传递第二个参数至 belongsToMany 方法来实现：
```

return $this->belongsToMany('App\Role', 'user_roles');

```
除了自定义合并数据表的名称，你也可以通过传递额外参数至 belongsToMany 方法来自定义数据表里的键的字段名称。**第三个参数是你定义在关联中的模型外键名称，而第四个参数则是你要合并的模型外键名称：**
```

return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'role_id');

```
**定义相对的关联#**
要定义相对于多对多的关联，只需简单的放置另一个名为 belongsToMany 的方法到你关联的模型上。让我们接着以用户身份为例，在 Role 模型中定义 users 方法：
```

<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 属于该身份的用户。
     */
    public function users()
    {
        return $this->belongsToMany('App\User');
    }
}

```
如你所见，此定义除了简单的参考 App\User 模型外，与 User 的对应完全相同。因为我们重复使用了 belongsToMany 方法，当定义相对于多对多的关联时，所有常用的自定义数据表与键的选项都是可用的。
**获取中间表字段#**
要操作多对多关联需要一个中间数据表。Eloquent 提供了一些有用的方法来和这张表进行交互。例如，假设 User 对象关联到很多的 Role 对象。访问这些关联对象时，我们可以在模型中使用 pivot 属性来访问中间数据表的数据：
```

$user = App\User::find(1);
foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}

```
注意我们取出的每个 Role 模型对象，都会被自动赋予 pivot 属性。此属性代表中间表的模型，它可以像其它的 Eloquent 模型一样被使用。
默认情况下，pivot 对象只提供模型的键。如果你的 pivot 数据表包含了其它的属性，则可以在定义关联方法时指定那些字段：
```

return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

```
如果你想要中间表自动维护 created_at 和 updated_at 时间戳，可在定义关联方法时加上 withTimestamps 方法：
```

return $this->belongsToMany('App\Role')->withTimestamps();

```

##  预加载# 
**当通过属性访问 Eloquent 关联时，该关联数据会被「延迟加载」**。意味着该关联数据只有在你使用属性访问它时才会被加载。
不过，Eloquent 可以在你查找上层模型时「预加载」关联数据。**预加载避免了 N + 1 查找的问题。**要说明 N + 1 查找的问题，可试想一个关联到 Author 的 Book 模型，如下所示：
```

<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * 获取编写该书的作者。
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}

```
现在，让我们来获取所有书籍及其作者的数据：
```

$books = App\Book::all();
foreach ($books as $book) {
    echo $book->author->name;
}

```
上方的循环会运行一次查找并取回所有数据表上的书籍，接着每本书会运行一次查找作者的操作。
因此，若存在着 25 本书，则循环就会执行 26 次查找：1 次是查找所有书籍，其它 25 次则是在查找每本书的作者。
很幸运地，我们可以使用预加载来将查找的操作减少至 2 次。可在查找时**使用 with 方法来指定想要预加载的关联数据**：
```

$books = App\Book::with('author')->get();
foreach ($books as $book) {
    echo $book->author->name;
}

```
对于该操作则只会运行两次查找：
```

select * from books
select * from authors where id in (1, 2, 3, 4, 5, ...)

```
**预加载多种关联#**
```

$books = App\Book::with('author', 'publisher')->get();

```
**嵌套预加载#**
```

$books = App\Book::with('author.contacts')->get();

```
**预加载条件限制#**
有时你可能想要预加载关联，并且指定预加载查询的额外条件。如下所示：
```

$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();

```

##  写入关联模型# 
**Save 方法#**
```

$comment = new App\Comment(['message' => 'A new comment.']);
$post = App\Post::find(1);
$post->comments()->save($comment);

```
**注意我们并没有使用动态属性来访问 comments 关联。相反地，我们调用了 comments 方法来获取关联的实例。save 方法会自动在新的 Comment 模型中增加正确的 post_id 值。**
如果你需要保存多个关联模型，则可以使用 saveMany 方法：
```

$post = App\Post::find(1);
$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);

```
**Save 与多对多关联#**
当使用多对多关联时，save 方法允许传入一个额外的中间表属性数组来作为第二个参数：
```

App\User::find(1)->roles()->save($role, ['expires' => $expires]);

```
**Create 方法#**
除了 save 与 saveMany 方法外，你也可以使用 create 方法，该方法允许传入属性的数组来建立模型并写入数据库。**save 与 create 的不同之处在于，save 允许传入一个完整的 Eloquent 模型实例，但 create 只允许传入原始的 PHP 数组：**
```

$post = App\Post::find(1);
$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);

```

**更新「从属」关联#**
当更新一个 belongsTo 关联时，可以使用 associate 方法。此方法会将外键设置到下层模型：
```

$account = App\Account::find(10);
$user->account()->associate($account);
$user->save();

```
当删除一个 belongsTo 关联时，你可以使用 dissociate 方法。此方法会重置关联到下层模型的外键：
```

$user->account()->dissociate();
$user->save();

```
**附加与卸除#**
当使用多对多关联时，Eloquent 提供了一些额外的辅助方法让操作关联模型更加方便。例如，让我们假设一个用户可以拥有多个身份，且每个身份都可以被多个用户拥有。要附加一个规则至一个用户，并连接模型以及将记录写入至中间表，则可以使用 attach 方法：
```

$user = App\User::find(1);
$user->roles()->attach($roleId);

```
当附加一个关联至模型时，你也可以传递一个需被写入至中间表的额外数据数组：
```

$user->roles()->attach($roleId, ['expires' => $expires]);

```
**有时我们也需要来移除用户的身份。要移除一个多对多的纪录，可使用 detach 方法。detach 方法会从中间表中移除正确的纪录**；当然，$user 和 $role 这两个模型依然会存在于数据库中：
```

// 从用户身上移除单个身份...
$user->roles()->detach($roleId);
// 从用户身上移除所有身份...
$user->roles()->detach();

```
为了方便，attach 与 detach 都允许传入 ID 数组：
```

$user = App\User::find(1);
$user->roles()->detach([1, 2, 3]);
$user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

```
**为了方便的同步#**
你也可以使用 sync 方法来构建多对多关联。sync 允许传入放置于中间表的 ID 数组。**任何不在指定数组中的 ID 都将会从中间表中删除。所以，在此操作结束后，中间表中只存在数组中的 ID：**
```

$user->roles()->sync([1, 2, 3]);

```
你也可以传递中间表上该 ID 额外的值：
```

$user->roles()->sync([1 => ['expires' => true], 2, 3]);

```
**连动上层时间戳#**
当一个模型 belongsTo 或 belongsToMany 另一个模型时，像是一个 Comment 属于一个 Post。这对于下层模型被更新时，要更新上层的时间戳相当有帮助。
举例来说，当一个 Commnet 模型被更新时，你可能想要「连动」更新 Post 所属的 updated_at 时间戳。Eloquent 使得此事相当容易。只要在关联的下层模型中增加一个包含名称的`  touches 属性 `即可：
```

<?php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 所有的关联将会被连动。
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * 获取拥有此评论的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}

```
现在，当你更新一个 Comment 时，它所属的 Post 拥有的 updated_at 字段也会被同时更新：
```

$comment = App\Comment::find(1);
$comment->text = '编辑此评论！';
$comment->save();

```