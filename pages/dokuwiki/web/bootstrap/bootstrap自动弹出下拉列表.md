title: bootstrap自动弹出下拉列表 

#  bootstrap自动弹出下拉列表 
```

$(document).ready(function(){
	dropdownOpen();//调用
});
/**
 * 鼠标划过就展开子菜单，免得需要点击才能展开
 */
function dropdownOpen() {

	var $dropdownLi = $('li.dropdown');

	$dropdownLi.mouseover(function() {
		$(this).addClass('open');
	}).mouseout(function() {
		$(this).removeClass('open');
	});
}

```