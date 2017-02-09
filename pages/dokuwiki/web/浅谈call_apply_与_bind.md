title: 浅谈call_apply_与_bind 

#  JS核心系列：浅谈 call apply 与 bind 
原文:http://web.jobbole.com/85221/
在JavaScript中，` call、apply和bind 是Function对象自带的三个方法，这三个方法的主要作用是改变函数中的this指向，从而可以达到接花移木的效果 `。本文将对这三个方法进行详细的讲解，并列出几个经典应用场景。

##  call(thisArgs [,args...]) 
该方法可以传递一个thisArgs参数和一个参数列表，**thisArgs指定了函数在` 运行期 `的调用者，也就是函数中的this对象，而参数列表会被传入调用函数中。**thisArgs的取值有以下4种情况：
（1） 不传，或者传null,undefined， 函数中的this指向window对象
（2） 传递另一个函数的函数名，函数中的this指向这个函数的引用
（3） 传递字符串、数值或布尔类型等基础类型，函数中的this指向其对应的包装对象，如 String、Number、Boolean
（4） 传递一个对象，函数中的this指向这个对象
```

function a(){
    console.log(this); //输出函数a中的this对象
}
function b(){} //定义函数b
 
var obj = {name:'onepixel'}; //定义对象obj
 
a.call(); //window
a.call(null); //window
a.call(undefined);//window
a.call(1); //Number
a.call(''); //String
a.call(true); //Boolean
a.call(b);// function b(){}
a.call(obj); //Object

```
这是call的核心功能，**它允许你在一个对象上调用该对象没有定义的方法，并且这个方法可以访问该对象中的属性，**至于这样做有什么好处，我待会再讲，我们先看一个简单的例子：
```

var a = {
 
    name:'onepixel', //定义a的属性
 
    say:function(){ //定义a的方法
        console.log("Hi,I'm function a!");
    }
};
 
function b(name){
    console.log("Post params: "+ name);
    console.log("I'm "+ this.name);
    this.say();
}
 
b.call(a,'test');
>>
Post params: test  //参数name
I'm onepixel    //a函数中的name
I'm function a! //a中的函数say（）

```
**当执行b.call时，字符串test作为参数传递给了函数b,由于call的作用，函数b中的this指向了对象a, 因此相当于调用了对象a上的函数b,而实际上a中没有定义b 。**

##  apply(thisArgs[,args[]]) 
**apply和call的唯一区别是第二个参数的传递方式不同，apply的第二个参数必须是一个数组**，而call允许传递一个参数列表。值得你注意的是，虽然apply接收的是一个参数数组，**但在传递给调用函数时，却是以参数列表的形式传递，**我们看个简单的例子：
```

function b(x,y,z){
    console.log(x,y,z);
}
 
b.apply(null,[1,2,3]); // 1 2 3

```

###  bind(thisArgs [,args...]) 
**bind是ES5新增的一个方法**，它的传参和call类似，但又和call/apply有着显著的不同，
**即调用call或apply都会` 自动执行 `对应的函数，而bind不会执行对应的函数，只是返回了对函数的引用。**粗略一看，bind似乎比call/apply要落后一些，那ES5为什么还要引入bind呢？
其实，**ES5引入bind的真正目的是为了弥补call/apply的不足，` 由于call/apply会对目标函数自动执行，从而导致它无法在事件绑定函数中使用 `，因为事件绑定函数不需要我们手动执行，它是在事件被触发时由JS内部自动执行的。而bind在实现改变函数this的同时又不会自动执行目标函数，因此可以完美的解决上述问题，**看一个例子就能明白：
```

var obj = {name:'onepixel'};
 
/**
 * 给document添加click事件监听，并绑定onClick函数
 * 通过bind方法设置onClick的this为obj，并传递参数p1,p2
 */
document.addEventListener('click',onClick.bind(obj,'p1','p2'),false);
 
//当点击网页时触发并执行
function onClick(a,b){
    console.log(
            this.name, //onepixel
            a, //p1
            b  //p2
    )
}

```
当点击网页时，onClick被触发执行，输出onepixel p1 p2, 说明onClick中的this被bind改变成了obj对象，为了对bind进行深入的理解，我们来看一下bind的polyfill实现：
```

if (!Function.prototype.bind) {
    Function.prototype.bind = function (oThis) {
        var aArgs = Array.prototype.slice.call(arguments, 1),
            fToBind = this, //this在这里指向的是目标函数
            fBound = function () {
                return fToBind.apply(
                    //如果外部执行var obj = new fBound(),则将obj作为最终的this，放弃使用oThis
                    this instanceof fToBind
                            ? this  //此时的this就是new出的obj
                            : oThis || this, //如果传递的oThis无效，就将fBound的调用者作为this

                    //将通过bind传递的参数和调用时传递的参数进行合并，并作为最终的参数传递
                    aArgs.concat(Array.prototype.slice.call(arguments)));
            };

        //将目标函数的原型对象拷贝到新函数中，因为目标函数有可能被当作构造函数使用
        fBound.prototype = this.prototype;

        //返回fBond的引用，由外部按需调用
        return fBound;
    };
}

```
##  应用场景一：继承 
大家知道，JavaScript中没有诸如Java、C#等高级语言中的extend 关键字，**因此JS中没有继承的概念，如果一定要继承的话，call和apply可以实现这个功能：**
```

function Animal(name,weight){
   this.name = name;
   this.weight = weight;
}
 
function Cat(){
    Animal.call(this,'cat','50');
  //Animal.apply(this,['cat','50']);
 
   this.say = function(){
      console.log("I am " + this.name+",my weight is " + this.weight);
   }
}
 
var cat = new Cat();
cat.say();//I am cat,my weight is 50

```
当通过new运算符产生了cat时，Cat中的this就指向了cat对象，而继承的关键是在于Cat中执行了Animal.call(this,’cat’,’50′) 这句话，在call中将this作为thisArgs参数传递，于是Animal方法中的this就指向了Cat中的this，而cat中的this指向的是cat对象，所以Animal中的this指向的就是cat对象，**在Animal中定义了name和weight属性，就相当于在cat中定义了这些属性，因此cat对象便拥有了Animal中定义的属性，从而达到了继承的目的。**

##  应用场景二：移花接木 
在讲下面的内容之前，我们首先来认识一下JavaScript中的一个**非标准专业术语：ArrayLike(类数组/伪数组)**
**ArrayLike 对象即拥有数组的一部分行为**，在DOM中早已表现出来，而jQuery的崛起让ArrayLike在JavaScript中大放异彩。ArrayLike对象的精妙在于它和JS原生的Array类似，但是它是自由构建的，它来自开发者对JavaScript对象的扩展，也就是说：对于它的原型(prototype)我们可以自由定义，而不会污染到JS原生的Array。
**ArrayLike对象在JS中被广泛使用，比如DOM中的NodeList, 函数中的arguments都是类数组对象，这些对象像数组一样存储着每一个元素，但它没有操作数组的方法，**而我们可以通过call将数组的某些方法移接到ArrayLike对象，从而达到操作其元素的目的。比如我们可以这样遍历函数中的arguments:
```

function test(){
    //检测arguments是否为Array的实例
    console.log(
            arguments instanceof Array, //false
            Array.isArray(arguments)  //false
    );
    //判断arguments是否有forEach方法
    console.log(arguments.forEach); //undefined

    // 将数组中的forEach应用到arguments上
    Array.prototype.forEach.call(arguments,function(item){
        console.log(item); // 1 2 3 4
    });

}
test(1,2,3,4);

```
