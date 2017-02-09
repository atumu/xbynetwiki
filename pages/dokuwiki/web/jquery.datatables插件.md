title: jquery.datatables插件 

#  jquery.dataTables强大的数据表格插件 
官网：http://www.datatables.net/
插件：http://datatables.net/extensions/index
实例：http://dt.thxopen.com/example/
中文学习站点：http://dt.thxopen.com/ http://www.cnblogs.com/jobs2/p/3431567.html
Datatables是一款jquery表格插件。它是一个高度灵活的工具，可以将任何HTML表格添加高级的交互功能。
  * ` 分页，即时搜索和排序 `
  * 几乎支持任何数据源：DOM， javascript， Ajax 和 服务器处理
  * 支持不同主题 DataTables, jQuery UI, Bootstrap, Foundation
  * 各式各样的扩展: Editor, TableTools, FixedColumns ……
  * 丰富多样的option和强大的API
  * 支持国际化
  * 超过2900+个单元测试
  * 免费开源 （ MIT license ）！

##  安装 
在你的项目中使用datatables，只需要引入两个文件即可，一个js文件和一个css文件，看下面html代码
```

<!-- DataTables CSS -->
<link rel="stylesheet" type="text/css" href="http://cdn.datatables.net/1.10.7/css/jquery.dataTables.css">
 
<!-- jQuery -->
<script type="text/javascript" charset="utf8" src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
 
<!-- DataTables -->
<script type="text/javascript" charset="utf8" src="http://cdn.datatables.net/1.10.7/js/jquery.dataTables.js"></script>

```
```

<table id="table_id" class="display">
    <thead>
        <tr>
            <th>Column 1</th>
            <th>Column 2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Row 1 Data 1</td>
            <td>Row 1 Data 2</td>
        </tr>
        <tr>
            <td>Row 2 Data 1</td>
            <td>Row 2 Data 2</td>
        </tr>
    </tbody>
</table>

```
**` 注意：tbody是必须的 `**

初始化Datatables
```

$(document).ready( function () {
    $('#table_id').DataTable();
} );

```
![](/data/dokuwiki/web/pasted/20151106-145551.png)
##  数据 
数据是复杂的,并且所有的数据是不一样的。因此,datatable中有很多的选项可用于配置如何获得表中的数据显示,以及如何处理这些数据。
本节将讨论datatables处理数据的三个核心概念：
  * 处理模式
  * 数据类型
  * 数据源

###  处理模式(Processing modes) 
datatables中有两种不同的方式处理数据(排序、搜索等):
  * 客户端处理——所有的数据集预先加载和数据处理都是在浏览器中完成的。
  * 服务器端处理——数据处理是在服务器上执行。
每个都有自己的优点和缺点,选择哪种模式是由你的数据量决定的。**根据经验来看,数据少于10000行你可以选择客户端处理,超过100000行的使用服务器端处理。**请注意,两种处理模式不能同时使用,但是可以动态更改从一个模式到另一个。
**服务器模式**需要启用` serverSide `属性，完整的介绍请参考手册

###  数据源类型(Data source types) 
datatables可以使用三种基本的javascript数据类型来作为数据源：
  * 数组(Arrays [])
  * 对象(objects {})
  * 实例(new myclass())
datatables可以用` colums.data 或者colums.render `选项来显示数据，**默认操作模式是数组**，对象和实例能直观地处理更复杂的数据

####  数组(Arrays) 
数组在DataTable中很容易使用当使用数组作为数据源**,每个数组元素的数量必须等于表中的列数**。例如,对于一个6列的表您如下:
```

var data = [
    [
        "Tiger Nixon",
        "System Architect",
        "Edinburgh",
        "5421",
        "2011/04/25",
        "$3,120"
    ],
    [
        "Garrett Winters",
        "Director",
        "Edinburgh",
        "8422",
        "2011/07/25",
        "$5,300"
    ]
]

```
然后datatables这样初始化：
```

$('#example').DataTable( {
    data: data
} );

```

####  对象(Objects) 
对象看起来很直观，使用起来和数组略有不同。如果你已经参考了api，你可以知道通过对象获得特定的数据非常简单，你只需要使用一个属性的名字，而不是记住这个数组的索引，比如data.name，而不是data[0]<
根据表格的需求显示，**对象可以包含更多的信息**,例如包括数据库的主键而用户是看不见的.
**使用对象前，你需要明确告诉datatables那个属性对应那一列， 通过使用` columns.data 或者 columns.render ` 选项完成。**
下面看看object是个什么样的格式：
```

[
    {
        "name":       "Tiger Nixon",
        "position":   "System Architect",
        "salary":     "$3,120",
        "start_date": "2011/04/25",
        "office":     "Edinburgh",
        "extn":       "5421"
    },
    {
        "name":       "Garrett Winters",
        "position":   "Director",
        "salary":     "$5,300",
        "start_date": "2011/07/25",
        "office":     "Edinburgh",
        "extn":       "8422"
    }
]

```
```

$('#example').DataTable( {
    data: data,
    columns: [
        { data: 'name' },
        { data: 'position' },
        { data: 'salary' },
        { data: 'office' }
    ]
} );

```

####  实例(Instances) 
datatables从实例中获取数据显示是非常有用的，这些实例可以定义成抽象的方法来更新数据。
```

function Employee ( name, position, salary, office ) {
    this.name = name;
    this.position = position;
    this.salary = salary;
    this._office = office;
 
    this.office = function () {
        return this._office;
    }
};
 
$('#example').DataTable( {
    data: [
        new Employee( "Tiger Nixon", "System Architect", "$3,120", "Edinburgh" ),
        new Employee( "Garrett Winters", "Director", "$5,300", "Edinburgh" )
    ],
    columns: [
        { data: 'name' },
        { data: 'salary' },
        { data: 'office()' },
        { data: 'position' }
    ]
} );

```
注意,name,salary,position是属性而office是一个方法，datatables允许这样使用，他会自动识别

##  Data sources 
datatables支持三种数据源显示：
  * dom
  * javascript
  * ajax

DOM
datatables初始化后，它会自动检查表格中的数据，如果存在即作为表显示的数据（注意，如果你这时使用data或者ajax传递数据将不会显示），这是使用datatables最简单的方法。
注意,当使用DOM显示表,datatables将使用数组作为数据源(见上)。

HTML5
datatables中还可以利用**HTML5 data- *属性**,可以提供datatables中**排序和搜索数据**的附加信息。例如您可能有一个列是一个日期格式,如“21st November 2013”，浏览器将难以排序,但是你可以提供一个` data-order属性 `作为HTML的一部分包含一个时间戳,就可以很容易地解决。此外,可以使用` data-search `搜索数据。例如:
```

<td data-search="21st November 2013 21/11/2013" data-order="1384992000">
    21st November 2013
</td>

```
datatable中会自动检测:
  * 排序数据: ` data-order 和 data-sort ` 属性
  * 查找数据: ` data-search 和 data-filter ` 属性

Javascript
你可以指定datatables使用哪一种数据作为初始化，这些数据可以是数组，对象或者实例(见上)，只要javascript可以访问到数据就可以交给datatables显示。
查看datatable的api，使用` row.add()和row.remove() `方法可以**动态添加删除表格中的数据**

Ajax
ajax和javascript数据很类似，**你只需要指定要加载的数据的url即可。**
服务器端处理是一种特殊的数据源，**每页的数据通过异步请求来显示相应的数据**，这允许大量的数据集显示，怎么实现服务器处理，详细参考手册

##  选项配置 
datatables中大量的选项可以用来定制你的表格展现给用户。

**设置选项(Setting options)**
datatables的配置是通过设置你定义的选项来完成的，如下：
```

$('#example').DataTable( {
    paging: false
} );

```
通过设置paging选项，禁止表格分页(默认是打开的)
假设你要在表格里使用滚动，你需要加上scrollY选项：
```

$('#example').DataTable( {
    scrollY: 400
} );

```
当然你可以组合多个选项来初始化datatables，启动滚动条，禁用分页
```

$('#example').DataTable( {
    paging: false,
    scrollY: 400
} );

```
如果你有其他需要可以加入更多的选项来配置你的表格，更多datatables选项，请参考

**常用选项(Common options)**
下面列举了一些比较有用的选项:
  * ajax - 异步数据源配置
  * data - javascript数据源配置
  * serverSide - 开启服务器模式
  * columns.data - 配置每一列的数据源
  * scrollX - 水平滚动条
  * scrollY - 垂直滚动条
**全局默认设置(Setting defaults)**
在实际项目中，可能需要用到多个表格，你使用dom选项把所有的表格设置为相同的布局，这时你可以使用` $.fn.dataTable.defaults ` **对象处理。**
在这个例子中,我们禁用datatable中默认的搜索和排序功能,但当表初始化时启用了排序(重写默认选项)。
```

// 默认禁用搜索和排序
$.extend( $.fn.dataTable.defaults, {
    searching: false,
    ordering:  false
} );
 
// 这样初始化，排序将会打开
// 搜索功能仍然是关闭
$('#example').DataTable( {
    ordering: true
} );

```

###  完整选项列表 
**` 请参考： `**http://dt.thxopen.com/reference/option/
##  样式 
对于网站来说，风格设计是很重要的。Datatables为此做了考虑，提供了许多样式表可以无缝的和你的网站融合在一起。Datatables有自己一套默认的样式，**同时也集成了当前比较流行的前端css框架，比如bootstrap。**
Bootstrap是Twitter推出的一个用于前端开发的开源工具包。它由Twitter的设计师Mark Otto和Jacob Thornton合作开发，是一个CSS/HTML框架。 目前，Bootstrap最新版本为3.0 。
**Datatables已经很好的支持Bootstrap，**通过下面的示例展示给大家怎么使用Bootstrap来美化Datatables
在早起的版本中应用Bootstrap到Datatables稍微麻烦，但是现在新版已经做了很好的改善，你只需要引入Datatables已经写好的css样式和js即可
css: http://cdn.datatables.net/plug-ins/28e7751dbec/integration/bootstrap/3/dataTables.bootstrap.css
js: http://cdn.datatables.net/plug-ins/28e7751dbec/integration/bootstrap/3/dataTables.bootstrap.js
你也可以在github上clone下来 https://github.com/DataTables/Plugins/tree/master/integration/bootstrap

##  事件 
DataTables provides three methods for working with DataTables events, matching the core jQuery event methods:
  * on() - Listen for events
  * off() - Stop listening for events
  * one() - Listen for a single event.
function on( event, callback )
```

var table = $('#example').DataTable();
table.on( 'draw', function () {
    alert( 'Table redrawn' );
} );

var table = $('#example').DataTable( {
    ajax: "/data.json"
} );
 
table.on( 'xhr', function ( e, settings, json ) {
    console.log( 'Ajax event occurred. Returned data: ', json );
} );

```
###  事件类型 
` **事件类型与回调函数传参请参考** `：http://datatables.net/reference/event/ 和http://dt.thxopen.com/reference/event/

![](/data/dokuwiki/web/pasted/20151106-154309.png)![](/data/dokuwiki/web/pasted/20151106-154329.png)

##  服务器处理(Server-side processing) 
是不是发现在处理太多dom数据或者ajax一次性把数据获得后，Datatables表现的不是很满意？这是肯定的，因为dt需要渲染，创建tr，td所以数据越多，速度就越慢了。 为了解决这个问题Datatables提供了服务器模式，把本来客户端所做的事情交给服务器去处理，比如 排序，分页，过滤。对于客户端来说这些操作都是比较消耗资源的， 所以打开服务器模式后有再多的数据也不用怕了

**当你打开服务器模式的时候，每次绘制表格的时候，Datatables会给服务器发送一个请求（包括当前分页，排序，搜索等等）。Datatables会发向 服务器发送大量的变量去执行所需要的处理，然后在服务器拼好相应的数据返回给Datatables。**
**开启服务器模式**需要使用 ` serverSideDT ` 选项，并且配置`  ajaxDT ` ，进一步的信息，请参考下面的配置选项。

###  发送参数(Sent parameters) 
当使用服务器处理时，Datatables会发送如下数据给服务器
![](/data/dokuwiki/web/pasted/20151106-155401.png)
###  返回的数据(Returned data) 
Datatables发送以上参数后需要按照下面要求的参数返回
![](/data/dokuwiki/web/pasted/20151106-155431.png)
除了上面的返回参数以外你还可以加入下面的参数，来实现对表格的自动绑定
![](/data/dokuwiki/web/pasted/20151106-155500.png)
###  配置（Configuration） 
**使用服务器模式**需要启用 ` serverSide `DT 选项 ，设置为 true，**并且配置 ajaxDT 设置url，告诉Datatables该 从那里获得数据**
所以最简单的服务器初始化代码如下所示：
```

$('#example').DataTable( {
    serverSide: true,
    ajax: '/data-source'
} );

```
ajaxDT 可以直接接受一个字符串也可以作为一个对象使用。作为对象他就像 jQuery.ajax 配置一样
比如我要Datatables发送的请求为 post代码如下
```

$('#example').DataTable( {
    serverSide: true,
    ajax: {
        url: '/data-source',
        type: 'POST'
    }
} );

```
在Datatables中的 ajax选项配置详细参考 ajaxDT
###  示例数据（Example data） 
服务器端处理的例子，返回使用数组作为数据源 。示列：http://dt.thxopen.com/example/server_side/simple.html
```

$(document).ready(function() {
    $('#example').dataTable( {
        "processing": true,
        "serverSide": true,
        "ajax": "../resources/server_processing_custom.php"
    } );
} );

```
```

{
    "draw": 1,
    "recordsTotal": 57,
    "recordsFiltered": 57,
    "data": [
        [
            "Angelica",
            "Ramos",
            "System Architect",
            "London",
            "9th Oct 09",
            "$2,875"
        ],
        [
            "Ashton",
            "Cox",
            "Technical Author",
            "San Francisco",
            "12th Jan 09",
            "$4,800"
        ],
        ...
    ]
}

```
服务器端处理的例子,返回使用对象还包括DT_RowId 和 DT_RowData。示列：http://datatables.net/examples/server_side/object_data.html
```

$(document).ready(function() {
    $('#example').DataTable( {
        "processing": true,
        "serverSide": true,
        "ajax": "scripts/objects.php",
        "columns": [
            { "data": "first_name" },
            { "data": "last_name" },
            { "data": "position" },
            { "data": "office" },
            { "data": "start_date" },
            { "data": "salary" }
        ]
    } );
} );

```
```

{
    "draw": 1,
    "recordsTotal": 57,
    "recordsFiltered": 57,
    "data": [
        {
            "DT_RowId": "row_3",
            "DT_RowData": {
                "pkey": 3
            },
            "first_name": "Angelica",
            "last_name": "Ramos",
            "position": "System Architect",
            "office": "London",
            "start_date": "9th Oct 09",
            "salary": "$2,875"
        },
        {
            "DT_RowId": "row_17",
            "DT_RowData": {
                "pkey": 17
            },
            "first_name": "Ashton",
            "last_name": "Cox",
            "position": "Technical Author",
            "office": "San Francisco",
            "start_date": "12th Jan 09",
            "salary": "$4,800"
        },
        ...
    ]
}

```
##  国际化（Internationalisation） 
Datatables默认的是英语，但是可以很容翻译成其他的语言。 
由社区提供的超过50多种语言，让你在Datatables中使用
**配置（Configuration）**
Datatables中所使用的语言选项是通过 language 来配置的。 这是一个对象字符串，通过一个参数来描述Datatables的每个部分。 语言选项的完整参数可以参考 language 文档。
在Datatables中，语言选项的配置与 其他配置 方式完全相同，作为初始化设置的一部分，这个例子展示了如何修改搜索字符串:
```

  $('#example').DataTable( {
    language: {
        search: "在表格中搜索:"
    }
} );

```
与其他初始化选项一样，你可以按你所希望的改变尽可能多或尽可能少的选项。那些选项你不赋值，Datatables会使用默认值， 本示例显示每个语言选项，用中文显示Datatables接口：
```

$('#example').DataTable({
    language: {
        "sProcessing": "处理中...",
        "sLengthMenu": "显示 _MENU_ 项结果",
        "sZeroRecords": "没有匹配结果",
        "sInfo": "显示第 _START_ 至 _END_ 项结果，共 _TOTAL_ 项",
        "sInfoEmpty": "显示第 0 至 0 项结果，共 0 项",
        "sInfoFiltered": "(由 _MAX_ 项结果过滤)",
        "sInfoPostFix": "",
        "sSearch": "搜索:",
        "sUrl": "",
        "sEmptyTable": "表中数据为空",
        "sLoadingRecords": "载入中...",
        "sInfoThousands": ",",
        "oPaginate": {
            "sFirst": "首页",
            "sPrevious": "上页",
            "sNext": "下页",
            "sLast": "末页"
        },
        "oAria": {
            "sSortAscending": ": 以升序排列此列",
            "sSortDescending": ": 以降序排列此列"
        }
    }
});

```
**异步加载翻译（Ajax loading a translation）**
为了方便起见，DataTables提供一个选项，**用于从服务器Ajax加载语言信息**。这个通过 language.url 进行配置，例如：
中文CDN：http://cdn.datatables.net/plug-ins/1.10.9/i18n/Chinese.json
```

$('#example').DataTable({
    language: {
        url: 'http://cdn.datatables.net/plug-ins/1.10.9/i18n/Chinese.json'
    }
});

```

**可用的翻译(Available translations)**
DataTables社区共同递交了超过50个翻译，可以直接在Datatables中使用。用法参考 [plug-ins](http://datatables.net/plug-ins/i18n/)。

##  API列表 
**` 请参考： `**http://dt.thxopen.com/reference/api/

更多参考资料:http://www.cnblogs.com/sheldon-lou/p/4169902.html
http://www.cnblogs.com/sheldon-lou/p/4179002.html