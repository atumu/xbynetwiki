title: css技巧总结 

#  css技巧总结 
##  CSS清除浮动 
1.设置父元素overflow属性为hidden：
.box {
overflow:hidden;
}
2.设置空的div，属性clear:both;
.clearfloat {
clear:both;
}
3.为父元素添加一个类，样式如下：
.clearfix:after {
content: ".";
clear: both;
height: 0;
visibility: hidden;
display: block;
}

