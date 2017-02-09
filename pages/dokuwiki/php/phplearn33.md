title: phplearn33 

#  PHP学习之PHP-FIG标准PSR与流行组件 
PHP组件和框架数量很多。有Symfony和Laravel这样的大型框架，也有Silex和Slim这样的微型框架。也有专一的日志框架monolog,phpmailer,swiftmailer，PHPUnit,Twig,Guzzle,Carbon等
PHP社区已经从中心化框架模型进化为分布式生态系统了。组件效率高、互操作性好、作用专一。

##  打破旧局面的PHP-FIG 
PHP Framework Group (PHP-FIG,http://www.php-fig.org)为了提高框架的互操作性，指定了一系列推荐规范。
PSR(PHP Standards Recommandation)：PHP推荐标准：目前共发布了五个推荐规范:
PSR-0:已废弃，被PSR-4替代。
PSR-1：基本的代码风格
PSR-2:严格的代码风格
PSR-3:日志记录器接口。最流行的实现的monolog/monolog,请参考[请参考这里](/pages/dokuwiki/php/phplearn09)
PSR-4:自动加载标准。

##  流行的PHP框架 
Aura
Laravel
Symfony
Yii
Zend
##  流行的组件 
PHP组件查找:https://packagist.org
国外友人收集的最好PHP组件列表:https://github.com/ziadoz/awesome-php
Composer组件依赖管理器。
日志:monolog/monolog
HTTP客户端:guzzlehttp/guzzle
处理CSV：league/csv
验证与过滤:aura/filter, respect/validation, symfony/validator
模板引擎:twig/twig,smarty/samrty
日期组件:nesbot/carbon
文件系统:league/flysystem
##  自定义组件 
组件样板仓库:https://github.com/thephpleague/skeleton
示例:https://github.com/modern-php/scanner
composer.json:
```

{
    "name": "modernphp/scanner",
    "description": "Scan URLs from a CSV file and report inaccessible URLs",
    "keywords": ["url", "scanner", "csv"],
    "homepage": "http://example.com",
    "license": "MIT",
    "authors": [
        {
            "name": "Josh Lockhart",
            "homepage": "https://github.com/codeguy",
            "role": "Developer"
        }
    ],
    "support": {
        "email": "help@example.com"
    },
    "require": {
        "php" : ">=5.6.0",
        "guzzlehttp/guzzle": "^6.1"
    },
    "require-dev": {
        "phpunit/phpunit": "^5.0"
    },
    "suggest": {
        "league/csv": "^8.0"
    },
    "autoload": {
        "psr-4": {
            "Oreilly\\ModernPHP\\": "src/"
        }
    }
}


```
src/Url/Scanner.php:
```

<?php
namespace Oreilly\ModernPHP\Url;

class Scanner
{
    /**
     * @var array An array of URLs
     */
    protected $urls;

    /**
     * @var \GuzzleHttp\Client
     */
    protected $httpClient;

    /**
     * Constructor
     * @param array $urls An array of URLs to scan
     */
    public function __construct(array $urls)
    {
        $this->urls = $urls;
        $this->httpClient = new \GuzzleHttp\Client();
    }

    /**
     * Get invalid URLs
     * @return array
     */
    public function getInvalidUrls()
    {
        $invalidUrls = [];
        foreach ($this->urls as $url) {
            try {
                $statusCode = $this->getStatusCodeForUrl($url);
            } catch (\Exception $e) {
                $statusCode = 500;
            }

            if ($statusCode >= 400) {
                array_push($invalidUrls, [
                    'url' => $url,
                    'status' => $statusCode
                ]);
            }
        }

        return $invalidUrls;
    }

    /**
     * Get HTTP status code for URL
     * @param string $url The remote URL
     * @return int The HTTP status code
     */
    protected function getStatusCodeForUrl($url)
    {
        $httpResponse = $this->httpClient->options($url);

        return $httpResponse->getStatusCode();
    }
}


```
使用:
```

<?php
require 'vendor/autoload.php';

$urls = [
    'http://www.apple.com',
    'http://php.net',
    'http://sdfssdwerw.org'
];
$scanner = new \Oreilly\ModernPHP\Url\Scanner($urls);
print_r($scanner->getInvalidUrls());


```