title: laravel框架23 

#  Laravel之本地化 
Laravel 的本地化功能提供方便的方法来获取多语言的字符串，让你的网站可以简单的支持多语言。
语言包存放在 ` resources/lang ` 文件夹的文件里。在此文件夹内应该有网站对应支持的语言并将其对应到每一个子目录：
```

/resources
    /lang
        /en
            messages.php
        /es
            messages.php

```
语言包简单地返回键值和字符串数组，例如：
```

<?php

return [
    'welcome' => 'Welcome to our application'
];

```
**切换语言#**
网站的默认语言保存在`  config/app.php ` 配置文件。你可以在任何时候使用 **App facade 的 setLocale 方法动态地更改现有语言**：
```

Route::get('welcome/{locale}', function ($locale) {
    App::setLocale($locale);

    //
});

```
你也可以设置 "备用语言"，它将会在当现有语言没有指定语句时被使用。就像默认语言，备用语言也可以在 config/app.php 配置文件设置：
```

'fallback_locale' => 'en',

```
**基本用法#**
你可以使用 trans 辅助函数来获取语言字符串，trans 函数的第一个参数接受文件名和键值名称，例如，从 resources/lang/messages.php 语言包获取名称为 welcome 的句子：
```

echo trans('messages.welcome');

```
当然，若你使用了 Blade 模版引擎, 则可以使用 ![](/data/dokuwiki ) 来输出句子：
```

![](/data/dokuwiki trans('messages.welcome') )

```
如果句子不存在， trans 方法将会返回键值的名称，如上例子会返回 messages.welcome 。

**在句子中做替代#**
如果需要，**你也可以在语言包中定义占位符，占位符使用 : 开头**，例如，你可以自定义一则欢迎消息的占位符：
```

'welcome' => 'Welcome, :name',

```
接着，传入替代用的第二个参数给 trans 方法：
```

echo trans('messages.welcome', ['name' => 'Dayle']);

```

