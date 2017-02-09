title: css3学习一 

#  CSS3学习一 
CSS3技巧:文字阴影，盒阴影，渐变，圆角，自定义字体，多重背景图片，动画，变形，转换。

##  浏览器兼容私有前缀 
以圆角为例：
```

.round{
	border-radius:10px;
}

```
但是浏览器兼容写法为:
```

.round{
	-ms-border-radius:10px;/*IE*/
	-moz-border-radius:10px;/*w3c*/
	-webkit-border-radius:10px;/*w3c*/
	border-radius:10px;/*w3c*/
}

```

##  CSS3多栏布局，多列网格 
```

<div id="main" role="main">
	<p>haahaM</p>
	<p>asasdafd</p>
</div>
#main{
	column-width:12em;
	//column-count:4;栏数
	//column-gap:2em;栏位间隙
	//column-rule:thin dotted #999 分界线	
}

```
##  文字换行wrap 
```

word-wrap:break-word;

```
##  CSS3选择器 
属性选择器：
  * 属性是否存在， img[alt]
  * 属性是否等于某个值 img[alt="haha"]
  * 属性是否以特定前缀开头  img[alt^="haha"]
  * 属性是否包含特定字符串  img[alt*="haha"]
  * 属性是否以特定后缀结尾 img[alt$="haha"]
结构伪类：
  * li:first-child
  * li:last-child
  * p:empty
  * p:before	
  * p:after

否定选择器：
  * p:not(.redclass)
  * div:not(p)

##  颜色： 

  * color:#fe0208
  * color:rgb(255,102,0)
  * color:rgba(255,144,12,0.8)


##  CSS3阴影与背景渐变效果 
###  文字阴影 
```

.ele{
	text-shadow:1px 1px 1px #cccccc;  /*右，下，模糊距离，颜色*/
}
.ele{
	text-shadow:4px 4px 0px rgba(255,120,0,0.8);  /*右，下，模糊距离，颜色*/
}
.ele a{
	text-shadow:none; /*取消文字阴影*/
}
.ele a{
	text-shadow:none; -4px -4px 0px rgba(255,120,0,0.8);/*左上方阴影*/
}

```
###  盒阴影 
```

img{
  box-shadow:0px 3px 5px #444444
}

```
###  背景渐变 
背景颜色代码资源：http://lea.verou.me/css3patterns/
```

线性渐变
background:linear-gradient(90deg,#ffffff 0%, #e4e4e4 50%, #ffffff 100%); /*角度，起始，中间，结束*/
径向渐变：
background:radial-gradient(center,ellipse cover, #ffffff 72%, #dddddd 100%); /*起点，形状及半径选取方式，中间，结束*/

```

##  变形 
  * scale缩放 transform:scale(0.5)
  * translate移动transform:(40px,0px)
  * rotate旋转transform:rotate(90deg)
  * skew斜切transform:skew(10deg,2deg)
  * matrix



