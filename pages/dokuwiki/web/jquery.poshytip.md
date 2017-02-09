title: jquery.poshytip 

#  jQuery.poshytip消息提醒插件 
官网：http://vadikom.com/tools/poshy-tip-jquery-plugin-for-stylish-tooltips/
demo:http://vadikom.com/demos/poshytip
特点支持提示内容样式。
` showOn `属性
Possible Values:`  'hover', 'focus', 'none `'
Handler for showing the tip. Use 'none' if you would like to trigger the tooltip just manually (i.e. by calling the 'show' and 'hide' methods).

**hover效果**:
```

$('#demo-basic').poshytip({
  	//className: 'tip-violet',皮肤
	alignTo: 'target',//布局相对于目标元素
	alignX: 'right',//相对于target水平居右
	alignY: 'center',//相对于target垂直居中
	offsetX: 5});//水平偏移5px
<p><a id="demo-basic" title="Hey, there! This is a <span style='color:red;'>tooltip</span>." href="#">Hover for a <span style="color:red;">tooltip</span></a></p>			

```
![](/data/dokuwiki/web/pasted/20160225-143113.png)

**Click效果**
```

<p><a id="demo-manual-trigger" href="#">This link has a tooltip that is not triggered automatically</a></p>
$('#demo-manual-trigger').poshytip({
	content: 'Hey, there! This is a tooltip.',
	showOn: 'none',
	alignTo: 'target',
	alignX: 'inner-left',
	offsetX: 0,
	offsetY: 5
});
$('#button-show').click(function() { $('#demo-manual-trigger').poshytip('show'); });
$('#button-hide').click(function() { $('#demo-manual-trigger').poshytip('hide'); });

```
参考http://www.cnblogs.com/xiaoyao2011/archive/2011/10/13/2211036.html