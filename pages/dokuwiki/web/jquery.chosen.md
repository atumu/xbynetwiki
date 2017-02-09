title: jquery.chosen 

#  jquery.chosen下拉多选框 
github:https://github.com/harvesthq/chosen
jQuery Chosen Plugin 是一个 jQuery 插件，用来将网页中的下拉框进行功能扩展和美化。可实现对下拉框的搜索，多个标签编辑等功能，如下图所示：
![](/data/dokuwiki/web/pasted/20151106-163242.png)
Chosen 是一个支持jquery的select下拉框美化插件，它能让丑陋的、很长的select选择框变的更好看、更方便。不仅如此，它更扩展了select，增加了自动筛选的功能。它可对列表进行分组，同时也可禁用某些选择项。
1、先把js和css文件引用到网页里面去：
```

<link href="js/jqueryUI/chosen/chosen.css" type="text/css" rel="stylesheet" />
<script type="text/javascript" src="js/jquery.1.4.4.min.js"></script>
<script type="text/javascript" src="js/jqueryUI/chosen/chosen.jquery.js"></script>

```
2、创建一个select元素，如下： 
```

<select name="dept" style="width:
 150px;" id="dept" class="dept_select"> 
    <option value="部门1">部门1</option>
    <option value="部门2">部门2</option>
    <option value="部门3">部门3</option>
    <option value="部门4">部门4</option>
    <option value="部门5">部门5</option>
</select>

```
3、然后在js中调用Chosen定义的方法：
```

$(function(){
    $('.dept_select').chosen();
});

```
4、搞定收工，屌丝立马变成高富帅有木有~ 
##  chosen插件的一些设置项: 
1、默认文字选项
你可以在select元素上添加` data-placeholder `属性定义默认文字，也就是在没有选择选项的情况下，显示的文字。 
```

<select data-placeholder="选择部门" style="width:150px;" class="dept_select">
    <option value="-1"></option>
    <option value="部门1">部门1</option>
    <option value="部门2">部门2</option>
    <option value="部门3">部门3</option>
    <option value="部门4">部门4</option>
    <option value="部门5">部门5</option>
</select>

```
这里还要注意一点，**要想显示出默认文字，select下的第一个选择项必须为空的option。**

2、对其方式
选项文字默认是左对齐的，可以在class属性中加入“chzn-rtl”来设置右对齐： 
```

<select data-placeholder="选择部门" class="dept_select
 chzn-rtl" style="width:150px;">

```
3、JS参数设置
在调用chosen()方法时，我们可以设置一些参数： 
no_results_text	无搜索结果显示的文本
allow_single_deselect	是否允许取消选择
max_selected_options	当select为多选时，最多选择个数
```

$(".some_select").chosen({
    /*max_selected_options:
 2,*/
    no_results_text:
"没有找到",
    allow_single_deselect:
true
});

```

**4、事件**
　　a) change事件：
```

$(".dept-select").chosen().change(function(){
    //do
 something...
});

```
　　b) 当我们需要**动态更新select下的选择项时**，该怎么办呢？只要在更新选择项后触发Chosen中的` chosen:updated `事件就可以了：　　 
```

//...$(".dept-select").html('...<option>部门6</option>...');
$(".dept-select").trigger("chosen:updated");

```

其他问题：
1、如果不想要搜索框的话，很简单，用css把它隐藏掉就OK了：
```

.chzn-container-single
 .chzn-search {
    display:
none;
}

```

参考：http://blog.csdn.net/iamduoluo/article/details/11519909