title: laravel框架1 

#  Laravel入门与安装 
官网:https://laravel.com/ (已被墙)
中文:http://laravel-china.org/
APIDoc:https://laravel.com/api/5.1/index.html
国外资料:http://www.tutorialspoint.com/laravel/index.htm
学习版本:Laravel 5.1 LTS.要求PHP 5.6以上

使用PHP & Laravel 创建的项目列表:
  * [phphub](https://github.com/summerblue/phphub)

Laravel 让我们书写优雅的代码，为 Web 艺术家创造的 PHP 框架。全栈框架，旨在提高开发效率

Laravel 5.1 LTS运行环境要求
Laravel 框架会有一些系统上的要求。当然，这些要求在 [Laravel Homestead](http://laravel-china.org/docs/5.1/homestead) 虚拟机上都已经完全配置好了：
  * PHP >= 5.5.9
  * OpenSSL PHP Extension
  * PDO PHP Extension
  * Mbstring PHP Extension
开发推荐使用 Homestead，线上环境的部署可参考[ Homestead 的环境部署脚本](https://github.com/laravel/settler/blob/master/scripts/provision.sh) 来实现开发环境和线上环境的统一。
##  安装 Laravel 
**Laravel 使用 Composer 来管理代码依赖。**所以，在使用 Laravel 之前，请先确认你的电脑上安装了 Composer。
首先，使用 Composer 下载 Laravel 安装包：
```

composer global require "laravel/installer"

```
请确定你已将 ~/.composer/vendor/bin 路径加到 PATH，只有这样系统才能找到 laravel 的执行文件。
一旦安装完成，就可以使用`  laravel new 命令 `在指定的目录创建一个新的 Laravel 项目，例如：laravel new blog 将会在当前目录下创建一个叫 blog 的目录，此目录里面存放着新安装的 Laravel 和代码依赖。这个方法的安装速度比通过 Composer 安装要快上许多：
```

laravel new blog

```
但是我通过实验报错  
[Symfony\Component\Process\Exception\RuntimeException]
TTY mode is not supported on Windows platform.

所以改用composer命令创建:
（通过`  Composer Create-Project ` 除此之外，你也可以通过 Composer 在命令行运行 create-project 命令来安装 Laravel：
composer create-project laravel/laravel --prefer-dist blog
Summer: 使用以下命令安装 Laravel 5.1 LTS
```

composer create-project laravel/laravel your-project-name --prefer-dist "5.1.*"

```

##  配置信息 
所有 Laravel 框架的配置文件都放置在 ` config 目录 `下。每个选项都有说明，因此你可以轻松地浏览这些文档，并且熟悉这些选项配置。
  * 目录权限：安装 Laravel 之后，你必须设置一些权限。` storage 和 bootstrap/cache 目录必须让服务器有写入权限。 `如果你使用 Homestead 虚拟机，那么这些权限应该已经被设置完成。
  * 应用程序密钥：在你安装完 Laravel 后，**首先需要做的事情是设置一个随机字符串到应用程序密钥**。(**假设你是通过 Composer 或是 Laravel 安装工具安装 Laravel，那么这个密钥已经通过 key:generate 命令帮你设置完成。**) 通常这个密钥应该有 32 字符长度。**这个密钥可以被设置在 .env 环境文件中。**如果你还没将 .env.example 文件重命名为 .env，那么你现在应该去设置下。如果应用程序密钥没有被设置的话，你的用户 Sessions 和其他的加密数据都是不安全的！
  * 其他设置:Laravel 几乎不需做任何其它设置就可以马上使用，但是建议你` 先浏览 config/app.php 文件和对应的文档 `，这里面包含着一些选项，如时区和语言环境。
你也可以设置 Laravel 的几个附加组件，像是：
  * 缓存
  * 数据库
  * Session
一旦 Laravel 安装完成，你应该立即设置本机环境。
###  生成app key: 

```

php artisan key:generate

```
##  优雅链接-路由实现机制 
**Apache**
Laravel 框架通过 public/` .htaccess ` 文件来让网址中不需要 index.php。如果你的服务器是使用 Apache，请确认是否有开启 ` mod_rewrite 模块 `。
假设 Laravel 附带的 .htaccess 文件在 Apache 无法作用的话，请尝试下方的做法：
```

Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]

```

**Nginx**
若使用 Nginx，可以在你的网站设置中增加下面的设置，以开启「优雅链接」：
```

server {
    listen 80 ;
    server_name  localhost;
    # 设定网站根目录，到public目录层次
    root /var/www/laravel/public;
    # 网站默认首页
    index index.php index.html index.htm;
    # 修改为 Laravel 转发规则，否则PHP无法获取$_GET信息，提示404错误
    location / {
        try_files $uri $uri/ /index.php$is_args$query_string;        
    }

    # PHP 支持
    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

```
当然，如果你使用了 Homestead 的话，它将会自动的帮你设置好优雅链接。

##  环境配置 
**应用程序常常需要根据不同的运行环境设置不同的值。**例如，你会希望在你的本机开发环境上有与正式环境不同的缓存驱动。只要通过配置文件，就可以轻松完成。
Laravel 使用 Vance Lucas 的 DotEnv PHP 函数库来实现项目内环境变量的控制，在安装好的全新 Laravel 应用程序里，在根目录下会包含一个 .env.example 文件。如果你通过 Composer 安装 Laravel，这个文件将自动被更名为`  .env `，否则你只能手动更改文件名。

当你的应用程序收到请求时，这个文件所有的变量都会被加载到 PHP 超级全局变量 ` $_ENV ` 里。你可以使用**辅助方法 env 来获取这些变量的值**。
**应用程序的当前环境是由 .env 文件中的 APP_ENV 变量所决定。**你可以通过 App facade 的 ` environment 方法 `获取该值：
```

$environment = App::environment();

```
**你也可以传递参数至 environment 方法中，来确认当前环境是否与参数相符合**：
```

if (App::environment('local')) {
    // 环境是 local
}
if (App::environment('local', 'staging')) {
    // 环境是 local 或 staging...
}

```
也可通过 app 辅助方法获取应用程序实例：
```

$environment = app()->environment();

```
##  缓存配置信息 
为了让应用程序的速度获得提升，**可以使用 Artisan 命令 config:cache 将所有的配置文件缓存到单个文件。通过此命令将所有的设置选项合并成一个文件，让框架能够更快速的加载。**
你应该将运行`  php artisan config:cache ` 命令作为部署工作的一部分。**此命令不应该在本机开发时运行**，因为设置选项会在应用程序的开发时经常变动。

**获取设置值**
你可以使用 **config 辅助方法获取你的设置值**，设置值可以通过「点」语法来获取，其中包含了文件与选项的名称。你也可以指定一个默认值，当该设置选项不存在时就会返回默认值：
```

$value = config('app.timezone');

```
若要**在运行期间修改设置值**，请传递一个数组至 config 辅助方法：
```

config(['app.timezone' => 'America/Chicago']);

```

##  命名你的应用程序 
在安装完成 Laravel 后，你可以来「命名」你的应用程序。**默认情况下，app 的目录的命名空间是 App，然后会通过 Composer 使用 PSR-4 自动加载标准 来自动加载。**不过，你可以轻松地通过 **Artisan 命令 app:name 来修改命名空间**，以配合你的应用程序名称。
举例来说，假设你的应用程序叫做「Horsefly」，你可以在安装完的根目录运行下方的命令：
```

php artisan app:name Horsefly

```
你可以随意重命名你的应用程序，如果你愿意的话也可以继续保持命名空间为 App。

##  维护模式 
当你的应用程序处于维护模式时，所有传递至应用程序的请求都会显示出一个自定义视图。在你更新应用或进行性能维护时，这么做可以很轻松的「关闭」整个应用程序。维护模式会检查包含在应用程序的默认的中间件堆栈。**如果应用程序处于维护模式，HttpException 会抛出 503 的状态码。**
启用维护模式，只需要运行 Artisan 命令 down：
```

php artisan down

```
关闭维护模式，请使用 Artisan 命令 up：
```

php artisan up

```
**维护模式的响应模板**
维护模式的默认模板放在 ` resources/views/errors/503.blade.php `。
**维护模式与队列**
当应用程序处于维护模式中时，将不会处理任何[队列工作](http://laravel-china.org/docs/5.1/queues)。所有的队列工作将会在应用程序离开维护模式后继续被运行。