title: jquery.knob 

#  jquery.knob旋钮插件 
官网：http://anthonyterrien.com/knob/
github:https://github.com/aterrien/jQuery-Knob
主要特性
  * 支持只读模式
  * 两个供选择的callback方法：` change和release `
  * 支持自定义选项并且支持使用HTML5的**data属性**来配置插件选项
  * 内建不同的主题
  * 对于老的浏览器拥有不错的fallback机制
导入jQuery和knob插件类库：
```

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.0/jquery.min.js"></script>
<script src="js/jquery.knob-1.0.1.js"></script>

```
设定参数和callback方法：
```

$(".knob").knob({
    max: 940,
    min: 500,
    thickness: .3,
    fgColor: '#2B99E6',
    bgColor: '#303030',
    'release':function(value){
        $('#img').animate({width:value});
    }
});

```
当然，你也可以使用HTML5的标签属性来设置参数，如下：
```

<input class="knob2" data-width="150" data-fgColor="green" data-bgColor="#303030" data-skin="tron" data-thickness=".3" data-min="200" data-max="600" value="200">

```
参考：http://www.jq22.com/jquery-info392