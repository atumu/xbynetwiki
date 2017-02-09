title: html5学习一 

#  HTML5学习一 
如果需要IE8及其以下支持HTML5标签，可以采用modernizr.js（http://modernizr.com）来支持
HTML5样板文件(https://html5boilerplate.com/)
##  精简的部分 
```

<!Doctype html>
<meta charset="utf-8">
<link rel="stylesheet "href="cm.css">
<script src="abc.js"/>

```

##  <a>标签内可以嵌入多个元素： 
```

<a href="">
<h2>...</h2>
<p>asada</p>
</a>

```

##  全新的语义化标签 
语义化标签利于理解和利于搜索引擎检索
```

<section>用于定义文档或者程序的区域
<nav>用于定义主导航区域
<article>文章正文
<aside>侧边栏
<hgroup>包裹一组<h1>、<h2>等标题
<header>用作其他区块的介绍说明或简介，摘要等
<footer>用作页脚或者辅助信息
<address>
<i>通常用作字体图标的承载

```

##  媒体标签 
```

<video>、<audio>
支持格式MP4,ogg,webm
写法一:
<video src="" width="" height="" controls autoplay loop poster="视频缩略图" >不支持提示信息</video>
写法二:
<video src="" width="" height="" controls autoplay loop poster="视频缩略图" >
	<source src="" type="video/mp4">
	不支持提示信息
</video>

```
