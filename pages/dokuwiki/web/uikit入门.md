title: uikit入门 

#  uikit入门 
官网：http://getuikit.com/
中文网站:http://www.getuikit.net/index.html
UIkit是YOOtheme团队开发的一款轻量级开源的前端框架，可以帮助你快速的开发和创建前端UI界面，支持LESS、模块化、自定义主题、及响应式设计。UIKit提供了全面的HTML、CSS及JS组件，它们使用简单，容易定制和扩展。
目前版本是:2.23.0 (2015-11-4)
**` UIkit依赖于jQuery `**
```

<!DOCTYPE html>
<html>
    <head>
        <title></title>
        <link rel="stylesheet" href="uikit.min.css" />
        <script src="jquery.js"></script>
        <script src="uikit.min.js"></script>
    </head>
    <body>
    </body>
</html>

```
CDN
```

<link rel="stylesheet" href="//cdn.bootcss.com/uikit/2.XX.XX/css/uikit.css" />
<script src="//cdn.bootcss.com/uikit/2.22.0/js/uikit.js"></script>

```
##  断点 
UIkit 包含一系列为各种不同的视图宽度实现响应式内容的类。下面的表格提供了一个关于一些可用的视图断点以及相应的设备概述。你可以通过 定制器 来调整所有的断点。
大小	断点	设备
Mini	小于 479px	手机纵向
Small	480px 到 767px	手机横向
Medium	768px 到 959px	平板电脑纵向
Large	960px 到 1199px	台式机或平板电脑横向
Xlarge	1200px 或以上	大屏幕设备

##  CSS 架构 
为了避免与其他CSS框架冲突，所有的UIKit类均以`  uk- ` 作为前缀进行命名。**组件分为组件本身、子对象和调节器，其类名通常沿用组件名**。
对象	概述
组件	理论上一个网站经常用到一部分基于类的组件模块，比如按钮。这些都可重复使用和相互组合。
子对象	子对象都被放置在一个组件中用来增强其功能性，比如一个在提示框中的关闭按钮。
调节器	调节器的类用来调整修改组件及其子对象的样式，比如按钮的颜色和大小。

##  基础知识节选 
提供所有HTML元素的默认样式。你不必担心你的项目在不同的浏览器中的一致性问题。

**1、代码块**
对于多行代码，使用`  <pre> ` 元素定义预格式化文本。它能创建提供保留空格、制表符和换行符的新文本块。内部嵌套'' ```
'' 标签来定义代码块。
重要 为保证正确地渲染，必须确保避开代码中的任何尖括号。
<code xml>
<pre>
    <code>...</ code>
</pre>

```
注意 你也可以添加 效果组件 中介绍到的 ` .uk-scrollable-text ` 类, **这将设置最大高度为300px，并提供Y轴滚动条。**

**2、响应式图片**
` UIKit 中的所有图片默认都是响应的 `。如果布局变窄，图片会调整它们的大小并保持自身的比例。
 **若要避免响应式行为**并保持图像的原始尺寸，可以为单独的图片添加`  .uk-img-preserve ` 类。如果你具有多张图片，也可以将这个类添加到父容器中。

##  布局类 
##  网格 
创建一个完全响应式并可以嵌套的流动网格布局。
UIKit中的网格系统遵循移动优先的方式并且` 最多可容纳10个网格列 `。它使用网格内预定义的类对每个单元格的列宽进行了定义。
向一个**父元素**添加 ` .uk-grid ` 类来创建**网格容器**。对**子元素**添加一个`  .uk-width-* ` 类来**限定单元格的宽度**。网格支持1、2、3、4、5、6和10个单元划分。下面的列表为你提供了可以应用到单元的 uk-width-* 类的概述。
```

.uk-width-1-1	填满可见宽度的100％。
.uk-width-1-2	把网格等分为两半。
.uk-width-1-3 to .uk-width-2-3	将网格划分成三分之一。
.uk-width-1-4 to .uk-width-3-4	将网格划分成四分之一。
.uk-width-1-5 to .uk-width-4-5	将网格划分成五分之一。
.uk-width-1-6 to .uk-width-5-6	将网格划分成六分之一。
.uk-width-1-10 to .uk-width-9-10	将网格划分成十分之一。

```
我们特意为每组单元建立了一种冗余，所以在实际应用时，.uk-width-5-10 类与 .uk-width-1-2 类的实际效果都是一样的。
```

<div class="uk-grid">
    <div class="uk-width-1-2">...</div>
    <div class="uk-width-1-2">...</div>
</div>

```

###  响应式宽度 
UIKit中提供了一些非常有用的响应式宽度的类。基本上它们的使用方法就像通常宽度的类一样，但是它们带有前缀，这样它们只在特定的断点来产生效果。这些类可以结合 效果组件 中的可见性类来使用。**这对于调整不同尺寸设备的布局和内容是非常重要的。**
Class	描述
.uk-width-*	对于任何宽度的设备，网格都保持列并排。
.uk-width-small-*	影响宽度在 480px 以上的设备。网格列将在较小的视口中堆叠。
.uk-width-medium-*	影响宽度在 768px 以上的设备。网格列将在较小的视口中堆叠。
.uk-width-large-*	影响宽度在 960px 以上的设备。网格列将在较小的视口中堆叠。
重要 若要设置网格堆叠列**上下间的边距**，只需添加`  data-uk-grid-margin ` 属性。

###  网格间隙与网格分隔线 
网格排水沟/Grid gutter
好吧，其实是网格之间的**间隔**。网格会自动在列之间创建一个水平间距，以及在连续的两个网格间创建一个垂直方向的间隔。默认情况下，网格间隔在大屏幕上看起来会比较宽。
中等排水沟/Medium gutter
在网格之间应用中等的间隔，只需要添加`  .uk-grid-medium ` 类。
小点的排水沟/Small gutter
在网格之间应用较小的间隔，只需要添加 ` .uk-grid-small ` 类。
消除排水沟/Collapse gutter
完全地去掉间隔，只需要添加 ` .uk-grid-collapse ` 类。

添加`  .uk-grid-divider ` 类用线条分隔网格列。要使用水平线分隔网格，&lt;nowiki&gt;只需在 &lt;hr&gt; 或 &lt;div&gt; 元素添加这个类。&lt;/nowiki&gt;
```

<div class="uk-grid uk-grid-divider">...</div>
<hr class="uk-grid-divider">
<div class="uk-grid uk-grid-divider">...</div>

```
&lt;nowiki&gt;水平分隔线不能用在含有 uk-push-* 类或 uk-pull-* 类的网格中。&lt;/nowiki&gt;

###  网格的嵌套 
使用嵌套网格你可以轻松地扩展你的网格布局。
```

<div class="uk-grid">
    <div class="uk-width-1-2">...</div>
    <div class="uk-width-1-2">
        <div class="uk-grid">
            <div class="uk-width-1-2">...</div>
            <div class="uk-width-1-2">...</div>
        </div>
    </div>
</div>

```
###  居中网格 
添加 效果组件 中的`  .uk-container-center ` 类来让一个**网格居中显示**。

###  网格列偏移 
可以更改列的显示顺序，使其在源代码中保持一个特定的列顺序。添加 ` .uk-push-* ` 类，**向右移动列**。添加`  .uk-pull-* ` 类，**向左移动列**。这可以使你进行类似翻转移动设备或改变窗口宽度时改变列的显示顺序。这些类还可以用于偏移列，在列之间建立额外的空隙。
```

<div class="uk-grid">
    <div class="uk-width-medium-1-2 uk-push-1-2">...</div>
    <div class="uk-width-medium-1-2 uk-pull-1-2">...</div>
</div>

```

###  统一网格列的高度 
要匹配网格列的高度，只需要在你的网格添加`  data-uk-grid-match ` 属性。如果你的网格包含多个row，只有同一个row中的网格列会被匹配。只需要使用 data-uk-grid-match="{row: false}" 属性就能跨越row的隔阂来匹配网格列的高度。
###  匹配面板的高度 
如果想要匹配一个网格中多个面板的高度，只需要添加 ` .uk-grid-match ` 类。在使用data属性时，必须添加 {target:'.uk-panel'} 选择器。

##  包装多个行（row） 
你也可以**创建一个网格，使其包含任意多个列，这些列会自动地换到下一行**。只需添加`  data-uk-grid-margin ` 属性，就能创建网格行（row）之间的margin。通常这样的布局使用了 <ul> 元素。
```

<ul class="uk-grid" data-uk-grid-margin>

    <!-- 这些元素使用百分比宽度 -->
    <li class="uk-width-medium-1-5">...</li>
    <li class="uk-width-medium-3-10">...</li>

    <!-- 这些元素使用像素值宽度 -->
    <li class="uk-width" style="width: 100px;">...</li>
    <li class="uk-width" style="width: 150px;">...</li>

</ul>

```
###  均匀的网格列 
要创建一个子元素的宽度都是均匀分割的网格，你不必为网格中的每个列表元素里添加同样的类。只需要添加一个 .uk-grid-width-* 类到网格本身即可。
Class	描述
.uk-grid-width-1-2	将网格均匀分为两份
.uk-grid-width-1-3	将网格均匀分为三份
.uk-grid-width-1-4	将网格均匀分为四份
.uk-grid-width-1-5	将网格均匀分为五份
.uk-grid-width-1-6	将网格均匀分为六份
.uk-grid-width-1-10	将网格均匀分为十份

```

<ul class="uk-grid uk-grid-width-1-5">
    <li>...</li>
    <li>...</li>
</ul>

```

###  响应式宽度 
UIkit同样提供了网格的响应式宽度类。可以用它保持均匀尺寸的网格列，无论设备宽度如何。
Class	描述
.uk-grid-width-*	影响所有设备宽度
.uk-grid-width-small-*	影响 480px 以上的设备宽度。
.uk-grid-width-medium-*	影响 768px 以上的设备宽度。
.uk-grid-width-large-*	影响 960px 以上的设备宽度。
.uk-grid-width-xlarge-*	影响 1220px 以上的设备宽度。

```

<ul class="uk-grid uk-grid-width-1-2 uk-grid-width-medium-1-3 uk-grid-width-large-1-5">
    <li>...</li>
    <li>...</li>
</ul>

```
##  面板 
创建拥有不同样式的布局盒。
UIKit使用面板勾勒网页内容中的某些部分，它可以拥有不同的样式。通常，**面板被布置在网格组件的网格列中。**
面板组件**由面板本身，面板标题和面板徽章**组成。为了防止产生多余的空白，面板内容顶部和底部的maigin都被移除了。
```

.uk-panel	为<div>元素添加这个类，定义面板组件。
.uk-panel-title	为标题元素添加这个类创建面板标题。
.uk-panel-badge	为 <div> 元素添加这个类创建一个面板的徽章。风格化徽章样式最简单的方法，就是通过加入徽章组件中的修饰符类。

```

```

<div class="uk-panel">
    <div class="uk-panel-badge uk-badge">...</div>
    <h3 class="uk-panel-title">...</h3>
    ...
</div>

```
```

<div class="uk-grid">
    <div class="uk-width-medium-1-2">
        <div class="uk-panel">...</div>
    </div>
    <div class="uk-width-medium-1-2">
        <div class="uk-panel">...</div>
    </div>
</div>

```
###  面板修饰 
使用修饰类为面板添加一个特定的样式是很有必要的。UIkit含有多种面板修饰，你也可以自己创建。
**面板框**
添加`  .uk-panel-box ` 类来创建一个可见的面板框。这是默认的面板修饰样式。为面板创建鼠标经过效果，只需添加 ` .uk-panel-box-hover ` 类。这将在使用锚文本时带来方便。
**主要的面板盒/Panel box primary**
添加 ` .uk-panel-box-primary ` 类来修饰面板框，并以不同的颜色使其显得突出。
```

<div class="uk-panel uk-panel-box uk-panel-box-primary">...</div>

```
**次要的面板盒/Panel box secondary**
为面板框添加 ` .uk-panel-box-secondary ` 类，给它一个白色的背景。
**鼠标经过面板/Panel hover**
添加`  .uk-panel-hover ` 类为面板天鼠标经过效果，这将为用作锚的面板带来便利。
###  带图片预览的面板框 
为了在一个面板内显示不带有任何边框的图片，仅需添加` .uk-panel-teaser `类至该面板内部的&lt;nowiki&gt;&lt;div&gt;&lt;/nowiki&gt;元素。
```

<div class="uk-panel uk-panel-box">
    <div class="uk-panel-teaser">
        <img src="" alt="">
    </div>
</div>

```
###  带图标的面板 
在面板标题中为` &lt;nowiki&gt;<i>或&lt;span&gt;&lt;/nowiki&gt;</i> `元素添加一个` .uk-icon-* `类，可以轻松地将 图标组件中的图标应用在面板中。
```

<div class="uk-panel">
    <h3 class="uk-panel-title"><i class="uk-icon-user"></i>...</h3>
</div>

```

##  块/Block 
通过将内容片段打包成拥有不同样式的块来分割内容。
只需要为容器元素添加 ` .uk-block ` 类，就能使用这个组件了。
```

<div class="uk-block">...</div>

```
**修饰**
使用不同的背景颜色和 padding ，添加以下类中的一个就行了。当两个连续的块拥有相同的背景色时，padding 会自动被分开。
```

.uk-block-default	添加默认的背景色彩，通常是白色或类似的颜色。
.uk-block-muted	添加亮背景色。
.uk-block-primary	添加表示着重的背景色。
.uk-block-secondary	添加另一种背景色，通常是暗色。

```
注意 为了在有色的块中恰到好处地显示色彩、border和其他元素，你可能需要使用对比度组件中的`  .uk-contrast ` 类。
```

<div class="uk-block uk-block-primary uk-contrast">...</div>

```
**Padding**
为块添加较大的 padding ，只需添加一个 ` .uk-block-large ` 类。你还可以使用效果组件中的`  .uk-padding-* ` 类，来设置块中 padding 。
```

<div class="uk-block uk-block-large uk-contrast">...</div>

```
##  布局效果与居中对齐、容器、边距 
一些实用的效果类集合，它们可以用来风格化你的网页内容。

**容器**
添加 ` .uk-container ` 类到一个块元素中，**为其设置一个最大宽度并将网页的主要内容包装在其中**。**对于大屏幕采用不同的最大宽度**。

**居中**
**想要将容器居中**，使用`  .uk-container-center ` 类。对于其它的块元素，你需要设定一个宽度。
```

<div class="uk-width-medium-1-2 uk-container-center">...</div>

```

**浮动与清除浮动**
浮动是创建各式布局的基础。**但浮动需要清除**，否则在最坏的情况下需，你会得到一个杂乱无章的页面。下面的类能帮助你创建基础的布局。
```

.uk-float-left	浮动元素为左对齐。
.uk-float-right	浮动元素为右对齐。
.uk-clearfix	向父容器添加这个类来清除浮动。

```
```

<div class="uk-clearfix">
    <div class="uk-float-right">...</div>
    <div class="uk-float-left">...</div>
</div>

```

**图片与对象的对齐**
有间距地将图片与其他元素（比如文本）对齐。
```

.uk-align-left	向左浮动元素，并创建右侧及底部的间距。
.uk-align-right	向右浮动元素，并创建左侧及底部的间距。
.uk-align-medium-left	仅影响宽度 768px 及以上的设备。
.uk-align-medium-right	仅影响宽度 768px 及以上的设备。
.uk-align-center	居中对齐元素，并创建底部间距。

```
```

<p class="uk-clearfix">
    <img class="uk-align-medium-right" src="" alt="">
    ...
</p>

```

**垂直对齐**
将对象垂直对齐，**你必须为需要对齐的对象创建一个父容器。**
.uk-vertical-align	向父容器添加这个类。这个容器需要被设定一个高度。
` .uk-vertical-align-middle	向子元素添加这个类，使内容居中对齐。 `
.uk-vertical-align-bottom	向子元素添加这个类，使内容与底部对齐。
.uk-height-1-1	这个辅助类用来赋予100％的高度。
```

<div class="uk-vertical-align">
    <div class="uk-vertical-align-middle">
    ...
    </div>
</div>

```
需要对齐的元素需要有一个宽度或最大宽度，它的宽度必须比父容器宽度小。

**居中整个页面**
如果你想将 <html> 和 <body> 元素扩展到整个页面的高度，&lt;nowiki&gt; .uk-height-1-1 &lt;/nowiki&gt;类便派上了用场。创建错误页面时，这是非常有用的。
<html class="uk-height-1-1">
    ...
    <body class="uk-height-1-1">
        <div class="uk-vertical-align">
            <div class="uk-vertical-align-middle">...</div>
        </div>
    </body>
</html>

**水平居中**
水平居中你的网页内容，添加`  .uk-text-center ` 类到父容器并添加`  .uk-container-center ` 类到子元素。这是由于响应式特性而必须这样的。

**视窗高度**
添加 ` .uk-height-viewport ` 类，就可以创建一个填充整个视窗高度的容器，**例如用于全屏图像或视频。**

**元素的定位**
UIkit提供一系列的类去**定位你的内容**，而无须更改你自己的CSS。
.uk-position-top	将元素绝对定位在顶部
.uk-position-top-left	将元素绝对定位在左侧顶部
.uk-position-top-right	将元素绝对定位在右侧顶部
.uk-position-bottom	将元素绝对定位在底部
.uk-position-bottom-left	将元素绝对定位在左侧底部
.uk-position-bottom-right	将元素绝对定位在右侧底部
.uk-position-cover	为元素添加绝对定位，并将其扩展覆盖其父元素
.uk-position-relative	为元素添加relative定位方法
.uk-position-z-index	为元素添加数值为1的 z-index 属性

**响应式对象**
在UIkit中，图片会默认地自适应它的父容器。如果你想将响应式特性应用于媒体元素，比如视频对象，只需要添加下面的类中的一个。
` .uk-responsive-width `	根据父容器的宽度调整对象的宽度，保持对象原始的宽高比。
` .uk-responsive-height `	根据父容器的高度调整对象的高度，保持对象原始的宽高比。
NOTE 同样，可以添加 .uk-responsive-width 类名到 iframe ，此iframe需要有预设的width和height属性。

**外边距 Margin**
添加一个下面的类为块元素添加外边距。
.uk-margin	为一个段落添加相同的顶部和底部外边距。
.uk-margin-top	添加顶部外边距。
.uk-margin-bottom	添加底部外边距。
.uk-margin-left	添加左侧外边距。
.uk-margin-right	添加右侧外边距。
较大的外边距
使用一个下面的类来为块元素添加较大的外边距。
.uk-margin-large	为一个段落添加较大的顶部和底部外边距。
.uk-margin-large-top	添加较大的顶部外边距。
.uk-margin-large-bottom	添加较大的底部外边距。
.uk-margin-large-left	添加较大的左侧外边距。
.uk-margin-large-right	添加较大的右侧外边距。
较小的外边距
使用一个下面的类来为块元素添加较小的外边距。
.uk-margin-small	为一个段落添加较小的顶部和底部外边距。
.uk-margin-small-top	添加较小的顶部外边距。
.uk-margin-small-bottom	添加较小的底部外边距。
.uk-margin-small-left	添加较小的左侧外边距。
.uk-margin-small-right	添加较小的右侧外边距。
移除外边距
添加一个下面的类来移除块元素的外边距。
.uk-margin-remove	移除全部外边距。
.uk-margin-top-remove	移除顶部外边距。
.uk-margin-bottom-remove	移除底部外边距。

**自动外边距Auto margin**
为堆叠的多个元素间距，例如，在较小的视口中堆叠显示多个并列的内联元素时，只需要添加 ` data-uk-margin ` 属性到它们的父元素，即可自动为子元素添加 the .uk-margin-small-top 。
```

<p data-uk-margin>
    <button class="uk-button">...</button>
    <button class="uk-button">...</button>
</p>

```
注意 默认情况下，data属性为堆叠的元素添加 .uk-margin-small-top 类。如果需要添加更大的margin，只需要添加 {cls:'.uk-margin-top'} 选项就行。

**Padding**
移除块元素内的 padding ，添加以下类中的一个就行了。
.uk-padding-remove	移除所有padding.
.uk-padding-top-remove	移除顶部padding.
.uk-padding-bottom-remove	移除底部padding
.uk-padding-vertical-remove	移除顶部和底部padding.

**Border 半径**
要为元素添加**圆角**，添加 ` .uk-border-rounded `即可。要使用**圆形**，添加 ` .uk-border-circle ` 即可。
```

<img class="uk-border-rounded" src="" alt="">
<img class="uk-border-circle" src="" alt="">

```

**可滚动的预格式化文本**
添加`  .uk-scrollable-text ` 类设置一个最大高度，并提供一个垂直滚动条。这对预格式化的文本是非常有用的，它可以让你的代码块节省更多的空间。

**可滚动的盒子**
添加`  .uk-scrollable-box ` 类创建一个具有最大高度及垂直滚动条的看起来像面板的盒子。它可以包含任何类型的内容。
```

<div class="uk-scrollable-box">
    <ul class="uk-list">
        <li><label><input type="checkbox">...</label></li>
        <li><label><input type="checkbox">...</label></li>
    </ul>
</div>

```

**溢出容器/Overflow container**
当容器内部的元素宽度超过了容器本身，只需要为容器的&lt;nowiki&gt; &lt;div&gt;&lt;/nowiki&gt; 元素添加一个`  .uk-overflow-container ` 类，就能为容器带来一个**水平方向的滚动条**。 在响应式网页**中处理表格时很有用**，因为表格可能在某些断点会显得过于宽大。

**显示效果**
添加这些类中的一个改变元素的 display 属性。
.uk-display-block	强制将元素改变成块元素。
.uk-display-inline	强制将元素改变成内联元素。
.uk-display-inline-block	强制将元素改变成内联块元素。

**可见性效果**
.uk-hidden	在所有设备上隐藏该元素。
.uk-hidden-touch	在触控设备上隐藏
.uk-hidden-notouch	在非触控设备上隐藏
.uk-invisible	隐藏该元素，但是不在流量上删除该元素。
.uk-visible-hover	悬停时通过 display: block来显示隐藏的内容。将这个类添加到父元素中。
.uk-visible-hover-inline	悬停时通过 display: inline-block 来显示隐藏的内容。将这个类添加到父元素中。

**响应式可见性**
你可以在特定的视口宽度下对内容进行显示或隐藏。通过设置**断点变量**可以很容易的进行修改。由于不同设备的尺寸变得越来越模糊，所以类的名称保持通用性而不提及任何具体的设备名称。
![](/data/dokuwiki/web/pasted/20151105-113031.png)
##  Flex 布局 
**利用Flexbox的力量创建广泛的布局。**
这个组件使用了Flexbox —— 一个比较新的概念，它拥有强大的布局效果。
用法
若要使用这个组件，只需要添加`  .uk-flex ` 类到一个&lt;nowiki&gt; &lt;div&gt; &lt;/nowiki&gt;元素。这样将会创建flex容器**。默认地，所有flex条目都会左对齐，并被赋予一致的高度和宽度。**
```

<div class="uk-flex">
    <div>...</div>
</div>

```
**行内的Flex**
默认情况下，flex容器显示为块元素。为了仍然保持按照flexbox模型对其内容进行布局，并赋予其行内元素的行为，需要用到`  .uk-flex-inline ` 类来替代 uk-flex.

**修饰**
你可以添加多个不同的类来调整flex的行为。

**对齐**
这些类定义flex条目的水平或垂直对齐，并赋予它们彼此之间的间距。
**.uk-flex-center	添加这个类，水平居中flex条目**
.uk-flex-right	添加这个类，右对齐flex条目
.uk-flex-top	添加这个类，顶部对齐flex条目
**.uk-flex-middle	添加这个类，垂直居中flex条目**
.uk-flex-bottom	添加这个类，底部对齐flex条目
.uk-flex-space-between	添加这个类，使得条目均匀分布，第一个条目在主轴的开头，最后一个条目在主轴的末尾。
.uk-flex-space-around	添加这个类，使得条目均匀分布,使每个条目具有相同的左右空间。
```

<div class="uk-flex uk-flex-middle uk-flex-space-between">...</div>

```

**方向**
这些类用于定义flex主轴的方向。默认地，flex条目按照水平从左到右的方向放置。
.uk-flex-row-reverse	添加这个类，使flex条目从右到左排列。
.uk-flex-column	添加这个类，使flex条目垂直排列成一列。
.uk-flex-column-reverse	添加这个类，使flex条目从下到上排列。

**换行**
**默认情况下，flex条目将它们自身拟合到一行中**。添加 ` .uk-flex-wrap ` 类，**使条目不再匹配视口时切换到另一行**。要改变条目的方向，使它们从右到左排列，添加 ` .uk-flex-wrap-reverse ` 类即可。下面这些类用来修饰换行的flex条目的对齐属性。**强制将 flex条目放入一行，添加 ` .uk-flex-nowrap ` 类即可。**
.uk-flex-wrap-top	添加这个类，使多行flex条目对齐到顶部。
.uk-flex-wrap-middle	添加这个类，使多行flex条目垂直居中。
.uk-flex-wrap-bottom	添加这个类，使多行flex条目对齐到底部。
.uk-flex-wrap-space-between	添加这个类，使条目的行均匀分布，第一行在容器顶部，最后一行在容器底部。
.uk-flex-wrap-space-around	添加这个类，是条目的行均匀分布，每一行都有一样的空间。
```

<div class="uk-flex uk-flex-wrap uk-flex-wrap-reverse uk-flex-wrap-space-around">...</div>

```

**条目排序**
默认地，flex条目根据源码的顺序排列。要将某个元素作为第一个或者最后一个进行显示，只需要添加下列类名中的一个。
.uk-flex-order-first	将此条目显示为第一个
.uk-flex-order-last	将此条目显示为最后一个
.uk-flex-order-first-small
.uk-flex-order-last-small	作用于视口宽度 480px 以上设备。
.uk-flex-order-first-medium
.uk-flex-order-last-medium	作用于视口宽度 768px 以上设备。
.uk-flex-order-first-large
.uk-flex-order-last-large	作用于视口宽度 960px 以上设备。
.uk-flex-order-first-xlarge
.uk-flex-order-last-xlarge	作用于视口宽度 1220px 以上设备。
```

<div class="uk-flex">
   <div class="uk-flex-order-first">...</div>
</div>

```

**条目的规模**
要确定一个flex条目需要占用多大的空间，为条目添加以下类中的一个即可。
.uk-flex-item-none	由内容决定其尺寸
.uk-flex-item-auto	按条目的内容分配空间
.uk-flex-item-1	空间分配完全基于Flex

Flex与网格
**Flex组件可以与 网格 组合使用。**

##  覆盖/Cover 
**扩展图片或视频至覆盖整个容器。**
这个组件允许你使用图片、对象甚至iframe（images, objects or even iframes）来**创建全屏效果**。无论是什么元素，**它都将垂直居中、水平居中并且不会失去原有的比例即实现覆盖它的容器**。` 你还可以在图片或者视频上面放入附加内容，比如文字或图片等。 `
用法
这个覆盖组件具有不同的用法，**取决于你究竟是使用的背景图片、对象或者iframe。**最简单的方式就是为**带有背景图片**的&lt;nowiki&gt; &lt;div&gt; &lt;/nowiki&gt;元素添加 ` .uk-cover-background ` 类。
```

div class="uk-cover-background">...</div>

```
**视频**
创建一个覆盖它的父容器的视频，添加`  .uk-cover-object ` 类到视频。然后用一个容器元素包裹视频并为该容器添加`  .uk-cover ` 类来裁剪内容。
```


<div class="uk-cover" style="height: 300px;">
                                <video class="uk-cover-object" width="600" height="400" autoplay="" loop="" muted="" controls="">
                                    <source src="http://www.quirksmode.org/html5/videos/big_buck_bunny.mp4?test1" type="video/mp4">
                                    <source src="http://www.quirksmode.org/html5/videos/big_buck_bunny.ogv?test1" type="video/ogg">
                                </video>
                            </div>


```
**Iframe**
要将覆盖组件应用到 iframe ，你只需要为 iframe 添加 ` data-uk-cover ` 属性。然后，再添加 ` .uk-cover ` 类到包含iframe的容器来裁剪内容。
```

<div class="uk-cover">
    <iframe data-uk-cover src="" width="" height="" frameborder="0" allowfullscreen></iframe>
</div>

```

**响应式**
为覆盖图片添加响应式行为，你需要添加 ` .uk-invisible ` 类到 &lt;nowiki&gt;&lt;img&gt;&lt;/nowiki&gt; 元素，**并将它放在覆盖元素内部**。这样的话，它就能适应图片的响应式行为了。
注意 添加 效果组件 中的`  .uk-height-viewport ` 类，**会扩展父容器的高度填满整个视口。**
```

<div class="uk-cover-background">
    <img class="uk-invisible" src="" width="" height="" alt="">
</div>

```
**视频**
为视频添加响应式行为，你同样需要为覆盖容器添加 ` .uk-position-relative ` 类，并将 ` .uk-position-absolute ` 类添加到覆盖对象上。对于irame也是这样操作。
```

<div class="uk-cover uk-position-relative">
    <img class="uk-invisible" src="" width="" height="" alt="">
    <video class="uk-cover-object uk-position-absolute" width="" height="">
        <source src="" type="">
    </video>
</div>

```
**内容的定位/Position content**
你还能在覆盖元素上面绝对定位内容。要实现这个效果，只需添加 效果组件 中的`  .uk-position-cover ` 类到图片或视频后面的容器元素。如果想要实现垂直居中并且水平居中，那就使用 Flex 组件 吧。
```

<div class="uk-cover-background uk-position-relative" style="height: 300px; background-image: url(images/placeholder_600x400.svg);">
                                <img class="uk-invisible" src="images/placeholder_600x400.svg" width="600" height="400" alt="">
                                <div class="uk-position-cover uk-flex uk-flex-center uk-flex-middle">
                                    <div style="background: rgba(42, 142, 183, 0.8); font-size: 50px; line-height: 75px; color: #fff;">Bazinga!</div>
                                </div>
</div>

```