title: phplearn27 

#  PHP学习之依赖管理工具Composer 
官网:https://getcomposer.org
中文:http://www.phpcomposer.com/
官方仓库:http://packagist.org
中国仓库:http://pkg.phpcomposer.com/
对于现代语言而言，包管理器基本上是标配。Java 有 Maven，Python 有 pip，Ruby 有 gem，Nodejs 有 npm。PHP 的则是 PEAR，不过 PEAR 坑不少：
依赖处理容易出问题
配置非常复杂
难用的命令行接口
好在我们有 Composer，PHP依赖管理的利器。
##  快速入门 
###  安装 Composer 
Composer 需要 PHP 5.3.2+ 才能运行。
$ curl -sS https://getcomposer.org/installer | php
这个命令会将 composer.phar 下载到当前目录。PHAR（PHP 压缩包）是一个压缩格式，可以在命令行下直接运行。
你可以使用 --install-dir 选项将 Composer 安装到指定的目录，例如：
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=bin
当然也可以进行全局安装：
$ curl -sS https://getcomposer.org/installer | php

$ mv composer.phar /usr/local/bin/composer
在 Mac OS X 下也可以使用 homebrew 安装：
brew tap josegonzalez/homebrew-php  
brew install josegonzalez/php/composer  

windows下安装:
下载并且运行 [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe)，它将安装最新版本的 Composer ，并设置好系统的环境变量，因此你可以在任何目录下直接使用 composer 命令。

不过**通常情况下只需将 composer.phar 的位置加入到 PATH 环境变量就可以，不一定要全局安装。**

###  声明依赖 
在项目目录下创建一个 ` composer.json ` 文件，指明依赖，比如，你的项目依赖 monolog：
```

{
    "require": {
        "monolog/monolog": "1.2.*"
    }
}

```
###  安装依赖 
` packagist.org ` 是Composer的仓库，很多著名的 PHP 库都能在其中找到。
安装依赖非常简单，只需在项目目录下运行：
` composer require 'phpdocumentor/phpdocumentor:1.8.0'  ` 

###  自动加载 
Composer 提供了自动加载的特性，只需在你的代码的初始化部分中加入下面一行：
` require 'vendor/autoload.php'; `  
###  配置仓库 
参考https://getcomposer.org/doc/06-config.md
本地仓库地址:C:\Users\<user>\AppData\Local\Composer
国内仓库:国内http://pkg.phpcomposer.com/
**例1：修改 composer 的全局配置文件**
打开命令行窗口（windows用户）或控制台（Linux、Mac 用户）并执行如下命令：
composer config -g repo.packagist composer https://packagist.phpcomposer.com
**例2：修改当前项目的 composer.json 配置文件：**
打开命令行窗口（windows用户）或控制台（Linux、Mac 用户），进入你的项目的根目录（也就是 composer.json 文件所在目录），执行如下命令：
composer config repo.packagist composer https://packagist.phpcomposer.com
上述命令将会在当前项目中的 composer.json 文件的末尾自动添加镜像的配置信息（你也可以自己手工添加）：
```

"repositories": {
    "packagist": {
        "type": "composer",
        "url": "https://packagist.phpcomposer.com"
    }
}

```
以 laravel 项目的 composer.json 配置文件为例，执行上述命令后如下所示（注意最后几行）：
```

{
    "name": "laravel/laravel",
    "description": "The Laravel Framework.",
    "keywords": ["framework", "laravel"],
    "license": "MIT",
    "type": "project",
    "require": {
        "php": ">=5.5.9",
        "laravel/framework": "5.2.*"
    },
    "require-dev": {
        "fzaninotto/faker": "~1.4",
        "mockery/mockery": "0.9.*",
        "phpunit/phpunit": "~4.0",
        "symfony/css-selector": "2.8.*|3.0.*",
        "symfony/dom-crawler": "2.8.*|3.0.*"
    },
    "autoload": {
        "classmap": [
            "database"
        ],
        "psr-4": {
            "App\\": "app/"
        }
    },
    "autoload-dev": {
        "classmap": [
            "tests/TestCase.php"
        ]
    },
    "scripts": {
        "post-root-package-install": [
            "php -r \"copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "php artisan key:generate"
        ],
        "post-install-cmd": [
            "php artisan clear-compiled",
            "php artisan optimize"
        ],
        "pre-update-cmd": [
            "php artisan clear-compiled"
        ],
        "post-update-cmd": [
            "php artisan optimize"
        ]
    },
    "config": {
        "preferred-install": "dist"
    },
    "repositories": {
        "packagist": {
            "type": "composer",
            "url": "https://packagist.phpcomposer.com"
        }
    }
}

```
OK，一切搞定！试一下 composer install 来体验飞一般的速度吧！

###  更新依赖版本 
更新你的依赖版本请使用 update 命令。这将获取最新匹配的版本（根据你的 composer.json 文件）并将新版本更新进锁文件。
php composer.phar update
如果只想安装或更新一个依赖，你可以白名单它们：
php composer.phar update monolog/monolog [...]

###  更新composer版本 
composer selfupdate

###  部署 
当你在部署代码的时候，你只需运行composer install或者composer update。
##  使用说明 
**配置文件**
在我们开始一个项目的时候，首先会给项目取一个名字，我们暂且叫丝绸之路吧，代号silk。首先要写一个Composer的配置文件，来描述项目，为此，
在项目的根目录下，建立文件名为` composer.json `的配置文件。内容如下：
```

{
"name":             "meta/silk",
"description":      "another e-commerce website",
"keywords":         ["silk", "online shop", "good"],
"homepage":         "http://www.xxx.com ",
"time":             "2014-12-30",
"license":          "MIT",
"authors": [
    {
        "name":         "Elvis Lim",
        "email":        "elvis@xxx.com",
        "homepage":     "http://www.xxx.com",
        "role":         "Engineer"
    }
]}

```
###  依赖管理 
在composer.json文件里增加一个新的字段：require。这个字段的值是一个对象，同样以键值对的形式构成。以上述提到的两个依赖位置，写成composer管理的方式如下：
```

"require": {
"swiftmailer/swiftmailer": 5.3.*@dev,
"phpoffice/phpexcel": "dev-master"
}

```
以swiftmailer为例，swiftmailer/swiftmailer 代表的是包名称，5.3.@dev , 是版本信息。合起来的意思就是说，我们将要开发的应用，依赖于swiftmailer的5.3.版本。其中：
5.3.*表示，可以使用5.3.1版本，也可以使用5.3.2版本，composer在获取的时候，将寻找5.3版本下最新的版本。版本号支持一些更加宽泛的约束，比如>=1.0, >=1.0, <2.0，更加具体的信息可以查看：http://docs.phpcomposer.com/01-basic-usage.md#The-require-Key

**@dev表示可以获取开发版本**。稳定性标签可以作用于特定的依赖项，也可以作用于全局。
作用特定依赖项：**默认情况下，composer只会获取稳定版本**，如果这个例子我们不加@dev约束，而5.3.*版本都是开发版本，那么在获取的时候composer就会报错，指出改版本不符合要求。如果确定这个开发版本没有问题，那么就可以通过加@dev ，让Composer获取这个开发版本。

**全局稳定性设置：通过设置minimum-stability的值，来告诉Composer当前开发的项目的依赖要求的包的全局稳定性级别，它的值包括：dev、alpha、beta、RC、stable，stable是默认值。**

至此，两个依赖添加完毕，我们可以运行下**Composer包更新命令**，看看效果啦。
```

composer install

```
成功运行完毕，会在根目录下发现vendor文件夹，里面包含了刚刚我们列出来的两个包文件代码。

**require-dev**
有时候，我们会发现，**有些包依赖只会在开发过程中使用，正式发布的程序不需要这些包**，这个时候，就需要用到另外一个键，即require-dev。例如，我们想用codeception进行单元测试，那么就可以通过require-dev引入这个开发环境下的依赖包：
```

“require-dev”: {
"codeception/codeception": "2.0.0 "
}

```
加了这个依赖后，再运行下命令看看效果。
` composer install `

###  版本号说明 
![](/data/dokuwiki/php/pasted/20160408-150230.png)
` 符号^ ` 表示 ^1.2.3 is equivalent to >=1.2.3 <2.0.0(而~1.2.3表示>=1.2.3 <1.3.0)， For pre-1.0 versions it also acts with safety in mind and treats ^0.3 as >=0.3.0 <0.4.0.
Example: ^1.2.3
https://getcomposer.org/doc/articles/versions.md
###  命令行使用 
http://docs.phpcomposer.com/03-cli.html
###  自动加载 
自此，composer已经帮我们把需要的库文件下载下来啦，接下去想到的就是如何引用这些库文件。最简单的方式就是require或者include，但这就不够高大上了啊，需要花时间去库文件里查看需要引入哪些文件，费事而且容易出错。**好在composer可以帮我们解决这个问题。那就是autoload。**
**在运行完composer install命令后**，怎么调用PHPExcel库呢？很简单，**只要引入vendor目录下的autoload.php文件就可以了**。可以在根目录下，建一个index.php文件，加入一下内容：
```

include “vendor/autoload.php”
$excel = new PHPExcel();
var_dump($excel);

```
用浏览器访问一下这个页面，就会发现PHPExcel对象已经被成功创建啦，是不是很方便？
**其实到目前为止，我们并没用在composer.json文件里加入autoload字段，那么什么时候需要加入呢？ 那就是当我们想让composer帮我们自动加载我们自己定义的类的时候。**
####  方式一: 
例如，我们自己写了个订单管理类，取名OrderManager，放在lib目录下的OrderManager.php文件里。内容如下：
```

class OrderManager
{
    public function test()
    {
        echo "hello";
    }
}

```
那么如何让composer帮我们自动加载这个类呢？ 在composer.json里加入下面的内容：
```

“autoload”:{
    "files":["lib/OrderManager.php"]
}

```
files键对应的值是一个数组，数组元素是文件的路径，路径是相对于应用的根目录。加上上述内容后，运行命令：
` composer dump-autoload `
让composer**重建自动加载的信息**，完成之后，就可以在index.php里调用OrderManager类啦。

通过文件引入的方法虽然直观，但是很费劲，每个文件都得引入一次，实在不是好的解决办法。
####  方式二: 
**有没有更好的办法呢？尝试将autoload的值改成：**
```

 "classmap":["lib"]

```
再此运行composer dump-autoload，尝试调用，依然能够成功创建OrderManager类。其实，classmap通过建立类到文件的对应关系，当程序需要OrderManager类时，compoer的自动加载类通过查找OrderManager类所在的文件，然后再将改文件include进来。因此，这又导致了一个问题，那就是每加一个新类，就需要运行一次composer dump-autoload来创建类到文件到对应关系，比files方法虽然好一点，但是还是很不够舒爽啊！于是，PSR出场了。先了解下什么是PSR。
FIG组织制定的一组PHP相关规范，简称PSR，其中
  * PSR-0自动加载 （已被废弃）
  * PSR-1基本代码规范 
  * PSR-2代码样式 
  * PSR-3日志接口 
  * PSR-4 自动加载
PSR-4规范的具体内容见：https://github.com/hfcorriez/fig-standards/blob/zh_CN/%E6%8E%A5%E5%8F%97/PSR-4-autoloader.md
简而言之，**就是希望通过一组约定的目录，文件名，类名定义方式，来实现快速通过全限定类名查找到文件，然后包含进来，实现自动加载。** 
PSR-4和PSR-0最大的区别是对下划线（underscore）的定义不同。PSR-4中，在类名中使用下划线没有任何特殊含义。而PSR-0则规定类名中的下划线_会被转化成目录分隔符。
不管是PSR-0还是PSR-4，**都要求有个命名空间**，所以我们需要对OrderManager类进行一些小的修改，**加上命名空间**：
```

namespace Silk;
class OrderManager
{
    public function test()
    {
        echo "hello";
    }
 }

```
提示要加上分隔符：
```

"autoload":{
    "psr-4":{
        "Silk\\":"lib"
    }
}

```
**说明:全限定类名:Silk\OrderManager,对应的物理路径lib\OrderManager**
再次**composer dump-autoload**，运行测试，OK通过！
掌握require和autoload部分，其实就算Compoer入门啦，在详细的内容，可以通过查看composer文档来了解。Happy Coding！


参考:http://my.oschina.net/u/248080/blog/359008
http://docs.phpcomposer.com/01-basic-usage.html