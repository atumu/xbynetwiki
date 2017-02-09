title: phplearn023 

#  PHP学习之面向对象学习组合之性状(trait) 
性状(trait).这是PHP5.4引入的概念，既像类又像接口。性状(trait)是类的部分实现,可以混入(Mixin)一个或多个现有的PHP类中。
trait有两个作用:表明类可以做什么(像是接口)，提供模块化实现(像是类)。
trait拥有继承没有的优点：那就是可以把模块化实现的方式注入多个无关的类中，促进代码重用。
例如:
定义trait:
```

<?php
trait Geocodable
{
    /** @var string */
    protected $address;

    /** @var \Geocoder\Geocoder */
    protected $geocoder;

    /** @var \Geocoder\Model\AddressCollection */
    protected $geocoderResult;

    public function setGeocoder(\Geocoder\Geocoder $geocoder)
    {
        $this->geocoder = $geocoder;
    }

    public function setAddress($address)
    {
        $this->address = $address;
    }

    public function getLatitude()
    {
        if (!isset($this->geocoderResult)) {
            $this->geocodeAddress();
        }

        return $this->geocoderResult->first()->getLatitude();
    }

    public function getLongitude()
    {
        if (!isset($this->geocoderResult)) {
            $this->geocodeAddress();
        }

        return $this->geocoderResult->first()->getLongitude();
    }

    protected function geocodeAddress()
    {
        $this->geocoderResult = $this->geocoder->geocode($this->address);

        return true;
    }
}


```
**使用trait， PHP性状的使用方法很简单，把use MyTrait;语句加到类的定义体中即可。**
```

<?php
class RetailStore
{
    use Geocodable;

    // Class implementation goes here
}


```

**注意：命名空间和trait都使用use，但是使用的地方不同。命名空间、类、常量、接口、函数在类的定义体之外导入，而trait在类的定义体内倒入。**
PHP解释器会在编译时把性状复制粘贴到类的定义体中。

使用性状:
```

<?php
require 'vendor/autoload.php';
require 'Geocodable.php';
require 'RetailStore.php';

$adapter = new \Ivory\HttpAdapter\CurlHttpAdapter();
$geocoder = new \Geocoder\Provider\GoogleMaps($adapter);

$store = new RetailStore();
$store->setAddress('420 9th Avenue, New York, NY 10001 USA');
$store->setGeocoder($geocoder);

$latitude = $store->getLatitude();
$longitude = $store->getLongitude();

echo $latitude, ':', $longitude;


```
