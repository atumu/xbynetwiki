title: 全局css样式 

#  Bootstrap全局CSS样式 
##  排版与链接 
![](/data/dokuwiki/web/bootstrap/pasted/20150613-122148.png)
##  布局容器 
Bootstrap 需要为页面内容和栅格系统包裹一个 ` .container ` 容器。我们提供了两个作此用处的类。注意，由于 padding 等属性的原因，**这两种 容器类不能互相嵌套**。
` .container ` 类用于**固定宽度**并支持响应式布局的容器。
```

<div class="container">
  ...
</div>

```
` .container-fluid ` 类用于 **100% 宽度**，占据全部视口（viewport）的容器。
```

<div class="container-fluid">
  ...
</div>

```
##  栅格系统 
Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，**系统会自动分为最多12列**。它包含了易于使用的预定义类，还有强大的mixin 用于生成更具语义的布局。
**栅格系统用于通过一系列的行（row）与列（column）的组合来创建页面布局**，你的内容就可以放入这些创建好的布局中。下面就介绍一下 Bootstrap 栅格系统的工作原理：
  * “行（row）”必须包含在 .container （固定宽度）或 .container-fluid （100% 宽度）中，以便为其赋予合适的排列（aligment）和内补（padding）。
  * 通过“行（row）”在水平方向创建一组“列（column）”。
  * 你的内容应当放置于“列（column）”内，并且，只有“列（column）”可以作为行（row）”的直接子元素。
  * 类似 .row 和 .col-xs-4 这种预定义的类，可以用来快速创建栅格布局。Bootstrap 源码中定义的 mixin 也可以用来创建语义化的布局。
  * 通过为“列（column）”设置 padding 属性，从而创建列与列之间的间隔（gutter）。通过为 .row 元素设置负值 margin 从而抵消掉为 .container 元素设置的 padding，也就间接为“行（row）”所包含的“列（column）”抵消掉了padding。
  * 负值的 margin就是下面的示例为什么是向外突出的原因。在栅格列中的内容排成一行。
  * 栅格系统中的列是通过指定1到12的值来表示其跨越的范围。例如，三个等宽的列可以使用三个 .col-xs-4 来创建。
  * 如果一“行（row）”中包含了的“列（column）”大于 12，多余的“列（column）”所在的元素将被作为一个整体另起一行排列。
  * 栅格类适用于与屏幕宽度大于或等于分界点大小的设备 ， 并且针对小屏幕设备覆盖栅格类。 因此，在元素上应用任何 .col-md-* 栅格类适用于与屏幕宽度大于或等于分界点大小的设备 ， 并且针对小屏幕设备覆盖栅格类。 因此，在元素上应用任何 .col-lg-* 不存在， 也影响大屏幕设备。

##  媒体查询 

在栅格系统中，我们在 Less 文件中使用以下媒体查询（media query）来创建关键的分界点阈值。

/* 超小屏幕（手机，小于 768px） */
/* 没有任何媒体查询相关的代码，因为这在 Bootstrap 中是默认的（还记得 Bootstrap 是移动设备优先的吗？） */

/* 小屏幕（平板，大于等于 768px） */
@media (min-width: @screen-sm-min) { ... }

/* 中等屏幕（桌面显示器，大于等于 992px） */
@media (min-width: @screen-md-min) { ... }

/* 大屏幕（大桌面显示器，大于等于 1200px） */
@media (min-width: @screen-lg-min) { ... }
我们偶尔也会在媒体查询代码中包含 max-width 从而将 CSS 的影响限制在更小范围的屏幕大小之内。
@media (max-width: @screen-xs-max) { ... }
@media (min-width: @screen-sm-min) and (max-width: @screen-sm-max) { ... }
@media (min-width: @screen-md-min) and (max-width: @screen-md-max) { ... }
@media (min-width: @screen-lg-min) { ... }
###  栅格参数 
![](/data/dokuwiki/web/bootstrap/pasted/20150613-122954.png)
###  实例：从堆叠到水平排列 
使用单一的一组 .col-md-* 栅格类，就可以创建一个基本的栅格系统，在手机和平板设备上一开始是堆叠在一起的（超小屏幕到小屏幕这一范围），在桌面（中等）屏幕设备上变为水平排列。**所有“列（column）必须放在 ” .row 内。**
![](/data/dokuwiki/web/bootstrap/pasted/20150613-123129.png)
```

<div class="row">
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
</div>
<div class="row">
  <div class="col-md-8">.col-md-8</div>
  <div class="col-md-4">.col-md-4</div>
</div>
<div class="row">
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4">.col-md-4</div>
</div>
<div class="row">
  <div class="col-md-6">.col-md-6</div>
  <div class="col-md-6">.col-md-6</div>
</div>

```
###  实例：流式布局容器 
将最外面的布局元素`  .container 修改为 .container-fluid `，就可以将固定宽度的栅格布局转换为 100% 宽度的布局。
```

<div class="container-fluid">
  <div class="row">
    ...
  </div>
</div>

```
###  实例：移动设备和桌面屏幕 
是否不希望在小屏幕设备上所有列都堆叠在一起？那就使用针对超小屏幕和中等屏幕设备所定义的类吧，即 .col-xs-* 和 .col-md-*。请看下面的实例，研究一下这些是如何工作的。
```

<!-- Stack the columns on mobile by making one full-width and the other half-width -->
<div class="row">
  <div class="col-xs-12 col-md-8">.col-xs-12 .col-md-8</div>
  <div class="col-xs-6 col-md-4">.col-xs-6 .col-md-4</div>
</div>

<!-- Columns start at 50% wide on mobile and bump up to 33.3% wide on desktop -->
<div class="row">
  <div class="col-xs-6 col-md-4">.col-xs-6 .col-md-4</div>
  <div class="col-xs-6 col-md-4">.col-xs-6 .col-md-4</div>
  <div class="col-xs-6 col-md-4">.col-xs-6 .col-md-4</div>
</div>

<!-- Columns are always 50% wide, on mobile and desktop -->
<div class="row">
  <div class="col-xs-6">.col-xs-6</div>
  <div class="col-xs-6">.col-xs-6</div>
</div>

```
###  实例：多余的列（column）将另起一行排列 
**如果在一个 .row 内包含的列（column）大于12个**，包含多余列（column）的元素将作为一个整体单元被另起一行排列。
![](/data/dokuwiki/web/bootstrap/pasted/20150613-123508.png)
```

<div class="row">
  <div class="col-xs-9">.col-xs-9</div>
  <div class="col-xs-4">.col-xs-4<br>Since 9 + 4 = 13 &gt; 12, this 4-column-wide div gets wrapped onto a new line as one contiguous unit.</div>
  <div class="col-xs-6">.col-xs-6<br>Subsequent columns continue along the new line.</div>
</div>

```
###  列重置列偏移嵌套列列排序 
略。。。
##  Less mixin 和变量 
除了用于快速布局的预定义栅格类，Bootstrap 还包含了一组 Less 变量和 mixin 用于帮你生成简单、语义化的布局。
**变量**
通过变量来定义列数、槽（gutter）宽、媒体查询阈值（用于确定合适让列浮动）。我们使用这些变量生成预定义的栅格类，如上所示，还有如下所示的定制 mixin。
@grid-columns:              12;
@grid-gutter-width:         30px;
@grid-float-breakpoint:     768px;
**mixin**
mixin 用来和栅格变量一同使用，为每个列（column）生成语义化的 CSS 代码。
##  排版 
**标题**
HTML 中的所有标题标签，<h1> 到 <h6> 均可使用。另外，还提供了 .h1 到 .h6 类，为的是给内联（inline）属性的文本赋予标题的样式。
在标题内还可以包含 <small> 标签或赋予 .small 类的元素，可以用来标记副标题。
**页面主体**
**Bootstrap 将全局 font-size 设置为 14px，line-height 设置为 1.428。这些属性直接赋予 <body> 元素和所有段落元素。**另外，<p> （段落）元素还被设置了等于 1/2 行高（即 10px）的底部外边距（margin）。
**中心内容**
通过添加 ` .lead ` 类可以让段落**突出显示**。
` <p class="lead">...</p> `
**使用 Less 工具构建**
` variables.less ` 文件中定义的两个 Less 变量决定了排版尺寸：` @font-size-base 和 @line-height-base `。第一个变量定义了全局 font-size 基准，第二个变量是 line-height 基准。我们使用这些变量和一些简单的公式计算出其它所有页面元素的 margin、 padding 和 line-height。自定义这些变量即可改变 Bootstrap 的默认样式。
**内联文本元素**
高亮显示：` You can use the mark tag to <mark>highlight</mark> text. `
被删除的文本：` <del>This line of text is meant to be treated as deleted text.</del> `
无用文本：<s>This line of text is meant to be treated as no longer accurate.</s>
插入文本：<ins>This line of text is meant to be treated as an addition to the document.</ins>
带下划线的文本：<u>This line of text will render as underlined</u>
小号文本：<small>This line of text is meant to be treated as fine print.</small>
着重：<strong>rendered as bold text</strong>
斜体：<em>rendered as italicized text</em>
对齐：
```

<p class="text-left">Left aligned text.</p>
<p class="text-center">Center aligned text.</p>
<p class="text-right">Right aligned text.</p>
<p class="text-justify">Justified text.</p>
<p class="text-nowrap">No wrap text.</p>

```
改变大小写:
<p class="text-lowercase">Lowercased text.</p>
<p class="text-uppercase">Uppercased text.</p>
<p class="text-capitalize">Capitalized text.</p>
缩略语:
当鼠标悬停在缩写和缩写词上时就会显示完整内容，Bootstrap 实现了对 HTML 的 <abbr> 元素的增强样式。缩略语元素带有 title 属性
<abbr title="attribute">attr</abbr>
地址:
```

<address>
  <strong>Twitter, Inc.</strong><br>
  795 Folsom Ave, Suite 600<br>
  San Francisco, CA 94107<br>
  <abbr title="Phone">P:</abbr> (123) 456-7890
</address>

```
引用:
```

<blockquote>
<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Integer posuere erat a ante.</p>
</blockquote>

```
对于标准样式的 ` <blockquote> `，可以通过几个简单的变体就能改变风格和内容。
命名来源
添加 ` <footer> ` 用于标明引用来源。来源的名称可以包裹进<    cite     >标签中。
```

<blockquote>
<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Integer posuere erat a ante.</p>
<footer>Someone famous in <cite title="Source Title">Source Title</cite></footer>
</blockquote>

```
通过赋予 .blockquote-reverse 类可以让引用呈现内容右对齐的效果。
```

<blockquote class="blockquote-reverse">
...
</blockquote>

```
列表
```

<ul>
<li>...</li>
</ul>

<ol>
<li>...</li>
</ol>

```
内联列表
通过设置 display: inline-block; 并添加少量的内补（padding），将所有元素放置于**同一行**。
<ul class="list-inline">
<li>...</li>
</ul>
描述:带有描述的短语列表。
<dl>
<dt>...</dt>
<dd>...</dd>
</dl>
##  代码 
内联代码:通过 ```
 标签包裹内联样式的代码片段。For example, <code>&lt;section&gt;</code> should be wrapped as inline.
代码块:多行代码可以使用 <pre> 标签。为了正确的展示代码，注意将尖括号做转义处理。
<pre>&lt;p&gt;Sample text here...&lt;/p&gt;</pre>
还可以使用`  .pre-scrollable ` 类，其作用是**设置 max-height 为 350px ，并在垂直方向展示滚动条。**
##  表格 
<table class="table">
 ...
</table>
条纹状表格:通过 ` .table-striped ` 类可以给 <tbody> 之内的每一行增加斑马条纹样式。
<table class="table table-striped">
...
</table>
带边框的表格:添加`  .table-bordered ` 类为表格和其中的每个单元格增加边框。
<table class="table table-bordered">
...
</table>
鼠标悬停:通过添加`  .table-hover ` 类可以让 <tbody> 中的每一行对鼠标悬停状态作出响应。
<table class="table table-hover">
...
</table>
状态类:通过这些状态类可以为行或单元格设置颜色。

Class	描述
.active	鼠标悬停在行或单元格上时所设置的颜色
.success	标识成功或积极的动作
.info	标识普通的提示信息或动作
.warning	标识警告或需要用户注意
.danger	标识危险或潜在的带来负面影响的动作
![](/data/dokuwiki/web/bootstrap/pasted/20150613-143239.png)
<code xml>
<!-- On rows -->
<tr class="active">...</tr>
<tr class="success">...</tr>
<tr class="warning">...</tr>
<tr class="danger">...</tr>
<tr class="info">...</tr>

<!-- On cells (`td` or `th`) -->
<tr>
  <td class="active">...</td>
  <td class="success">...</td>
  <td class="warning">...</td>
  <td class="danger">...</td>
  <td class="info">...</td>
</tr>

```
响应式表格:**将任何 .table 元素包裹在 .table-responsive 元素内，即可创建响应式表格，**其会在小屏幕设备上（小于768px）水平滚动。当屏幕大于 768px 宽度时，水平滚动条消失。
```

<div class="table-responsive">
  <table class="table">
    ...
  </table>
</div>

```
##  表单 
单独的表单控件会被自动赋予一些全局样式。所有设置了`  .form-control ` 类的 <input>、<textarea> 和 <select> 元素都将被默认设置宽度属性为 width: 100%;。 将 label 元素和前面提到的控件包裹在 ` .form-group ` 中可以获得最好的排列。
```

<form>
  <div class="form-group">
    <label for="exampleInputEmail1">Email address</label>
    <input type="email" class="form-control" id="exampleInputEmail1" placeholder="Enter email">
  </div>
  <div class="form-group">
    <label for="exampleInputPassword1">Password</label>
    <input type="password" class="form-control" id="exampleInputPassword1" placeholder="Password">
  </div>
  <div class="form-group">
    <label for="exampleInputFile">File input</label>
    <input type="file" id="exampleInputFile">
    <p class="help-block">Example block-level help text here.</p>
  </div>
  <div class="checkbox">
    <label>
      <input type="checkbox"> Check me out
    </label>
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
</form>

```
**内联表单**:为 <form> 元素添加 .form-inline 类可使其内容左对齐并且表现为 inline-block 级别的控件。只适用于视口（viewport）至少在 768px 宽度时（视口宽度再小的话就会使表单折叠）。
**水平排列的表单**:通过为表单添加`  .form-horizontal ` 类，并联合使用 Bootstrap 预置的栅格类，可以将 label 标签和控件组水平并排布局。这样做将改变 .form-group 的行为，使其表现为栅格系统中的行（row），因此就无需再额外添加 .row 了。
```

<form class="form-horizontal">
  <div class="form-group">
    <label for="inputEmail3" class="col-sm-2 control-label">Email</label>
    <div class="col-sm-10">
      <input type="email" class="form-control" id="inputEmail3" placeholder="Email">
    </div>
  </div>
  <div class="form-group">
    <label for="inputPassword3" class="col-sm-2 control-label">Password</label>
    <div class="col-sm-10">
      <input type="password" class="form-control" id="inputPassword3" placeholder="Password">
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-10">
      <div class="checkbox">
        <label>
          <input type="checkbox"> Remember me
        </label>
      </div>
    </div>
  </div>
  <div class="form-group">
    <div class="col-sm-offset-2 col-sm-10">
      <button type="submit" class="btn btn-default">Sign in</button>
    </div>
  </div>
</form>

```
**校验状态**
Bootstrap 对表单控件的校验状态，如 error、warning 和 success 状态，都定义了样式。使用时，添加`  .has-warning、.has-error 或 .has-success ` 类到这些控件的父元素即可。任何包含在此元素之内的 .control-label、.form-control 和 .help-block 元素都将接受这些校验状态的样式。
```

<div class="form-group has-success">
  <label class="control-label" for="inputSuccess1">Input with success</label>
  <input type="text" class="form-control" id="inputSuccess1">
</div>
<div class="form-group has-warning">
  <label class="control-label" for="inputWarning1">Input with warning</label>
  <input type="text" class="form-control" id="inputWarning1">
</div>
<div class="form-group has-error">
  <label class="control-label" for="inputError1">Input with error</label>
  <input type="text" class="form-control" id="inputError1">
</div>
<div class="has-success">
  <div class="checkbox">
    <label>
      <input type="checkbox" id="checkboxSuccess" value="option1">
      Checkbox with success
    </label>
  </div>
</div>
<div class="has-warning">
  <div class="checkbox">
    <label>
      <input type="checkbox" id="checkboxWarning" value="option1">
      Checkbox with warning
    </label>
  </div>
</div>
<div class="has-error">
  <div class="checkbox">
    <label>
      <input type="checkbox" id="checkboxError" value="option1">
      Checkbox with error
    </label>
  </div>
</div>

```
**添加额外的图标**
你还可以针对校验状态为输入框添加额外的图标。只需设置相应的`  .has-feedback ` 类并添加正确的图标即可。
反馈图标（feedback icon）只能使用在文本输入框 <input class="form-control"> 元素上。
**控件尺寸**
通过 ` .input-lg ` 类似的类可以为控件设置高度，通过 ` .col-lg-* ` 类似的类可以为控件设置宽度。
水平排列的表单组的尺寸
通过添加 .form-group-lg 或 .form-group-sm 类，为 .form-horizontal 包裹的 label 元素和表单控件快速设置尺寸。
##  按钮 
![](/data/dokuwiki/web/bootstrap/pasted/20150613-145442.png)
为 <a>、<button> 或 <input> 元素添加按钮类（button class）即可使用 Bootstrap 提供的样式。
```

<!-- Standard button -->
<button type="button" class="btn btn-default">（默认样式）Default</button>

<!-- Provides extra visual weight and identifies the primary action in a set of buttons -->
<button type="button" class="btn btn-primary">（首选项）Primary</button>

<!-- Indicates a successful or positive action -->
<button type="button" class="btn btn-success">（成功）Success</button>

<!-- Contextual button for informational alert messages -->
<button type="button" class="btn btn-info">（一般信息）Info</button>

<!-- Indicates caution should be taken with this action -->
<button type="button" class="btn btn-warning">（警告）Warning</button>

<!-- Indicates a dangerous or potentially negative action -->
<button type="button" class="btn btn-danger">（危险）Danger</button>

<!-- Deemphasize a button by making it look like a link while maintaining button behavior -->
<button type="button" class="btn btn-link">（链接）Link</button>

```
尺寸:需要让按钮具有不同尺寸吗？使用 ` .btn-lg、.btn-sm 或 .btn-xs ` 就可以获得不同尺寸的按钮。
通过给按钮添加 ` .btn-block ` 类可以将其拉伸至父元素100%的宽度，而且按钮也变为了块级（block）元素。
激活状态:
<button type="button" class="btn btn-primary btn-lg active">Primary button</button>
<button type="button" class="btn btn-default btn-lg active">Button</button>
对于 <button> 元素，是通过 :active 状态实现的。对于 <a> 元素，是通过 .active 类实现的。
禁用状态:
为 <button> 元素添加 disabled 属性，使其表现出禁用状态。
<button type="button" class="btn btn-lg btn-primary" disabled="disabled">Primary button</button>
<button type="button" class="btn btn-default btn-lg" disabled="disabled">Button</button>
##  图片 
**响应式图片**
在 Bootstrap 版本 3 中，通过为图片添加 ` .img-responsive ` 类可以让图片支持响应式布局。其实质是为图片设置了 max-width: 100%;、 height: auto; 和 display: block; 属性，从而让图片在其父元素中更好的缩放。
如果需要让使用了 .img-responsive 类的图片水平居中，请使用 ` .center-block ` 类，不要用 .text-center。
<img src="..." class="img-responsive" alt="Responsive image">
**图片形状**
通过为 <img> 元素添加以下相应的类，可以让图片呈现不同的形状。
<img src="..." alt="..." class="img-rounded">
<img src="..." alt="..." class="img-circle">
<img src="..." alt="..." class="img-thumbnail">
![](/data/dokuwiki/web/bootstrap/pasted/20150613-145843.png)
##  辅助类 
**情境文本颜色**
通过颜色来展示意图，Bootstrap 提供了一组工具类。这些类可以应用于链接，并且在鼠标经过时颜色可以还可以加深，就像默认的链接一样。
<p class="text-muted">...</p>
<p class="text-primary">...</p>
<p class="text-success">...</p>
<p class="text-info">...</p>
<p class="text-warning">...</p>
<p class="text-danger">...</p>
**情境背景色**
和情境文本颜色类一样，使用任意情境背景色类就可以设置元素的背景。链接组件在鼠标经过时颜色会加深，就像上面所讲的情境文本颜色类一样。
<p class="bg-primary">...</p>
<p class="bg-success">...</p>
<p class="bg-info">...</p>
<p class="bg-warning">...</p>
<p class="bg-danger">...</p>
**关闭按钮**
通过使用一个象征关闭的图标，可以让模态框和警告框消失。
<button type="button" class="close" aria-label="Close"><span aria-hidden="true">&times;</span></button>
**三角符号**
通过使用三角符号可以指示某个元素具有下拉菜单的功能。
<span class="caret"></span>
**快速浮动**
通过添加一个类，可以将任意元素向左或向右浮动。
<div class="pull-left">...</div>
<div class="pull-right">...</div>
**让内容块居中**
为任意元素设置 display: block 属性并通过 margin 属性让其中的内容居中。
<div class="center-block">...</div>
**清除浮动**
通过为父元素添加 ` .clearfix ` 类可以很容易地清除浮动（float）。
<!-- Usage as a class -->
<div class="clearfix">...</div>
**显示或隐藏内容**
` .show 和 .hidden ` 类可以强制任意元素显示或隐藏
<div class="show">...</div>
<div class="hidden">...</div>
##  响应式工具 
![](/data/dokuwiki/web/bootstrap/pasted/20150613-150450.png)
##  使用 Less 

Bootstrap 的 CSS 文件是通过 Less 源码编译而来的。Less 是一门预处理语言，支持变量、mixin、函数等额外功能。对于希望使用 Less 源码而非编译而来的 CSS 文件的用户，Bootstrap 框架中包含的大量变量、mixin 将非常有价值。
