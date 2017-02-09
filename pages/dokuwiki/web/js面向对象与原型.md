title: js面向对象与原型 

#  js面向对象与原型 
##  typeof和instanceof 
```

//typeof主要用了检查值类型数据，如：
alert(typeof (1) + " " + typeof ("1") + " " + typeof (false) + " " + typeof (undefined));
//instanceof主要用了检查对象，如：
var arr = new Array();
alert((arr instanceof Object) + " " + (arr instanceof Array) + " " + (arr instanceof Number));//既是Object也是array，但不是Number

```
##  一、 工厂模式： 
```

function createPerson(name) {
    var o = new Object();
    o.name = name;
    o.sayName = function () {
        alert(this.name);
    };
    return o;
}
var obj = createPerson("张三");
var obj2 = createPerson("李四");
alert(obj instanceof Object);
alert(obj instanceof createPerson)

```
由上可知，工厂模式简单、思路清晰、容易理解，也可以创建对象**，不过有个缺点不能确定对象类型。因为它总是一个object类型，而不能判定是createPerson类型。**
##  二、构造函数模式  
```

var obj = { name: "李四" };
function Person(name) {
    this.name = name;
    this.sayName = function () {
        alert(this.name);
    };
}
var per = new Person("张三");
per.sayName();//张三
var per2 = new Person("李四");
alert(per.sayName==per2.sayName);//false

```

其实，构造函数模式我们在上篇博文就简单介绍过了。同样，**构造函数模式也不完美，因为每个实例化出来的对象所拥有的方法都是独立的，而一个对象类型的方法完全是可以同享引用来节省内存空间**。

##  三、原型模式 
1.0在使用原型模式之前，我们首先需要了解什么是原型。我的理解就是，原始对象类型的模型。每个对象都有一个属性（prototype）指向对象的原型。
```

function Person() {
    this.sayHi1 = function () { }
}
Person.prototype.sayHi2 = function () { };

var per1 = new Person();
var per2 = new Person();
alert(per1.sayHi1 === per2.sayHi1);//每个实例化出来的对象所独有的，所以为false
alert(per1.sayHi2 === per2.sayHi2);//因为是同一个引用，所以为true

```
我们再次证明了` 构造函数中的属性(或是方法、对象)是实例化对象独有的，原型中的属性(或是方法、对象)是共享的。 `
第一个比较是对象的属性，所以每个实例对象拥有独立的方法，而第二个比较是原型方法，就算是实例对象，它们直接也是引用共享的。
我们看到了 per1.sayHi2 ####  per2.sayHi2  比较是true。**（是全等的意思，不仅比较值，还比较类型。）**
![](/data/dokuwiki/web/pasted/20151207-100732.png)
```

我们看到了 __proto__ 指向的就是我们所谓的原型（只有Firefox、 Safari 和 Chrome浏览器有此属性）。还有一个 constructor 指向我们的构造函数。
1.1 __proto__ 和原型 prototype 的关系（其实__proto__并不是一个js语言中规定的对象属性，只是某些浏览器实现了）

function Person() {
    this.name1 = "张三",
    this.sayHi1 = function () { }
}
Person.prototype.sayHi2 = function () { };

var per1 = new Person();
var per2 = new Person();
alert(per1.constructor);//constructor指向了构造函数
alert(per1.constructor.prototype);//constructor.prototype 指向了构造函数的原型
alert(per1.constructor.prototype === per1.__proto__);//true 由此看出__proto__和原型的关系。（指向了构造函数的原型）


1.2如果原型中的属性和构造函数中的属性重名，会优先访问构造函数中的属性 

function Person(name) {
    this.name1 = name;
};
Person.prototype.name1 = "test1";
Person.prototype.name2 = "test2";

var per1 = new Person("name1");
alert(per1.name1);//访问到的是实例对象中的name1属性“name1”
delete per1.name1;//删除实例对象中的name1属性
alert(per1.name1);//访问类型原型中的name1属性“test1”

alert(per1.name2);//访问原型属性“name2”
per1.name2 = "name2";//这里并不是修改了原型属性“name2”的值，而是为实例对象动态添加了一个“name2”的属性，并赋值。
alert(per1.name2);//访问实例属性“name2”
delete per1.name2;
alert(per1.name2);//访问原型属性“name2”

```

##  继承方式 
###  一、原型链继承 
1.通过设置prototype指向&ldquo;父类&rdquo;的实例来实现继承。
```

function Obj1() {    this.name1 = "张三";
}
function Obj2() {
}
Obj2.prototype = new Obj1();var t2 = new Obj2();
alert(t2.name1);

```
```

function Obj1() {    this.arr = ["张三"];
}function Obj2() {
}
Obj2.prototype = new Obj1();var t2 = new Obj2();
alert(t2.arr);//打印&ldquo;张三&rdquo;t2.arr[t2.arr.length] = "李四";var t1 = new Obj2();
alert(t1.arr);//打印&ldquo;张三,李四&rdquo;

```
这里有个明显的缺点就是：（如果父类的属性是引用类型，那么我们在对象实例修改属性的时候会把原型中的属性修改，**这样会在每个实例对象中改变数据，而这不是我们想要的效果**）
###  2. 利用构造函数来实现继承 
```

function Obj1() {    this.arr = ["张三"];
}
function Obj2() {
    Obj1.call(this);//【1.新增】}
 
}
 //Obj2.prototype = new Obj1();【2.注释这行】
  var t2 = new Obj2();
alert(t2.arr);//打印&ldquo;张三&rdquo;
t2.arr[t2.arr.length] = "李四";
var t1 = new Obj2();
alert(t1.arr);//打印&ldquo;张三
  

```
我们看到上面代码，就注释了一行，新增了以后。打印出来的效果完全不一样了。**现在的arr属性是每个实例对象独有的了。（之前是定义到原型上的，而原型的属性对每个实例都是共享的）**
同样，单纯的这种方式也是有问题的。因为我们这样就无法继承对象的方法了。如：
```

function Obj1() {    this.arr = ["张三"];
}
Obj1.prototype.sayHi = function () { alert(this.arr); }////【1.新增】
function Obj2() {
    Obj1.call(this);
}        
var t2 = new Obj2();//t2里面是没有sayHi方法的

```
我们可以使用原型和构造的混用来解决，如下：
###  3.通过原型和构造来实现继承 
```

function Obj1() {    this.arr = ["张三"];
}
Obj1.prototype.sayHi = function () { alert(this.arr); }
function Obj2() {
    Obj1.call(this);
}
Obj2.prototype = new Obj1();//【1.新增】
var t2 = new Obj2();
t2.sayHi();

```
如上，通过构造函数中的  Obj1.call(this); 和设置原型属性 Obj2.prototype = new Obj1(); 结合使用，完美解决问题。
这里需要注意一个地方,如果把 Obj2.prototype = new Obj1(); 改成 Obj2.prototype = Obj1.prototype ; 的话，会有你想不到的问题。如：
```

function Obj1() {    this.arr = ["张三"];
}
Obj1.prototype.sayHi = function () { alert(this.arr); }function Obj2() {
    Obj1.call(this);
}
Obj2.prototype = Obj1.prototype;//【1.新增】
var t2 = new Obj2();
t2.constructor.prototype.sayHi = function () { alert("test") };//修改Obj2中的原型的方法
var t1 = new Obj1();
t1.sayHi();//影响到了Obj1中的原型的方法。因为 Obj2.prototype = Obj1.prototype;让两个对象的原型指向了同一处。//所以还是只能用Obj2.prototype = new Obj1();

```
###  4.什么是原型链  
如：
```


//*************Obj1****
function Obj1() {    this.arr = ["张三"];
}
Obj1.prototype.sayHi = function () { alert(this.arr); }//*************Obj2****
function Obj2() {
    Obj1.call(this);    
    this.name = "张三";
}
Obj2.prototype = new Obj1();
Obj2.prototype.sayHi2 = function () { alert(this.name); };//*************Obj3****
function Obj3() {
    Obj2.call(this);
}
Obj3.prototype = new Obj2();
Obj3.prototype.sayHi3 = function () { };//*******************
var t3 = new Obj3();
t3.sayHi();

```
 Obj3继承Obj2，Obj2继承Obj1。我们的Obj3的实例对象访问sayHi的时候，会先去Obj3的实例对象中找sayHi方法（没找到），然后去Obj3的原型中找（没找到），然后去父类Obj2的原型中找（没找到），然后去Obj1的原型中找（找到了）。**这个找的路径就是原型链。**

参考：http://haojima.net/zhaopei/516.html
http://haojima.net/zhaopei/517.html