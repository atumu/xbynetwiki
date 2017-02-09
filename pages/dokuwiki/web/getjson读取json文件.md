title: getjson读取json文件 

#  $.getJSON读取json文件 
jQuery.getJSON( url [, data ] [, success(data, textStatus, jqXHR) ] )
 使用一个HTTP GET请求从服务器加载JSON编码的数据。
```

$.getJSON('ajax/test.json', function(data) {
  var items = [];
 
  $.each(data, function(key, val) {
    items.push('<li id="' + key + '">' + val + '</li>');
  });
 
  $('<ul/>', {
    'class': 'my-new-list',
    html: items.join('')
  }).appendTo('body');
});

```