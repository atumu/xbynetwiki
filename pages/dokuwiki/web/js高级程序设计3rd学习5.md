title: js高级程序设计3rd学习5 

#  JS高级程序设计3rd学习5函数表达式 
ES中定义函数的方式有两种:一种是函数声明，另一种是函数表达式。
函数声明:
function abc(arg1){
}
函数表达式
var abc=function(arg1){
}
##  函数声明提升 
JavaScript Hoisting，即JavaScript声明提升，包括函数声明提升和变量声明提升。
**声明的提升，是按作用域来提升的。**比如说，函数中的一个变量声明是这样的：
```

function foo() {  
    console.log(a);//undefined  
    var a = 2;  
} 

``` 
这里面的变量a只能提升到函数foo内部的最上方，提升不到foo函数外面的作用域当中，只能是这样的：
```

function foo（） {  
    var a;  
    console.log(a);  
    a = 2;  
} 

``` 
**函数声明（Function delaration）能提升，但是函数表达式（Function expression）不能提升，**例：
```

foo(); // 错误 
var foo = function bar() {    
    // ...    
};  

``` 
```

foo(); //正确
function foo() {  
    console.log( 1 );  
} 

``` 

同时，声明提升遵循函数是“第一公民”（first class）的准则，**函数声明提升优先于变量声明提升。**
##  递归函数 
```

            function factorial(num){
                if (num <= 1){
                    return 1;
                } else {
                    return num * factorial(num-1);
                }
            }

            var anotherFactorial = factorial;
            factorial = null;
            alert(anotherFactorial(4));  //error!

```
解决上述错误的方案一：使用arguments.callee，它是一个指向正在执行函数的指针。
```

           function factorial(num){
                if (num <= 1){
                    return 1;
                } else {
                    return num * arguments.callee(num-1);
                }
            }

            var anotherFactorial = factorial;
            factorial = null;
            alert(anotherFactorial(4));  //24

```
但是上述方式在严格模式下，不能通过脚本访问arguments.callee,会导致错误。
下面我们来看另一种解决方案:
```

         var factorial=(function f(num){
                if (num <= 1){
                    return 1;
                } else {
                    return num * f(num-1);
                }
            });

```
##  闭包（Scope Closure） 
有不少开发人员搞不清匿名函数与闭包的区别。**闭包是指有权访问另一个函数作用域中的变量的函数。**创建闭包的常见方式就是在一个函数内部创建另一个函数。
```

function createFun(name){
	return function(obj1){
		var v1=obj1[name];
          	return v1;
	}
}

```
var v1=obj1[name];这行代码在createFun返回后还能获取name的值，**因为内部函数的作用域中包含了createFun()的作用域，即使这个时候createFun已经执行完毕并返回了也是如此。**
要想理解闭包，我们需要了解以前说过的**作用域链**的概念。**当某个函数被调用时会创建一个执行环境以及相应的作用域链。然后初始化函数的活动对象(如上面函数的this,name参数,arguments)。**
这个作用域链被保存在内部的[ [scope] ]属性中。作用域链本质上是一个指向变量对象的指针列表。当需要变量时会依次进行搜索整个作用域链。
一般来说，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域。**但是闭包的情况又有所不同。在另一个函数内部定义的函数会将包含函数(外部函数)的活动对象添加到它的作用域链中。**
从这个我们可以知道，**闭包会导致更多的内存占用，所以应该尽量少用闭包。**
###  闭包与变量 
```

           function createFunctions(){
                var result = new Array();
                
                for (var i=0; i < 10; i++){
                    result[i] = function(num){
                        return function(){
                            return num;
                        };
                    }(i);
                }
                
                return result;
            }

```
###  关于闭包中的this对象问题 
**this对象是在运行时基于函数的执行环境绑定的**。在全局函数中，this等于window。而当函数被作为某个对象的方法调用时，this等于那个对象。**不过匿名函数的执行环境具有全局性即此时this等于window。**
下面先看看有问题的代码：
```

        var name = "The Window";
        
        var object = {
            name : "My Object",
        
            getNameFunc : function(){
                return function(){
                    return this.name;
                };
            }
        };
        
        alert(object.getNameFunc()());  //"The Window"

```
解决办法:
```

            var name = "The Window";
            
            var object = {
                name : "My Object",
            
                getNameFunc : function(){
                    var that = this; //关键一步
                    return function(){
                        return that.name;
                    };
                }
            };
            
            alert(object.getNameFunc()());  //"MyObject"

```
##  模仿块级作用域(重要) 
JS中没有块级作用域的概念，这意味着在块语句中定义的变量，实际上是在包含函数中而非语句中创建的。也就是说它会自动加入当前函数的执行环境的作用域。
```

            function outputNumbers(count){
                for (var i=0; i < count; i++){
                    alert(i);
                }
            
                alert(i);   //count
            }

            outputNumbers(5);

```
###  私有作用域 
**匿名函数可以有用来模仿块级作用域来避免这个问题。**
```

(function(){
//这里是块级作用域
})();

```
以上代码**定义并立即调用**了一个匿名函数。将函数声明包含在一对圆括号()内表示它**实际上是一个函数表达式**。而紧随其后的另一对圆括号()会**立即调用这个函数**。
无论声明地方，只要临时需要一些变量就可以使用私有作用域。
```

            function outputNumbers(count){
            
                (function () {
                    for (var i=0; i < count; i++){
                        alert(i);
                    }
                })();
                
                alert(i);   //causes an error
            }

            outputNumbers(5);

```
在这个私有作用域中能够访问变量count，是因为这个匿名函数是一个闭包，它能够访问包含作用域中的所有变量。
这种技术经常在全局作用域中被用在函数外部，从而限制向全局作用域中添加过多的变量和函数。同时这种做法也可以减少闭包导致的占用内存问题。因为没有指向匿名函数的引用。执行完毕后便可以立即销毁其作用域链了。
##  私有变量 
私有变量包括函数的参数、局部变量和在函数内部定义的其他函数。
```

function add(num1,num2){
	var sum=num1+num2;
	return sum;
};

```
这里有三个私有变量num1,num2,sum
我们把有权访问私有变量和私有函数的公共方法称为特权方法(privileged method)
###  静态私有变量 
```

           (function(){
                var name = ""; //私有变量
                Person = function(value){  //函数表达式，由于Persion变量没有使用var声明，所以它是全局的，属于window对象。                
                    name = value;                
                };
                
                Person.prototype.getName = function(){ //特权方法，访问私有变量name
                    return name;
                };
                
                Person.prototype.setName = function (value){
                    name = value;
                };
            })();
            
            var person1 = new Person("Nicholas"); 
            alert(person1.getName());   //"Nicholas"
            person1.setName("Greg");
            alert(person1.getName());   //"Greg"
                               
            var person2 = new Person("Michael");
            alert(person1.getName());   //"Michael"
            alert(person2.getName());   //"Michael"

```
###  模块模式 
```

            function BaseComponent(){
            }
            function OtherComponent(){
            }
            var application = function(){
                //private variables and functions
                var components = new Array();
                //initialization
                components.push(new BaseComponent());
                //public interface
                return {
                    getComponentCount : function(){
                        return components.length;
                    },
            
                    registerComponent : function(component){
                        if (typeof component == "object"){
                            components.push(component);
                        }
                    }
                };
            };

            application.registerComponent(new OtherComponent());
            alert(application.getComponentCount());  //2

```
###  增强的模块模式 
```

            var application = function(){
            
                //private variables and functions
                var components = new Array();
            
                //initialization
                components.push(new BaseComponent());
            
                //create a local copy of application
                var app = new BaseComponent();
            
                //public interface
                app.getComponentCount = function(){
                    return components.length;
                };
            
                app.registerComponent = function(component){
                    if (typeof component == "object"){
                        components.push(component);
                    }
                };
            
                //return it
                return app;
            }();

            alert(application instanceof BaseComponent);
            application.registerComponent(new OtherComponent());
            alert(application.getComponentCount());  //2

```