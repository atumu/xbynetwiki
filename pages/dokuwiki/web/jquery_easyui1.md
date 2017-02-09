title: jquery_easyui1 

#  jquery easyui入门 
官网：http://www.jeasyui.com/
中文文档:http://www.jeasyui.net/
Demo:http://www.jeasyui.com/demo/main/index.php
API:http://www.jeasyui.com/documentation/index.php

easyui是一种基于jQuery的用户界面插件集合。easyui为创建现代化，互动，JavaScript应用程序，提供必要的功能。使用easyui你不需要写很多代码，你只需要通过编写一些简单HTML标记，就可以定义用户界面。
easyui是个完美支持HTML5网页的完整框架。easyui节省您网页开发的时间和规模。easyui很简单但功能强大的。
在` jQuery ` 和 ` HTML5 ` 上轻松使用` EasyUI `

##  两个方法声明的 UI 组件: 
1. 直接在 HTML 声明组件。
```

<div class="easyui-dialog" style="width:400px;height:200px"
    data-options="title:'My Dialog',collapsible:true,iconCls:'icon-ok',onOpen:function(){}">
        dialog content.
</div>

```
2. 编写 JavaScript 代码来创建组件。
```

<input id="cc" style="width:200px" />

$('#cc').combobox({
	url: ...,
	required: true,
	valueField: 'id',
	textField: 'text'
});

```
所有的 EasyUI 插件
jQuery easyui提供了一个完整的组件的集合，包括强大的DataGrid，树网格，面板。用户可以使用他们一起，或者只是用一些组件，组合和构建他想要的跨浏览器的网页应用。
![](/data/dokuwiki/web/pasted/20150922-061559.png)
##  Jquery EasyUI组件 
easyui 的每个组件都有**属性、方法和事件**。用户可以很容易地对这些组件**进行扩展**。
  * 属性：属性是定义在 ` jQuery.fn.{plugin}.defaults `。比如，dialog 的属性是定义在 jQuery.fn.dialog.defaults。
  * 事件：事件（回调函数）也是定义在 ` jQuery.fn.{plugin}.defaults `。
  * 方法：调用方法的语法：` $('selector').plugin('method', parameter); `
其中：
  - selector 是 jquery 对象选择器。
  - plugin 是插件名称。
  - method 是与插件相匹配的已存在方法。
  - parameter 是参数对象，可以是对象、字符串...

##  开始使用 jQuery EasyUI 
下载库，并在您的页面中引用 EasyUI CSS 和 JavaScript 文件。你也可以指定使用的主题和语言版本：
<blockquote><link rel="stylesheet" type="text/css" href="easyui/themes` /bootstrap `/easyui.css"> <!-- 使用bootstrap主题 -->
<link rel="stylesheet" type="text/css" href="easyui/themes/icon.css"> <!-- 图标样式-->
<script type="text/javascript" src="easyui/` jquery-1.7.2.min.js `"></script>  <!--依赖jquery-->
<script type="text/javascript" src="easyui/jquery.easyui.min.js"></script>  <!--easyui主js-->
<script type="text/javascript" src="easyui/` locale/easyui-lang-zh_CN.js `"></script>  <!--使用中文语言包-->
</blockquote>
一旦您引用了 EasyUI 必要的文件，您就可以通过标记或者使用 JavaScript 来定义一个 EasyUI 组件。**比如，要顶一个带有可折叠功能的面板**，您需要编写如下 HTML 代码：
```

<div id="p" class="easyui-panel" style="width:500px;height:200px;padding:10px;"
    title="My Panel" iconCls="icon-save" collapsible="true">
    The panel content
</div>

```
当通过标记创建组件，` 'data-options `' 属性被用来**支持自版本 1.3 以来 HTML5 兼容的属性名称**。所以您可以如下**重写上面的代码**：
```

<div id="p" class="easyui-panel" style="width:500px;height:200px;padding:10px;"
    title="My Panel" data-options="iconCls:'icon-save',collapsible:true">
    The panel content
</div>

```
下面的代码演示了如何创建一个绑定 'onSelect' 事件的组合框。
```

<input class="easyui-combobox" name="language"
    data-options="
    url:'combobox_data.json',
    valueField:'id',
    textField:'text',
    panelHeight:'auto',
    onSelect:function(record){
    alert(record.text)
    }">

```