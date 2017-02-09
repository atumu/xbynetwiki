title: css算术运算calc 

#  css算术运算calc()与自适应 
CSS中也可以做简单运算
通过CSS中的calc方法可以进行一些简单的运算，从而达到动态指定元素样式的目的。
```

.container{
	background-position: calc(100% - 50px) calc(100% - 20px);
}

```
calc()是css3新添加属性，它可以让你使用一个算术表达式来表达长度值，这意味着可以用它来定义div的宽度，并设置margin、padding、border等。
calc()的运算规则
  * 使用”+”、”-”、”*”、”/”四则运算；
  * 可以使用百分比、px、em、rem等单位；
  * 可以混合使用各种单位进行计算。
实例1：定位在页面上的块元素，含有外边距
```

.banner {
  position:absolute;
  left: 40px;
  width: -moz-calc(100% - 80px);
  width: -webkit-calc(100% - 80px);
  width: calc(100% - 80px);
  border: solid black 1px;
  box-shadow: 1px 2px;
  background-color: yellow;
  padding: 6px;
  text-align: center;
}

```
实例2：自动调整大小的表单，又适应容器

```

input {
  padding: 2px;
  display: block;
  width: -moz-calc(100% - 1em);
  width: -webkit-calc(100% - 1em);
  width: calc(100% - 1em);
}  

#formbox {
  width: -moz-calc(100%/6);
  width: -webkit-calc(100%/6);
  width: calc(100%/6);
  border: 1px solid black;
  padding: 4px;
}
<form>
  <div id="formbox">
  <label>Type something:</label>
  <input type="text">
  </div>
</form>

```