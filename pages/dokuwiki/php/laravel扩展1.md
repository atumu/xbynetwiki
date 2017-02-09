title: laravel扩展1 

#  在Laravel5中使用Purifier扩展包集成HTMLPurifier防止XSS跨站攻击 
http://www.tuicool.com/articles/2u2QZ3v
https://phphub.org/topics/36
1、安装
` HTMLPurifier ` 是基于 PHP 编写的富文本**HTML 过滤器**，通常我们可以使用它来**防止XSS 跨站攻击**，更多关于 HTMLPurifier的详情请参考其**官网： http://htmlpurifier.org/** 。Purifier 是在Laravel 5 中集成 HTMLPurifier 的扩展包，我们可以通过 Composer 来安装这个扩展包：
```

composer require mews/purifier

```
安装完成后，在配置文件 ` config/app.php 的 providers ` 中**注册HTMLPurifier服务提供者**：
```

'providers' => [
    // ...
    Mews\Purifier\PurifierServiceProvider::class,
]

```
然后在 aliases 中**注册Purifier门面**：
```

'aliases' => [
    // ...
    'Purifier' => Mews\Purifier\Facades\Purifier::class,
]

```
2、配置
**要使用自定义的配置，发布配置文件到 config 目录**：
```

php artisan vendor:publish

```
这样会在 config 目录下生成一个`  purifier.php ` 文件：
```

return [

    'encoding' => 'UTF-8',
    'finalize' => true,
    'preload'  => false,
    'cachePath' => null,
    'settings' => [
        'default' => [
            'HTML.Doctype'             => 'XHTML 1.0 Strict',
            'HTML.Allowed'             => 'div,b,strong,i,em,a[href|title],ul,ol,li,p[style],br,span[style],img[width|height|alt|src]',
            'CSS.AllowedProperties'    => 'font,font-size,font-weight,font-style,font-family,text-decoration,padding-left,color,background-color,text-align',
            'AutoFormat.AutoParagraph' => true,
            'AutoFormat.RemoveEmpty'   => true
        ],
        'test' => [
            'Attr.EnableID' => true
        ],
        "youtube" => [
            "HTML.SafeIframe" => 'true',
            "URI.SafeIframeRegexp" => "%^(http://|https://|//)(www.youtube.com/embed/|player.vimeo.com/video/)%",
        ],
    ],

];

```
3、使用示例
可以使用**辅助函数 clean** ：
```

clean(Input::get('inputname'));

```
或者使用 Purifier 门面提供的 clean 方法：
```

Purifier::clean(Input::get('inputname'));

```
还可以在应用中进行动态配置：
clean('This is my H1 title', 'titles');
clean('This is my H1 title', array('Attr.EnableID' => true));
或者你也可以使用 Purifier 门面提供的方法：
Purifier::clean('This is my H1 title', 'titles');
Purifier::clean('This is my H1 title', array('Attr.EnableID' => true));