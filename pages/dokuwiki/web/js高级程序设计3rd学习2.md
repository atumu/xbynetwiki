title: js高级程序设计3rd学习2 

#  JS高级程序设计3rd学习2作用域和内存问题 
JS变量包含两种不同数据类型的值：基本类型值和引用类型值。
  * 基本类型值(Number,Boolean,String)的传递是直接复制值，不会造成相互影响
  * 引用类型值的传递只是复制了一个指针指向的还是同一个对象，会造成相互影响。
JS中传递参数都是按值传递的。值可以是基本类型值，也可以是引用类型值。

##  执行环境及作用域 
执行环境(execution context)是JS中最为重要的一个概念。每个执行环境都有一个与之相关联的变量对象。
全局执行环境是最外围的一个执行环境。在浏览器中，被认为是window对象。
某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁。
` 没有块级作用域 `：**也就是说在if或者for，while等局部代码块中定义的变量会被添加到函数的执行上下文，是在整个函数执行上下文都可用的。**这一点与java不同，例如：
```

if(true){
	var color='red';
}
alert(color); //red

for(var i=0;i<10;i++){

}
alert(1);//10

```
**每个函数都有自己的执行环境**。每个环境都维护一个**变量的作用域链(scope chain)**.用来保证对执行环境有权访问的所有变量和函数的有序访问。
```

   var color = "blue";
        
        function changeColor(){
            var anotherColor = "red";
        
            function swapColors(){
                var tempColor = anotherColor;
                anotherColor = color;
                color = tempColor;
                //color, anotherColor, and tempColor are all accessible here
            }
            //color and anotherColor are accessible here, but not tempColor        
            swapColors();
        }
        
        changeColor();
        //只能访问color
        alert("Color is now " + color);

```
