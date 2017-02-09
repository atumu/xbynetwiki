title: js高级程序设计3rd学习4 

#  JS高级程序设计3rd学习4面向对象 
JS中对象是一组没有特定顺序的值，值可以是基本类型值，对象值或者函数值。
每个对象都是基于一个引用类型创建的。
##  ES中有两种对象属性 
ES中有两种对象属性：数据属性和访问器属性
###  1、数据属性 
数据属性特征
```

[[Configurable]]表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。默认true。
[[Enumerable]]表示能否通过for-in循环返回属性。默认true。
[[Writable]]表示能否修改属性的值。默认true。
[[Value]]包含这个属性的数据值。读取属性值的时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。默认undefined

```
修改属性默认特性的方式（理解用，此方法不常用）
ECMAScript5的方法，Object.defineProperty(targetObject, targetProperty, descriptor):
  * targetObject，属性所在的对象；
  * targetProperty，属性的名字；
  * descriptor，描述符（configurable、enumerable、writable和value，注意都是小写）。
```

//修改writable特性
var person = {};
Object.defineProperty(person, "name", {
  writable: false;
  value: "MirrorAvatar"
});
console.log(person.name);  //MirrorAvatar
person.name = "Mudface";  //严格模式（use strict）下，这句会抛出错误
console.log(person.name);  //MirrorAvatar
//修改configurable特性
var person ={};
Object.defineProperty(person, "name", {
  configurable: false,
  value: "MirrorAvatar"
});
console.log(person.name);  //MirrorAvatar
delete person.name;  //控制台会输出false，严格模式（use strict）下，这句会抛出错误
console.log(person.name);  //MirrorAvatar
//configurable特性的不可逆性
var person = {};
Object.defineProperty(person, "name", {
  configurable: false,
  value: "MirrorAvatar"
});
//抛出错误，控制台输出：TypeError: Cannot redefine property: name
Object.defineProperty(person, "name", {
  configurable: true,
  value: "MirrorAvatar"
});

```
Object.defineProperty()方法若是不指定configurable、enumerable和writable的时候，都默认是false。
IE8对Object.defineProperty支持不好，请不要在此浏览器上用此方法。
###  2、访问器属性 
访问器属性特征
```

[[Configurable]]表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。默认true。
[[Enumerable]]表示能否通过for-in循环返回属性。对于直接在对象上定义的属性。默认值为true。
[[Get]]在读取属性时调用的函数。默认值为undefined。
[[Set]]在写入属性时调用的函数。默认值为undefined。

```
访问器属性定义
访问器属性不能直接定义，必须使用ECMAScript5的Object.defineProperty()来定义。
```

var blog = {
  _year: 2015,
  edition: 1
};
Object.defineProperty(blog, "year", {
  get: function() {
    return this._year;
  },
  set: function(newValue) {
    if(newValue > 2015) {
      this._year = newValue;
      this.edition += newValue - 2015;
    }
  }
});
blog.year = 2016;  
console.log(blog.edition);  //2

```
_year前面的下划线是一种常用的记号，用于表示只能通过对象方法访问的属性。
**year这个属性就是所谓的访问器属性。**
由控制台输出结果可以发现： 设置一个属性值会使其他属性值改变，这是访问器属性的常见使用方式。
不需要同时指定getter和setter方法。

定义多个属性
ECMAScript5定义的Object.defineProperties()方法。
```

var blog = {};
Object.defineProperties(blog, {
  _year: {
    value: 2015
  },
  edition: {
    value: 1
  },
  year: {
    get: function() {
      return this._year;
    },
    set: function(newValue) {
      if(newValue > 2015) {
        this._year = newValue;
        this.edition += newValue - 2015;
      }
    }
  }
});
blog.year = 2016;
console.log(blog.edition);  // 输出1。为什么是1？不是2？因为还有个属性writable未设置，默认false，不可写。

```
##  创建对象的模式 
###  1、工厂模式 
```

       function createPerson(name, age, job){
            var o = new Object();
            o.name = name;
            o.age = age;
            o.job = job;
            o.sayName = function(){
                alert(this.name);
            };    
            return o;
        }
        
        var person1 = createPerson("Nicholas", 29, "Software Engineer");
        var person2 = createPerson("Greg", 27, "Doctor");
        
        person1.sayName();   //"Nicholas"
        person2.sayName();   //"Greg"

```
工厂模式解决的问题：避免创建多个相似对象的重复代码。
缺点：没有解决对象识别问题(即instanceof 只会将其识别为Object，而不是具体类型)。而且里面的方法达不到共用的效果。

###  3、构造函数模式 
ES中构造函数可用来创建特定类型的对象。
```

       function Person(name, age, job){
            this.name = name;
            this.age = age;
            this.job = job;
            this.sayName = function(){
                alert(this.name);
            };    
        }
        
        var person1 = new Person("Nicholas", 29, "Software Engineer");
        var person2 = new Person("Greg", 27, "Doctor");
        
        person1.sayName();   //"Nicholas"
        person2.sayName();   //"Greg"
        
        alert(person1 instanceof Object);  //true
        alert(person1 instanceof Person);  //true
        alert(person2 instanceof Object);  //true
        alert(person2 instanceof Person);  //true
        
        alert(person1.constructor == Person);  //true
        alert(person2.constructor == Person);  //true
        
        alert(person1.sayName == person2.sayName);  //false       

```
一般，**构造函数名的第一个字母大写**。构造函数本身也是函数，只不过可以用来通过new来创建对象。
**使用new通过构造器创建对象会经历以下4步:**
  * 创建一个新对象；
  * 将构造函数的作用域赋给新对象(因此**this就指向了这个新对象**)；
  * 执行构造函数中的代码
  * 返回新对象
每个对象内部都有一个特殊的属性constructor用来指向构造函数。

构造函数模式的优点：可以将它的实例标识为一种特定的类型，如上面的Persion，而不是一律的Object
缺点:每个方法都要在每个实例上重新创建一遍。这比较耗内存，而且也没有必要。达不到方法共享的目的。

解决方式一：将方法定义移至外部。缺点会导致全局函数越来越多。毫无封装性。
```

       function Person(name, age, job){
            this.name = name;
            this.age = age;
            this.job = job;
            this.sayName = sayName;
        }
        
        function sayName(){
            alert(this.name);
        }

```
###  3、原型模式 
我们创建的每个函数都有一个**prototype(原型)属性，这个属性是一个指针**，指向一个原型对象，而这个对象的用途是包含可以有**特定类型**的所有实例**共享**的属性和方法。
而每个原型对象都会自动获得一个constructor属性，它是指向所在函数的指针。
判断原型isPrototypeOf()或者Object.getPrototypeOf()。hasOwnProperty()方法可以判断一个属性是存在于实例中还是原型中。
虽然可以通过对象实例访问保存在原型中的值，但是却不能通过对象实例重写原型中的值。
constructor属性。创建函数后，自动获取到此属性。默认情况下，函数prototype的constructor指向函数本身。
```

function Foo() {  
}  
Foo.prototype.constructor === Foo;  //true 

``` 
prototype上其他方法继承自Object，如toString()、valueOf(),hasOwnPrototype()、isPrototypeOf()等
当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象。ECMA-262第5版中管这个指针叫[ [Prototype] ]。**这个连接存在于实例与构造函数的原型对象之间，而不是存在于实例与构造函数之间。**
![](/data/dokuwiki/web/pasted/20160305-154417.png)
参考：http://mirroravatar.iteye.com/blog/2190410
```

        function Person(){
        }
        
        Person.prototype.name = "Nicholas";
        Person.prototype.age = 29;
        Person.prototype.job = "Software Engineer";
        Person.prototype.sayName = function(){
            alert(this.name);
        };
        
        var person1 = new Person();
        var person2 = new Person();
        
        person1.name = "Greg";
        alert(person1.name);   //"Greg" – from instance
        alert(person2.name);   //"Nicholas" – from prototype

	 delete person1.name;
        alert(person1.name);   //"Nicholas" - from the prototype


```


**原型与in操作符**
```

        alert(person1.name);   //"Greg" – from instance
        alert(person1.hasOwnProperty("name"));  //true
        alert("name" in person1);  //true

        for (var prop in person1){}

```
**原型重写**
```

        function Person(){
        }
        
        Person.prototype = {
            constructor:Persion,  //由于原型重写会导致constructor属性丢失，所以需要显式指定
            name : "Nicholas",
            age : 29,
            job: "Software Engineer",
            sayName : function () {
                alert(this.name);
            }
        };

        var friend = new Person();
        
        alert(friend instanceof Object);  //true
        alert(friend instanceof Person);  //true
        alert(friend.constructor == Person);  //false
        alert(friend.constructor == Object);  //true

```
**原型的动态性**：可以在对象创建之后再修改它的原型，并且所有修改能立即反馈到该对象中.
```

        function Person(){
        }
        
        Person.prototype = {
            constructor: Person,
            name : "Nicholas",
            age : 29,
            job : "Software Engineer",
            sayName : function () {
                alert(this.name);
            }
        };
        
        var friend = new Person();
        
        Person.prototype.sayHi = function(){
            alert("hi");
        };
        
        friend.sayHi();   //"hi" – works!

```
但是此时不能进行原型重写，否则就会切断了构造函数与最初原型之间的联系。

####  原型对象存在的问题 
原型模式最大问题是由于其共享的本质导致的。
**对于原型中包含引用类型值的属性来说，共享将导致相互影响。**
```

       function Person(){
        }
        
        Person.prototype = {
            constructor: Person,
            name : "Nicholas",
            age : 29,
            job : "Software Engineer",
            friends : ["Shelby", "Court"], //这是一个引用类型属性。
            sayName : function () {
                alert(this.name);
            }
        };
        
        var person1 = new Person();
        var person2 = new Person();
        
        person1.friends.push("Van");
        
        alert(person1.friends);    //"Shelby,Court,Van"
        alert(person2.friends);    //"Shelby,Court,Van"
        alert(person1.friends === person2.friends);  //true

```
###  4、组合使用构造函数模式和原型模式 
创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。构造函数用于定义实例属性，而原型模式用于定义方法和共享的属性。这样便可以最大限度节省了内存。
这也是使用最广泛、认同度最高的一种创建自定义类型的方法。
```

        function Person(name, age, job){
            this.name = name;
            this.age = age;
            this.job = job;
            this.friends = ["Shelby", "Court"];
        }
        
        Person.prototype = {
            constructor: Person,
            sayName : function () {
                alert(this.name);
            }
        };
        
        var person1 = new Person("Nicholas", 29, "Software Engineer");
        var person2 = new Person("Greg", 27, "Doctor");
        
        person1.friends.push("Van");
        
        alert(person1.friends);    //"Shelby,Court,Van"
        alert(person2.friends);    //"Shelby,Court"
        alert(person1.friends === person2.friends);  //false
        alert(person1.sayName === person2.sayName);  //true

```
###  5、动态原型模式 
```

        function Person(name, age, job){
        
            //properties
            this.name = name;
            this.age = age;
            this.job = job;
            
            //methods
            if (typeof this.sayName != "function"){
            
                Person.prototype.sayName = function(){
                    alert(this.name);
                };
                
            }
        }

        var friend = new Person("Nicholas", 29, "Software Engineer");
        friend.sayName();

```
这里只有在sayName()方法不存在的情况下才会将它添加到原型中。如果有多个需要添加项，那么也只要检查其中一个即可。而没有必要每一个都用if进行判断。

###  7.稳妥构造函数模式 
所谓稳妥对象，指的是没有公共属性，而且其方法也不引用this的对象。稳妥对象最适合在一些安全的环境中（这些环境中会禁止使用this和new），或者在防止数据被其他应用程序（如Mashup程序）改动时使用。
稳妥构造函数遵循与寄生构造函数类似的模式，**但有两点不同：一是新创建对象的实例方法不引用this；二是不使用new操作符调用构造函数。**
```

function Person(name, age, job) {  
    //创建要返回的对象  
    var o = new Object();  
    //可以在这里定义私有变量和函数  
    //添加方法  
    o.sayName = function() {  
        alert(name);  
    }  
    //返回对象  
    return o;  
}  
  
var friend = Person("MirrorAvatar", 3, "coder");  
friend.sayName();  //"MirrorAvatar" 

``` 
特点： 1. 没有new操作符 2. 没有this

##  继承机制 
ES只支持实现继承，不支持接口继承。而且其**实现继承主要是依赖` 原型链 `来实现的。**
###  1、原型链 
原型链，作为实现继承的主要方式。
  * 每个函数都有一个原型（prototype）属性；
  * 原型属性是一个指针，指向一个对象；
  * 对象的用途是包含可以由特定类型的所有实例共享的属性和方法。
让原型对象等于另一个类型的实例，假如这个类型的原型又等于另一个类型的实例，这样层层递进，构成了实例和原型的链条。
![](/data/dokuwiki/web/pasted/20160305-154638.png)
####  原型链的代码实现的基本模式 
```

//组合构造函数模式和原型模式  
function SuperType() {  
    this.property = true;  
}  
SuperType.prototype.getSuperValue = function() {  
    return this.property;  
};  
  
function SubType() {  
    this.subproperty = false;  
}  
  
//继承了SuperType  
SubType.prototype = new SuperType();  
  
SubType.prototype.getSubValue = function() {  
    return this.subproperty;  
};  
  
var instance = new SubType();  
console.log(instance.getSuperValue());  //true  
console.log(instance.constructor === SuperType);  //true  
console.log(SubType.prototype.constructor === SuperType);  //true  

```
SubType继承了SuperType。继承是通过创建SuperType实例，然后赋值给SubType.prototype实现的。**该实现的本质是重写了原型对象，代之以一个新类型的实例.**
**新原型拥有SuperType实例所拥有的全部属性和方法，而且其内部的prototype指针指向了SuperType.prototype**
![](/data/dokuwiki/web/pasted/20160305-154916.png)
**关系结果：instance指向SubType的原型，Sub-Type的原型又指向SuperType的原型。**
注意两点：
getSuperValue()方法仍然还在SuperType.prototype中，但property则位于Sub-Type.prototype中。这是因为property是一个实例属性，而get-SuperValue()则是一个原型方法。既然SubType.prototype现在是SuperType的实例，那么property当然就位于该实例中了。
**instance.constructor现在指向的是SuperType。原因：SubType 的原型指向了另一个对象——SuperType 的原型，而这个原型对象的constructor 属性指向的是SuperType。**
通过实现原型链，本质上扩展了**原型搜索机制**。调用instance.getSu-perValue()会经历三个搜索步骤：
  * 搜索实例；
  * 搜索Sub-Type.prototype；
  * 搜索SuperType.prototype，最后一步才会找到该方法。
####  原型链的最顶层 
**所有函数的默认原型都是Object的实例，因此默认原型都会包含一个内部指针，指向Object.prototype。**
![](/data/dokuwiki/web/pasted/20160305-155358.png)
SubType继承了SuperType，而SuperType了继承Object。当调用instance.toString()时，实际上调用的是保存在Object.prototype中的那个方法。
####  确定原型和实例的关系 
instanceof操作符和isPrototypeOf()方法
```

//instanceof操作符  
console.log(instance instanceof Object);  //true  
console.log(instance instanceof SuperType); //true  
consolo.log(instance instanceof SubType);  //true  
  
//isPrototypeOf()方法  
console.log(Object.prototype.isPrototypeOf(instance));  //true  
console.log(SuperType.prototype.isPrototypeOf(instance));  //true  
console.log(SubType.prototype.isPrototypeOf(instance));  //true  

```
####  子类型定义方法注意 
**重写父类方法或定义新方法注意**：给原型添加方法的代码一定要放在替换原型的语句之后。为什么？因为，继承的本质就是重写子类的prototype,如果写在继承前面，那么后面创建实例的时候就访问不到新添加或修改的方法了。
在通过原型链实现继承时，**不能使用对象字面量创建原型方法。**
```

function SuperType() {  
    this.property = true;  
}  
  
SuperType.prototype.getSuperValue = function() {  
    return this.property;  
};  
  
function SubType() {  
    this.subproperty = false;  
}  
  
//继承SuperType,这个在前  
SubType.prototype = new SuperType();  
  
//子类添加父类没有的方法，在后  
SubType.prototype.getSubValue = function() {  
    return this.subproperty;  
};  
  
//重写父类方法，在后  
SubType.prototype.getSuperValue = function() {  
    return false;  
};  
  
var instance = new SubType();  
console.log(instance.getSuperValue());  //false  
  
var instanceSuper = new SuperType();  
console.log(instanceSuper.getSuperValue());  //true  

```
####  原型链的问题 
  * **包含引用类型值的原型，将会导致各个之类相会影响。**
  * 在创建子类型的实例时，不能向超类型的构造函数中传递参数。
```

//第一个问题  
function SuperType() {  
    this.colors = ["blue", "red", "yellow"];   //引用类型
}  
function SubType() {  
}  
//继承  
SubType.prototype = new SuperType();  
var instance1 = new SubType();  
instance1.colors.push("black");  
console.log(instance1.colors);  //"blue,red,yellow,black"  
  
var instance2 = new SubType();  
console.log(instance2.colors);  //"blue,red,yellow,black"  

```
实践中很少会单独使用原型链。

### 2、 借用构造函数 
借用构造函数（constructor stealing）进行继承的基本思想就是：**在子类型构造函数的内部调用超类型构造函数。**
那么，如何调用？**函数只不过是在特定环境中执行代码的对象**，因此通过使用` apply()和call()方 `法也可以在（将来）新创建的对象上执行构造函数。
父类中引用类型属性实例化：
```

function SuperType() {  
    this.number = [1, 2, 3];  
}  
  
function SubType() { 
  //继承了SuperType
    SuperType.call(this);  
}  
  
var instance1 = new SubType();  
instance1.number.push(4);  
instance1.number;  // "1,2,3,4"  
  
var instance2 = new SubType();  
instance2.number;  // "1,2,3"  

```
构造函数继承的优势：**可以传递参数**
```

function SuperType(name) {  
    this.name = name;  
}  
  
function SubType() {  
    Super.call(this, "MirrorAvatar");  
    this.age = 3;  
}  
  
var instance = new SuperType();  
  
instance.name;  //MirrorAvatar  
instance.age;  //3  

```
优点：解决原型链中引用类型属性导致的问题，而且可以传递参数给父类构造器。
缺点:方法都在构造函数中定义，函数无法复用。而且超类型的原型中定义的方法对子类型而言不可见。(因为它没有采用原型链，而是通过apply()或call())
借用构造函数的技术也是很少单独使用的。

### 3、 组合继承 
组合继承（combination inheritance），有时候也叫做伪经典继承，**指的是将原型链和借用构造函数的技术组合到一块**，从而发挥二者之长的一种继承模式。
实现思路
**使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。**这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。
```

//构造函数实现实例属性，构造函数的优点  
function SuperType(name) {  
    this.name = name;  
    this.numbers = [1,2,3];  
}  
//原型来定义方法，原型链的优点  
SuperType.prototype.sayName = function () {  
    alert(this.name);  
};  
function SubType (name, age) {  
    //继承属性  
    SuperType.call(this, name);  
    this.age = age;  
}  
//继承方法  
SubType.prototype = new SuperType();  
//子类新定义的方法一定要在上面的继承代码之后，要不然新写的方法就没了  
SubType.prototype.sayAge = function () {  
    alert(this.age);  
}  
var instance1 = new SubType("MirrorAvatar", 3);  
instance1.numbers.push(33);  
instance1.numbers;  // "[1,2,3,33]"  
instance1.sayName();  //MirrorAvatar  
instance1.sayAge();  //3  
  
var instance2 = new SubType("Cindy", 4);  
instance2.numbers;  //"[1,2,3]"  
instance2.sayName();  //Cindy  
instance2.sayAge();  //4  

```
**组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为JavaScript中最常用的继承模式。而且，instanceof和isPrototypeOf()也能够用于识别基于组合继承创建的对象。**
###  4、原型式继承 
```

function object(o){
	function F(){}
	F.prototype=o;
	return new F();
}

```
本质上，object()函数对传入其中的对象执行了一次浅复制。
ES5中新增` Object.create() `方法规范化了原型式继承。
```

        var person = {
            name: "Nicholas",
            friends: ["Shelby", "Court", "Van"]
        };
                           
        var anotherPerson = Object.create(person, {
            name: {
                value: "Greg"
            }
        });
        
        alert(anotherPerson.name);  //"Greg"

```
###  5、寄生组合式继承 
通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。构造函数 + 原型链。
**寄生组合式继承是引用类型最理想的继承范式。**
```

function object(o){
	function F(){}
	F.prototype=o;
	return new F();
}
function inheritPrototype(subType, superType) {  
    //创建对象  
    var prototype = object(superType.prototype);  
    //增强对象  
    prototype.constructor = subType;  
    //制定对象  
    subType.prototype = prototype;  
}  

```
```

function SuperType(name) {  
    this.name = name;  
    this.colors = ["red", "blue", "green"];  
}  
SuperType.prototype.sayName = function() {  
    alert(this.name);  
};  
  
function SubType(name, age) {  
    SuperType.call(this, name);  
    this.age = age;  
}  
inheritPrototype(SubType, SuperType);  
SubType.prototype.sayAge = function() {  
    alert(this.age);  
};  

```
##  总结(重要): 

本章重点:对象创建模式与类型继承模式、原型与原型链
**对象创建模式分类：**
1、工厂模式：优点，避免重复代码书写。缺点无法复用函数，无法自定义引用类型。
2、构造函数模式:优点，可以自定义引用类型。缺点，无法复用函数
3、原型模式:优点，可以服用函数，自定义引用类型。缺点，当存在引用类型属性时，会导致各个实例相互影响，封装性和安全性被破坏。
4、**组合使用构造函数模式与原型模式**：这是创建自定义类型的最常见方式。**构造函数用于定义实例属性，而原型模式用于定义方法和共享的属性。**这样便可以最大限度节省了内存。
继承主要通过原型链实现。
5、动态原型模式
**继承模式分类：**
1、原型链模式:缺点,当存在引用类型属性时，将导致封装性被破坏，造成相互影响。
2、借用构造函数模式:缺点,无法复用函数。父类原型中定义的成员对子类不可见。
3、**组合继承模式**:这是最推荐的继承方式。**使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。**这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。
4、原型式继承：本质执行对给定对象的浅复制。
5、寄生式继承：
6、寄生组合式继承：这是最理想的继承模式。