title: phplearn36 

#  PHP学习之PHPUnit与Xdebug测试与分析 
**PHPUnit**
官网:https://phpunit.de/
中文官方手册:https://phpunit.de/manual/current/zh_cn/index.html
github:https://github.com/sebastianbergmann/phpunit

安装:
```

composer  require --dev phpunit/phpunit

```
测试运行程序二进制文件会安装在vendor/bin/phpunit

概念:测试用例(test case)，测试组件(test suite)，运行测试(test runner)。
约定:
1、测试用例的类名必须以Test结尾，而且所在的文件名必须以Test.php结尾。
2、每个测试用例类继承` PHPUnit_Framework_TestCase `类

目录结构:
```

src/
tests/
	bootstrap.php 运行单元测试之前需要引入这个文件
composer.php
phpunit.xml 配置PHPUnit的测试运行程序
.travis.yml 用于配置持续测试Web服务Travis CI

```

安装Xdebug：(XHProf、Blackfire是与Xdebug功能类似的又一个分析器)
Xdebug是个PHP扩展。
ubuntu: sudo apt-get install php5-xdebug
更新php.init:
zend_extension="/path/to/xdebug.so"
xdebug.profiler_enable=0     #不让Xdebug在每次请求时都自动运行，这会极大地降低性能。阻碍开发
xdebug.profiler_enable_trigger=1  #可以在PHP应用的任何一个URL中加上XDEBUG_PROFILE=1查询参数，在单个请求中启动Xdebug.
xdebug.profiler_output_dir=/path/to/profiler/results #保存分析器生成的报告

分析报告:
Xdebug生成的报告是CacheGrind格式。我们需要通过工具来查看:
  * windows下使用WinCacheGrind
  * Linux下使用KCacheGrind
  * Web浏览器中运行WebGrind
  * Mac下brew install qcachegrind




PHP用于运行测试，Xdebug用于生成有用的覆盖度信息。

##  配置PHPUnit 
phpunit.xml:
```

<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="tests/bootstrap.php">
    <testsuites>
        <testsuite name="whovian">
            <directory suffix="Test.php">tests</directory>
        </testsuite>
    </testsuites>

    <filter>
        <whitelist>
            <directory>src</directory>
        </whitelist>
    </filter>
</phpunit>


```
```

<?php
require dirname(__DIR__) . '/src/Whovian.php';

class WhovianTest extends PHPUnit_Framework_TestCase
{
    public function testSetsDoctorWithConstructor()
    {
        $whovian = new Whovian('Peter Capaldi');
        $this->assertAttributeEquals('Peter Capaldi', 'favoriteDoctor', $whovian);
    }

    public function testSaysDoctorName()
    {
        $whovian = new Whovian('David Tennant');
        $this->assertEquals('The best doctor is David Tennant', $whovian->say());
    }

    public function testRespondToInAgreement()
    {
        $whovian = new Whovian('David Tennant');

        $opinion = 'David Tennant is the best doctor, period';
        $this->assertEquals('I agree!', $whovian->respondTo($opinion));
    }

    /**
     * @expectedException Exception
     */
    public function testRespondToInDisagreement()
    {
        $whovian = new Whovian('David Tennant');

        $opinion = 'No way. Matt Smith was awesome!';
        $whovian->respondTo($opinion);
    }
}


```
##  运行测试 
运行测试:vendor/bin/phpunit -c phpunit.xml
代码覆盖度:vendor/bin/phpunit -c phpunit.xml --coverage-html coverage
持续测试:Travis CI 

