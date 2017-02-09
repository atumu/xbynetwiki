title: laravel框架33 

#  Laravel数据库迁移 
**迁移就像是数据库中的版本控制**，它让团队能够轻松的修改跟共享应用程序的数据库结构。迁移通常会搭配上 Laravel 的结构构造器来让你轻松的构建数据库结构。
Laravel 的 ` Schema facade ` 对数据表的创建和操作提供了相应支持。它在所有 Laravel 支持的数据库系统中共用了一套相同的 API。
##  生成迁移# 
你可以使用 make:migration Artisan 命令 来创建迁移：
```

php artisan make:migration create_users_table

```
新的迁移文件将会被放置在 ` database/migrations ` 目录中。每个迁移文件的名称都包含了一个时间戳，以便让 Laravel 确认迁移的顺序。
- -table 和 - -create 选项可用来指定数据表的名称，或是该迁移被执行时会创建的新数据表。这些选项需在预生成迁移文件时填入指定的数据表：
```

php artisan make:migration add_votes_to_users_table --table=users
php artisan make:migration create_users_table --create=users

```
如果你想为生成的迁移指定一个自定义输出路径，则可以在运行 make:migration 命令时使用 - -path 选项。提供的路径必须是相对于应用程序的基本路径。
###  迁移结构# 
**一个迁移类会包含两个方法：up 和 down。**up 方法可为数据库增加新的数据表、字段、或索引，而 down 方法则可简单的反向运行 up 方法的操作。
让我们来创建一张 flights 数据表作为示例：
```

<?php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateFlightsTable extends Migration
{
    /**
     * 运行迁移。
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * 还原迁移。
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}

```

##  运行迁移# 
要运行应用程序中的所有未执行迁移，可以使用 migrate Artisan 命令。
```

php artisan migrate

```
如果在你运行时出现「class not found」的错误，请试着在运行 ` composer dump-autoload ` 命令后再次运行迁移命令。

**在线上环境强制运行迁移#**
**一些迁移的操作是具有破坏性的，意思是它们可能会导致数据丢失。**为了保护线上环境的数据库，系统会在这些命令被运行之前显示确认提示。若要忽略此提示并强制运行命令，则可以使用 - -force 标记：
```

php artisan migrate --force

```
**还原迁移#**
若要将迁移还原至上一个「操作」，则可以使用 rollback 命令。请注意，此还原是对上一次执行的「批量」迁移进行还原，其中可能包括多个迁移文件：
```

php artisan migrate:rollback

```
migrate:reset 命令会还原应用程序中的所有迁移：
```

php artisan migrate:reset

```
单个命令还原或运行迁移#
migrate:refresh 命令首先会还原数据库的所有迁移，接着再运行 migrate 命令。此命令能有效的重建整个数据库：
```

php artisan migrate:refresh
 php artisan migrate:refresh --seed

```
##  编写迁移# 
**创建数据表#**
要创建一张新的数据表，则可以使用 ` Schema facade 的 create方法 `。create 方法接收两个参数。第一个参数为数据表的名称，第二个参数为一个闭包，此闭包会接收一个用于定义新数据表的 ` Blueprint 对象： `
```

Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});

```
**检查数据表或字段是否存在#**
你可以使用 hasTable 和 hasColumn 方法简单的检查数据表或字段是否存在：
```

if (Schema::hasTable('users')) {
    //
}
if (Schema::hasColumn('users', 'email')) {
    //
}

```
**连接与存储引擎#**
如果你想要在一个非默认的数据库连接中进行结构操作，则可以使用 connection 方法：
```

Schema::connection('foo')->create('users', function ($table) {
    $table->increments('id');
});

```
**若要设置数据表的存储引擎**，只需在结构构造器上设置 engine 属性即可：
```

Schema::create('users', function ($table) {
    $table->engine = 'InnoDB';
    $table->increments('id');
});


```
**重命名与删除数据表#**
若要重命名一张已存在的数据表，则可以使用 rename 方法：
```

Schema::rename($from, $to);

```
要删除已存在的数据表，可使用 drop 或 dropIfExists 方法：
```

Schema::drop('users');
Schema::dropIfExists('users');

```

**创建字段#**
**若要更新一张已存在的数据表，我们会使用 Schema facade 的 table 方法。**如同 create 方法，table 方法会接收两个参数：一个是数据表的名称，另一个则是接收 Blueprint 实例的闭包。我们可以使用它来为数据表新增字段：
```

Schema::table('users', function ($table) {
    $table->string('email');
});

```
###  可用的字段类型# 
结构构造器包含了许多字段类型，供你构建数据表时使用：
![](/data/dokuwiki/php/pasted/20160421-140856.png)
![](/data/dokuwiki/php/pasted/20160421-140918.png)
![](/data/dokuwiki/php/pasted/20160421-140930.png)
###  字段修饰# 
除了上述的字段类型列表，还有一些其它的字段「修饰」，你可以将它增加到字段中。例如，若要让字段「nullable」，那么你可以使用 nullable 方法：
```

Schema::table('users', function ($table) {
    $table->string('email')->nullable();
});

```
以下列表为字段的可用修饰。此列表不包括索引修饰：
![](/data/dokuwiki/php/pasted/20160421-141010.png)

###  修改字段 
` 在修改字段之前，请务必在你的 composer.json 中增加 doctrine/dbal 依赖，并在你的命令行中运行 composer update 命令来安装该函数库。 `
**Doctrine DBAL 函数库**被用于判断当前字段的状态以及创建调整指定字段的 SQL 查询。
**更新字段属性#**
change 方法让你可以修改一个已存在的字段类型，或修改字段属性。例如，你或许想增加字符串字段的长度。要看看 change 方法的具体作用，让我们来将 name 字段的长度从 25 增加到 50：
```

Schema::table('users', function ($table) {
    $table->string('name', 50)->change();
});

```
我们也能将字段修改为 nullable：
```

Schema::table('users', function ($table) {
    $table->string('name', 50)->nullable()->change();
});

```

**重命名字段#**
```

Schema::table('users', function ($table) {
    $table->renameColumn('from', 'to');
});

```
注意：数据表的 enum 字段暂时不支持修改字段名称。

**移除字段#**
要移除字段，可使用结构构造器的 dropColumn 方法：
```

Schema::table('users', function ($table) {
    $table->dropColumn('votes');
});
Schema::table('users', function ($table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});

```
` 注意：SQLite 数据库并不支持在单个迁移中移除或修改多个字段。 `

###  创建索引# 
结构构造器支持多种类型的索引。首先，让我们先来看看一个示例，其指定了字段的值必须是唯一的。你可以简单的在字段定义之后链式调用 unique 方法来创建索引：
```

$table->string('email')->unique();

```
此外，你也可以在定义完字段之后创建索引。例如：
```

$table->unique('email');

```
你也可以传递一个字段的数组至索引方法来创建复合索引：
```

$table->index(['account_id', 'created_at']);

```
####  可用的索引类型# 
![](/data/dokuwiki/php/pasted/20160421-141434.png)
**移除索引#**
若要移除索引，则必须指定索引的名称。Laravel 默认会自动给索引分配合理的名称。其将数据表名称，索引的字段名称，及索引类型简单地连接在了一起。举例如下：
![](/data/dokuwiki/php/pasted/20160421-141521.png)

###  外键约束# 
Laravel 也为创建外键约束提供了支持，用于在数据库层中的强制引用完整性。例如，让我们定义一个有 user_id 字段的 posts 数据表，user_id 引用了 users 数据表的 id 字段：
```

Schema::table('posts', function ($table) {
    $table->integer('user_id')->unsigned();
    $table->foreign('user_id')->references('id')->on('users');
});

```
你也可以指定约束的「on delete」及「on update」：
```

$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');

```
**要移除外键，你可以使用 dropForeign 方法。**外键约束与索引采用相同的命名方式。所以，我们可以将数据表名称和约束字段连接起来，接着在该名称后面加上「_foreign」后缀：
```

$table->dropForeign('posts_user_id_foreign');

```
你也通过传递一个自动使用传统约束名称的数组值来移除外键：
```

$table->dropForeign(['user_id']);

```