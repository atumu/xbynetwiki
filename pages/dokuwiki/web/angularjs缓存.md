title: angularjs缓存 

#  AngularJS缓存 
```

<script src="http://ajax.googleapis.com/ajax/libs/angularjs/1.2.2/angular.js"></script>
<script>
  angular.module('app.ui', [])
   .run(['$templateCache','$cacheFactory',
     function($templateCache,$cacheFactory) {
       var cachetest = $cacheFactory('cache');
       cachetest.put('a','This is the content of the template');
  }]);
  angular.module('app', ['app.ui'])
    .run(['$templateCache','$cacheFactory',
      function(temp,cache) {
       var cachetest = cache.get('cache');
      temp.put('test',cachetest.get('a') );
       cachetest.destroy();
    }]);
  angular.bootstrap(document, ['app']);
</script>

```
$cacheFactory创建的的对象,有如下方法
{object} info() — 返回大小、id、一些缓存的参数
{value} put(key,value) — 插入键值对,键为字符串,值为任何类型,返回键值对中的值
{value} get(key) —通过键返回值，如果不存在返回 undefined
{void} remove(key) — 通过键来移除,无返回值
{void} removeAll() — 移除所有的模板缓存,无返回值
{void} destroy() —移除当前创建的缓存($templateCache无效)
$cacheFactory有所创建的对象直接通过$cacheFactory('cacheID',[,option])
其中的option对象主要有capacity参数为数字类型设置最大的列表长度, 默认值为Number.MAX_VALUE,当超过的时候最早的会被移除,当设置最长长度为2的时候, 方法如下
$cacheFactory('cache',{capacity:2});
获取的时候调用get方法参数为cache的ID即可,当不存在时返回undefined

##  $http中的缓存 
```

$http({
	method:'GET',
	url:'/api/users.json',
	cache:true
})

```
参考：http://www.geekcome.com/content-10-2612-1.html