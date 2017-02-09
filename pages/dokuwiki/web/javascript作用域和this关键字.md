title: javascript作用域和this关键字 

#  javascript 作用域和 this 关键字 
` **this的指向在函数定义的时候是确定不了的，**只有函数执行的时候才能确定this到底指向谁，实际上**this的最终指向的是那个运行时调用它的对象** `

##  this运行时确定的初步理解 
###  this与函数定义 
```

window.name = "window";
function f(){
    console.log(this.name);
}
f();//window
 
var obj = {name:'obj'};
f.call(obj); //obj

```
在执行f()时，此时f()的调用者是window对象，因此输出”window”
f.call(obj) 是把f()放在obj对象上执行，相当于obj.f(),此时f中的this就是obj,所以输出的是”obj”
下面是例子：
```

function a(){
    var user = "追梦子";
    console.log(this.user); //undefined
    console.log(this); //Window
}
a();

```
按照我们上面说的this最终指向的是调用它的对象**，这里的函数a实际是被Window对象所点出来的**，下面的代码就可以证明。
```

function a(){
    var user = "追梦子";
    console.log(this.user); //undefined
    console.log(this);　　//Window
}
window.a();

```
和上面代码一样吧，其实alert也是window的一个属性，也是window点出来的。

例子2：
```

var o = {
    user:"追梦子",
    fn:function(){
        console.log(this.user);  //追梦子
    }
}
o.fn();

```
**这里的this指向的是对象o，因为你调用这个fn是通过o.fn()执行的，那自然指向就是对象o**，
` 这**里再次强调一点，this的指向在函数创建的时候是决定不了的，在调用的时候才能决定，谁调用的就指向谁，一定要搞清楚这个。** `
###  this与对象定义 
1、对象方法
```

var name = "clever coder";
var person = {
	name : "foocoder",
	hello : function(sth){
		console.log(this.name + " says " + sth);
	}
}
person.hello("hello world");

```
输出 foocoder says hello world。this指向person对象，即当前对象。

2、作为构造函数
new FooCoder();
函数内部的this指向新创建的对象。

3、内部函数
```

var name = "clever coder";
var person = {
	name : "foocoder",
	hello : function(sth){
		var sayhello = function(sth) {
			console.log(this.name + " says " + sth);
		};
		sayhello(sth);
	}
}
person.hello("hello world");//clever coder says hello world

```
**在内部函数中，this没有按预想的绑定到外层函数对象上，而是绑定到了全局对象。**这里普遍被认为是JavaScript语言的设计错误，因为没有人想让内部函数中的this指向全局对象。
**一般的处理方式是将this作为变量保存下来，一般约定为that或者self：**
```

var name = "clever coder";
var person = {
	name : "foocoder",
	hello : function(sth){
		var that = this;
		var sayhello = function(sth) {
			console.log(that.name + " says " + sth);
		};
		sayhello(sth);
	}
}
person.hello("hello world");//foocoder says hello world

```

###  使用call和apply、bind设置this 
this本身是不可变的，但是 JavaScript中提供了call/apply/bind三个函数来在函数调用时设置this的值。
person.hello.call(person, "world");
apply和call类似，只是后面的参数是通过一个数组传入，而不是分开传入。两者的方法定义：
```

call( thisArg [，arg1，arg2，… ] );  // 参数列表，arg1，arg2，...
bind(obj1 [, arg1 [, arg2 [,arg3 [, ...]]]])// 参数列表，arg1，arg2，...
apply(thisArg [，argArray] );     // 参数数组，argArray

```
都是将某个函数绑定到某个具体对象上使用，自然此时的this会被显式的设置为第一个参数。
```

function add(numA, numB){
    console.log( this.original + numA + numB);
}
 
add(1, 2);
 
var obj = {original: 10};
add.apply(obj, [1, 2]);
add.call(obj, 1, 2);
 
var f1 = add.bind(obj);
f1(2, 3);
 
var f2 = add.bind(obj, 2);
f2(3);
// NaN
// 13
// 13
// 15
// 15

```
当直接调用add函数的时候，this将代表window，当执行”this.original”的时候，由于window对象并没有”original”属性，所以会得到”undefined”。
通过call/apply/bind，达到的效果就是把add函数绑定到了obj对象上，当调用add的时候，this就代表了obj这个对象。

###  DOM event handler与this 
当一个函数被当作event handler的时候，this会被设置为触发事件的页面元素（element）。
```

var body = document.getElementsByTagName("body")[0];
body.addEventListener("click", function(){
    console.log(this);
});
// <body>…</body>

```
###  this总结 
其实就可以总结为以下几点：
1.当函数作为对象的方法调用时，this指向该对象。
2.当函数作为单纯函数调用时，this指向全局对象（严格模式时，为undefined）
3.构造函数中的this指向新创建的对象
4.嵌套函数中的this不会继承上层函数的this，如果需要，可以用一个变量保存上层函数的this。
再总结的简单点，如果在函数中使用了this，只有在该函数直接被某对象调用时，该this才指向该对象。

###  更进一步思考 
我们可能经常会写这样的代码：
```

$("#some-ele").click = obj.handler;

```
如果在handler中用了this，this会绑定在obj上么？显然不是，**赋值以后，函数是在回调中执行的，this会绑定到$(“#some-div”)元素上**。这就需要理解函数的执行环境。
那我们如何能解决回调函数绑定的问题？ES5中引入了一个新的方法，bind():
```

fun.bind(thisArg[, arg1[, arg2[, ...]]])
thisArg当绑定函数被调用时,该参数会作为原函数运行时的this指向.当使用new 操作符调用绑定函数时,该参数无效.
arg1, arg2, ...当绑定函数被调用时,这些参数加上绑定函数本身的参数会按照顺序作为原函数运行时的参数.

```
**该方法创建一个新函数，称为绑定函数，绑定函数会以创建它时传入bind方法的第一个参数作为this，传入bind方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数**.
显然bind方法可以很好地解决上述问题。
```

$("#some-ele").click(person.hello.bind(person));
//相应元素被点击时，输出foocoder says hello world

```
其实该方法也很容易模拟，我们看下Prototype.js中bind方法的**源码**：
```

Function.prototype.bind = function(){
  var fn = this, 
  args = Array.prototype.slice.call(arguments), 
  object = args.shift();
  return function(){
    return fn.apply(object,
      args.concat(Array.prototype.slice.call(arguments)));
  };
};

```

##  一、作用域（scope） 
所谓作用域就是：变量在声明它们的函数体以及这个函数体嵌套的任意函数体内都是有定义的。
```

function scope(){
    var foo = "global";
    if(window.getComputedStyle){
        var a = "I'm if";
        console.log("if:"+foo); //if:global
    }
    while(1){
        var b = "I'm while";
        console.log("while:"+foo);//while:global
        break;
    }
    !function (){
        var c = "I'm function";
        console.log("function:"+foo);//function:global
    }();
    console.log(
         foo,//global
         a, // I'm if
         b, // I'm while
         c  // c is not defined
    );
}
scope();

```
（1）scope函数中定义的foo变量，除过自身可以访问以外，还可以在if语句、while语句和内嵌的匿名函数中访问。 因此，foo的作用域就是scope函数体。
（2）` 在javascript中，if、while、for 等代码块不能形成独立的作用域。 `因此，**javascript中没有块级作用域，只有函数作用域。**
 但是，在JS中有一种特殊情况：
 **如果一个变量没有使用var声明，window便拥有了该属性，因此这个变量的作用域不属于某一个函数体,而是window对象。**
```

function varscope(){
    foo = "I'm in function";
    console.log(foo);//I'm in function
}
varscope();
console.log(window.foo); //I'm in function

```
##  二、作用域链（scope chain） 
所谓作用域链就是：一个函数体中嵌套了多层函数体，并在不同的函数体中定义了同一变量， 当其中一个函数访问这个变量时，便会形成**一条作用域链（scope chain）。**
```

foo = "window";
function first(){
    var foo = "first";
    function second(){
       var foo = "second";
       console.log(foo);
    }
    function third(){
       console.log(foo);
    }
    second(); //second
    third();  //first
}
first();

```
当执行second时，JS引擎会将second的作用域放置链表的头部，其次是first的作用域，最后是window对象，于是会形成如下作用域链：
second->first->window,  此时，JS引擎沿着该作用域链查找变量foo, 查到的是”second”
当执行third时，third形成的作用域链：third->first->window, 因此查到的是：”frist”
特殊情况：with语句
JS中的with语句主要用来临时扩展作用域链，将语句中的对象添加到作用域的头部。with语句结束后，作用域链恢复正常。

##  实战应用 
code1:
```

var foo = "window";
var obj = {
    foo : "obj",
    getFoo : function(){
        return function(){
            return this.foo;
        };
    }
};
var f = obj.getFoo();
f(); //window

```
code2:
```

var foo = "window";
var obj = {
    foo : "obj",
    getFoo : function(){
        var that = this;
        return function(){
            return that.foo;
        };
    }
};
var f = obj.getFoo();
f(); //obj

```

**code1和code2解析：**
code1:
执行var  f = obj.getFoo()返回的是一个匿名函数，相当于:
```

var f = function(){
     return this.foo;
}

```
**f() 相当于window.f(), 因此f中的this指向的是window对象，this.foo相当于window.foo, 所以f()返回"window"** 

code2:
执行var f = obj.getFoo() 同样返回匿名函数，即：
```

var f = function(){
     return that.foo;
}

```
唯一不同的是f中的this变成了that, 要知道that是哪个对象之前，先确定f的作用域链：f->getFoo->window 并在该链条上查找that,此时可以发现that指代的是getFoo中的this, getFoo中的this指向其运行时的调用者，从var f = obj.getFoo() 可知**此时this指向的是obj对象，因此that.foo 就相当于obj.foo,所以f()返回"obj"**
参考：http://web.jobbole.com/85195/
http://web.jobbole.com/39305/
http://web.jobbole.com/85198/