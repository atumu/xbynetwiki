title: css对齐 

#  css对齐 
##  使用 margin 属性来水平居中对齐 
可通过将左和右外边距设置为 "auto"，来对齐块元素。
把左和右外边距设置为 auto，规定的是均等地分配可用的外边距。**结果就是居中的元素**：
实例
```

.center
{
margin-left:auto;
margin-right:auto;
width:70%;
background-color:#b0e0e6;
}

```
` 提示：如果宽度是 100%，则对齐没有效果。 `
##  使用 position 属性进行左和右对齐 
对齐元素的方法之一是**使用绝对定位**：
```

.right
{
position:absolute;
right:0px;
width:300px;
background-color:#b0e0e6;
}

```
` 注释：绝对定位元素会被从正常流中删除，并且能够交叠元素。 `
##  使用 float 属性来进行左和右对齐 
对齐元素的另一种方法是使用 float 属性：
```

.right
{
float:right;
width:300px;
background-color:#b0e0e6;
}

```
