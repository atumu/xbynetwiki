title: jquery标签输入框插件 

#  jQuery标签tag输入框插件 
tag输入框插件集合，请参考:http://jquery-plugins.net/tag/tag-input
![](/data/dokuwiki/web/pasted/20160505-152053.png)
本文主要介绍两个:
1、bootstrap-tagsinput:https://github.com/bootstrap-tagsinput/bootstrap-tagsinput
2、Tag-it:https://aehlke.github.io/tag-it/

#  bootstrap-tagsinput 
示例:http://bootstrap-tagsinput.github.io/bootstrap-tagsinput/examples/
特点：
  * Objects as tags
  * True multi value
  * Typeahead（即自动补全）
  * Designed for Bootstrap 2.3.2 and 3

###  标记启用: 

只需要添加 ` data-role="tagsinput" ` 到你的**input field** to automatically change it to a **tags input field.**
```

<input type="text" value="Amsterdam,Washington,Sydney,Beijing,Cairo" data-role="tagsinput" />

```
statement			returns
```

$("input").val()		"Amsterdam,Washington,Sydney,Beijing,Cairo"
$("input").tagsinput('items')	["Amsterdam","Washington","Sydney","Beijing","Cairo"]

```
###  支持select multiple 
```

<select multiple data-role="tagsinput">
  <option value="Amsterdam">Amsterdam</option>
  <option value="Washington">Washington</option>
  <option value="Sydney">Sydney</option>
  <option value="Beijing">Beijing</option>
  <option value="Cairo">Cairo</option>
</select>

```
![](/data/dokuwiki/web/pasted/20160505-152822.png)
```

$("select").val()	 ["Amsterdam","Washington","Sydney","Beijing","Cairo"]
$("select").tagsinput('items')	["Amsterdam","Washington","Sydney","Beijing","Cairo"]

```
###  Typeahead(自动补全) 
![](/data/dokuwiki/web/pasted/20160505-153259.png)
首先下载: [typeahead.js](https://github.com/bassjobsen/Bootstrap-3-Typeahead)
Typeahead is not included in Bootstrap 3, so you'll have to include your own typeahead library. I'd recommed [typeahead.js](https://github.com/bassjobsen/Bootstrap-3-Typeahead). An example of using this is shown below.
```

<input type="text" value="Amsterdam,Washington" data-role="tagsinput" />
<scri1pt>
var citynames = new Bloodhound({
  datumTokenizer: Bloodhound.tokenizers.obj.whitespace('name'),
  queryTokenizer: Bloodhound.tokenizers.whitespace,
  prefetch: {
    url: 'assets/citynames.json',
    filter: function(list) {
      return $.map(list, function(cityname) {
        return { name: cityname }; });
    }
  }
});
citynames.initialize();

$('input').tagsinput({
  typeahead: {
    name: 'citynames',
    displayKey: 'name',
    valueKey: 'name',
    source: citynames.ttAdapter()
  }
});
</scri1pt>

```
```

$("input").val()	"Amsterdam,Washington"
$("input").tagsinput('items')	["Amsterdam","Washington"]

```

###  多彩tag样式 
![](/data/dokuwiki/web/pasted/20160505-153618.png)
```

<input type="text" />
<script>
var cities = new Bloodhound({
  datumTokenizer: Bloodhound.tokenizers.obj.whitespace('text'),
  queryTokenizer: Bloodhound.tokenizers.whitespace,
  prefetch: 'assets/cities.json'
});
cities.initialize();

var elt = $('input');
elt.tagsinput({
  tagClass: function(item) {
    switch (item.continent) {
      case 'Europe'   : return 'label label-primary';
      case 'America'  : return 'label label-danger label-important';
      case 'Australia': return 'label label-success';
      case 'Africa'   : return 'label label-default';
      case 'Asia'     : return 'label label-warning';
    }
  },
  itemValue: 'value',
  itemText: 'text',
  typeahead: {
    name: 'cities',
    displayKey: 'text',
    source: cities.ttAdapter()
  }
});
elt.tagsinput('add', { "value": 1 , "text": "Amsterdam"   , "continent": "Europe"    });
elt.tagsinput('add', { "value": 4 , "text": "Washington"  , "continent": "America"   });
elt.tagsinput('add', { "value": 7 , "text": "Sydney"      , "continent": "Australia" });
elt.tagsinput('add', { "value": 10, "text": "Beijing"     , "continent": "Asia"      });
elt.tagsinput('add', { "value": 13, "text": "Cairo"       , "continent": "Africa"    });
</script>

```

更多关于选项、方法、事件的内容请参考http://bootstrap-tagsinput.github.io/bootstrap-tagsinput/examples/


##  2、jQuery tag-it 
github:https://github.com/aehlke/tag-it
demo:https://aehlke.github.io/tag-it/
![](/data/dokuwiki/web/pasted/20160505-154111.png)
```

<link rel="stylesheet" type="text/css" href="http://ajax.googleapis.com/ajax/libs/jqueryui/1/themes/flick/jquery-ui.css">
<link href="css/jquery.tagit.css" rel="stylesheet" type="text/css">
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.5.2/jquery.min.js" type="text/javascript" charset="utf-8"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.8.12/jquery-ui.min.js" type="text/javascript" charset="utf-8"></script>
<script src="js/tag-it.js" type="text/javascript" charset="utf-8"></script>

```
  
使用：针对ul标签:
```

<script type="text/javascript">
    $(document).ready(function() {
        $("#myTags").tagit();
    });
</script>

<ul id="myTags">
    <!-- Existing list items will be pre-added to the tags -->
    <li>Tag1</li>
    <li>Tag2</li>
</ul>

```
关于选项、方法、主题、自动完成等请参考:https://github.com/aehlke/tag-it
可以参考:http://www.jb51.net/article/64513.htm