title: js高级程序设计3rd学习1 

#  JS高级程序设计3rd学习1简介 
JavaScript Reference & API :https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference
ECMAScript doc:https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Language_Resources
JavaScript和ECMAScript(ES)通常都被用来表达相同含义
JS组成部分:
  * 核心(ECMAScript)
  * 文档对象模型DOM
  * 浏览器对象模型BOM

web浏览器只是ECMAScript实现可能的**宿主环境**之一。现在还有Node（服务端实现）和Flash等实现。
ECMAScript版本，最新为第5版，简称ES5。以下均以web浏览器环境说明。
<script >标签一般放在< /body>的最后面.以优先加载DOM结构，然后再下载执行JS。
<noscript >元素用于当浏览器禁用javascript功能时，提供提示信息。
严格模式：在js文件开头添加如下代码: "use strict";
语句以一个分号结束。
**未经初始化的变量值默认为undefined
var操作符定义的变量就是当前作用域中的局部变量，省略var定义变量，那么这个变量将变成全局变量。**
```

function test(){
	var v1;//为初始化，值为undefined
	var v2=0;//局部变量
	v3=4;//全局变量
}
alert(v2);//undefined;
alert(v3);//4

```
##  数据类型 
5种基本数据类型：Undefined,Boolean,Number,String,Null
1种复杂数据类型:Object,Function也是Object
注意：undefined派生自null，应该明确的让变量保存null值，而不是未定义，这样有助于区分它们。

###  typeof操作符与instanceof 
typeof主要用来鉴定是否为基本类型。typeof是一个关键字，不过可以采用如下方式使用:typeof a或者typeof(a),如果返回以下值对应关系
  * "undefined"-如果值未定义
  * "boolean"
  * "string"
  * "number"
  * "object"-如果是对象或者null
  * "function"

instanceof用来鉴定具体的复杂类型

Boolean类型
Number类型:Number.MIN_VALUE,Number.MAX_VALUE. Number.NEGATIVE_INFINITY. JS内置的isFinite()方法判断是否无穷大。
NAN:非数值是一个特殊的数值。如用来表示除0问题。NAN不与任何值相等，包括它自己。JS内置isNan()用于判断是否为NaN
String类型：它是不可变的。toString()方法转换为字符串

###  数值转换 
Number(),parseInt().parseFloat()

###  Object类型 
var o=new Object();
Object类型是所有它的实例的基础。
Object每个实例都具有下列属性和方法:
constructor:构造函数
hasOwnProperty(name)。
isPrototypeOf(object)
toString()
valueOf()

##  等于与全等 
等于不检查类型，自动转换类型；全等检查类型。

##  for-in语句 
for(property in expression){}，可以用来迭代对象的属性。
JS对象的属性没有顺序的。

##  with语句 
with语句的作用是将代码的作用域设置到一个特定的对象中。例如
```

with(location){
	var a=href;//location.href
}

```
##  switch语句 
```

switch(exp){
	case value: //可以是字符串
		statement
		break;
	default:
		statement
}

```

##  函数 
function abc(arg1,arg2){}
###  理解参数 
**JS函数不介意传递进来多少个参数，也不在乎传进来的参数类型**。也就是说即便你定义了只接受两个参数的哦函数，但是调用的时候可以传递1个，2个，3个，甚至更多的参数。**函数声明的参数只是期望得到的参数数量。**
所以这一特性，也决定了**` JS中不允许函数重载 `**，否则后面的定义会覆盖前面的定义。
在函数体内可以通过` arguments `对象来访问这个参数数组。



