title: css伪类与按钮_链接 

#  css伪类与按钮、链接 
伪类
W3C："W3C" 列指示出该属性在哪个 CSS 版本中定义（CSS1 还是 CSS2）。
属性	描述	CSS
:active	向被激活的元素添加样式。	1
:focus	向拥有键盘输入焦点的元素添加样式。	2
:hover	当鼠标悬浮在元素上方时，向元素添加样式。	1
:link	向未被访问的链接添加样式。	1
:visited	向已被访问的链接添加样式。	1
:first-child	向元素的第一个子元素添加样式。	2
:lang	向带有指定 lang 属性的元素添加样式。	2

例如：
```

<!DOCTYPE html>  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">  
<title>Insert title here</title>  
<style type="text/css">
.btn {  
 background: rgba(0,0,102,1); 
 line-height: 15px;  
 height: 95px;  
 width: 95px;  
 /* margin: 10px 0 0 20px;margin-top 和 margin-left 一句就可以实现了 */  
 border: none; /* border-top\left\right\bottom 也可以缩到一句 */  
}  
.btn:active{  
 background: rgba(0,0,102,0.5); 
 line-height: 15px;  
 height: 85px;  
 width: 85px;  
  /*margin: 10px 0 0 20px; margin-top 和 margin-left 一句就可以实现了 */  
 border: none; /* border-top\left\right\bottom 也可以缩到一句 */  
}  
</style>  
</head>  
<body>  
<center>  
    <button class="btn" onclick="">  
    </button>  
</center>  
</body>  
</html>  

```