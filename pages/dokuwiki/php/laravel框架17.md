title: laravel框架17 

#  Laravel之集合操作 
` Illuminate\Support\Collection ` 类提供一个流畅、便利的封装来操控数组数据。举个例子，查看下列的代码。我们将用` Laravel的 collect 辅助方法 `从数组创建一个新的集合实例，对每一个元素运行 strtoupper 函数，然后移除所有的空元素：
```

$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
    return strtoupper($name);
})
->reject(function ($name) {
    return empty($name);
});

```
如你所见，**Collection 类允许你链式调用它的方法以对底层的数组流畅地进行映射与删减。一般来说，每一个 Collection 方法会返回一个全新的 Collection 实例，你可以放心地进行链接调用。**

创建集合#
如上所述，` collect 辅助方法 `会用传入的数组返回一个新的 ` Illuminate\Support\Collection ` 实例。所以要创建一个集合就这么简单：
```

$collection = collect([1, 2, 3]);

```
**可用的方法#**
要记得的是，所有方法都能被链式调用调用，几乎所有的方法都会返回新的 Collection 实例，让你保留原版的集合以备不时之需。
![](/data/dokuwiki/php/pasted/20160419-095348.png)
具体参考http://laravel-china.org/docs/5.1/collections
