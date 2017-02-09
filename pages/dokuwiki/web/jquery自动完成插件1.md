title: jquery自动完成插件1 

#  jQuery自动完成插件之Bootstrap typeahead 
在jQuery中有个插件jQuery autocomplete,而其实在bootsrap中也有。
推荐两款：
1、推特开源的typeahead:http://twitter.github.io/typeahead.js/ 不过已经废弃，不推荐。
2、Bootstrap-3-Typeahead：https://github.com/bassjobsen/Bootstrap-3-Typeahead 推荐
##  Bootstrap-3-Typeahead 
Download the latest bootstrap3-typeahead.js or bootstrap3-typeahead.min.js.
Include it in your source after jQuery and Bootstrap's JavaScript.
There is no additional CSS required to use the plugin. 
使用：
```

<input type="text" autocomplete="off" data-provide="typeahead">

```
@* autocomplete 避免浏览器的自动提示对下拉选项的覆盖操作 *@
你也可以Via JavaScript形式
```

$('.typeahead').typeahead()

```
Destroys previously initialized typeaheads. This entails reverting DOM modifications and removing event handlers:
```

$('.typeahead').typeahead('destroy')

```
###  示例： 
常用参数说明
  * source：是个function或者 基本类型的数组。
  * items :下拉选项展示的个数
  * afterSelect：选中之后执行的回调函数。

```

$.get('example_collection.json', function(data){
    $("#name").typeahead({ source:data });
},'json');
//example_collection.json
// ["item1","item2","item3"]

```
```

var $input = $('.typeahead');
$input.typeahead({source:[{id: "someId1", name: "Display name 1"}, 
            {id: "someId2", name: "Display name 2"}], 
            autoSelect: true}); 
$input.change(function() {
    var current = $input.typeahead("getActive");
    if (current) {
        // Some item from your model is active!
        if (current.name == $input.val()) {
            // This means the exact match is found. Use toLowerCase() if you want case insensitive match.
        } else {
            // This means it is only a partial match, you can either add a new item 
            // or take the active if you don't want new items
        }
    } else {
        // Nothing is active so it is a new value (or maybe empty value)
    }
});

```

更多使用可以参考：http://www.cnblogs.com/sheldon-lou/p/4166076.html
###  与AngularJS集成 
https://github.com/davidkonrad/angular-bootstrap3-typeahead
###  Bloodhound 
Bloodhound is the typeahead.js suggestion engine, since version 0.10.0. Bloodhound is robust, flexible, and offers advanced functionalities such as prefetching, intelligent caching, fast lookups, and backfilling with remote data. To use Bloodhound with Bootstrap-3-Typeahead:
```

// instantiate the bloodhound suggestion engine
var numbers = new Bloodhound({
datumTokenizer: Bloodhound.tokenizers.whitespace,
queryTokenizer: Bloodhound.tokenizers.whitespace,
local:  ["(A)labama","Alaska","Arizona","Arkansas","Arkansas2","Barkansas"]
});

// initialize the bloodhound suggestion engine
numbers.initialize();

$('.typeahead').typeahead(
{
items: 4,
source:numbers.ttAdapter()  
});

```
###  Bootstrap Tags Input集成 
```

$('input').tagsinput({
  typeahead: {
    source: ['Amsterdam', 'Washington', 'Sydney', 'Beijing', 'Cairo']
  }
});
or

$('input').tagsinput({
  typeahead: {                  
    source: function(query) {
      return $.get('http://someservice.com');
    }
  }
});

```