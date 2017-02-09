title: javascript函数定义的几种方式 

#  javascript函数定义的几种方式 
先看几种常用的定义方式：
function func1([参数]){/*函数体*/}
var func2=function([参数]){/*函数体*/};
var func3=function func4([参数]){/*函数体*/}; **这是一个构造函数**
var func5=new Function();
上述第一种方式是最常用的方式，不用多说。
第二种是将一匿名函数赋给一个变量，调用方法：func2([函数]);
第三种是将func4赋给变量func3，调用方法：func3([函数]);或func4([函数]);
第四种是声明func5为一个对象。
再看看它们的区别：
```

function func(){
  //函数体
}
//等价于
var func=function(){
  //函数体
}

```
但同样是定义函数，在用法上有一定的区别。
```

<script>
//这样是正确的
func(1);
function func(a)
{
  alert(a);
}
</script>
<script>

```
```

//这样是错误的，会提示func未定义，主要是在调用func之前没有定义
func(1);
var func = function(a)
{
  alert(a);
}

```
```

//这样是正确的，在调用func之前有定义
var func = function(a)
{
  alert(a);
}
func(1);
</script>

```
