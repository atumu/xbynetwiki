title: js高级程序设计3rd学习3 

#  JS高级程序设计3rd学习3引用类型 
ES中，引用类型是一种数据结构，通常被称为类。
对象是某个引用类型的实例。
##  Object类型 
```

var o=new Object();
o.name='hjah';

var persion={
	name:'haha'
}
persion.age=11;
alert(persion['name']);
alert(persion.name);

```
##  Array类型 
特点：
  * ES中数组的每一项可以保存任何类型的数据，也就是说，**数组中不同项的数据类型可以不同。**
  * ES中数组的**大小是可以动态调整的**
  * ES中数组**可以模拟栈、列表**等数据结构
```

var colors=new Array();
var colors=new Array('red','white');
var colors=['red','blue'];
alert(colors.length);

```
**JS数组的length属性不是只读的。**如果修改length属性，将导致数组添加或者删除某些项。
###  检测数组 
if(` Array.isArray(value) `){}

转换方法：toString()，join(splitChar)
###  栈方法 
栈是一种LIFO(last-in-first-out)后进先出的数据结构.
pop(),push(args)
```

 var colors = new Array();                      //create an array
        var count = colors.push("red", "green");       //push two items
        alert(count);  //2
        
        count = colors.push("black");                  //push another item on
        alert(count);  //3
        
        var item = colors.pop();                       //get the last item
        alert(item);   //"black"
        alert(colors.length);  //2

```
###  队列方法 
队列数据结构规则是FIFO(first-in-first-out)先进先出。
shift(),push() 或者是unshift(),pop()
```

 var colors = new Array();                      //create an array
        var count = colors.push("red", "green");       //push two items
        alert(count);  //2
        
        count = colors.push("black");                  //push another item on
        alert(count);  //3
        
        var item = colors.shift();                     //get the first item
        alert(item);   //"red"
        alert(colors.length);  //2

```
###  排序方法 
reverse()和sort(),这两个方法都会对原数组产生影响。
reverse()反转数组项的顺序
```

 var values = [1, 2, 3, 4, 5];
        values.reverse();
        alert(values);       //5,4,3,2,1

```
默认情况下,sort()方法按升序排列。sort()方法会调用每个对象的toString()方法进行排序判断。
```

        var values = [0, 1, 5, 10, 15];
        values.sort();
        alert(values);    //0,1,10,15,5

```
可以给sort传入一个比较函数:
```

        function compare(value1, value2) {
           /* if (value1 < value2) {
                return -1;
            } else if (value1 > value2) {
                return 1;
            } else {
                return 0;
            }*/
	 return value2-value1;
        }
        
        var values = [0, 1, 5, 10, 15];
        values.sort(compare);
        alert(values);    //0,1,5,10,15

```
###  操作方法concat(),slice() 
concat(),slice()不会影响原数组,splice()会影响原数组
```

        var colors = ["red", "green", "blue"];
        var colors2 = colors.concat("yellow", ["black", "brown"]);
        
        alert(colors);     //red,green,blue        
        alert(colors2);    //red,green,blue,yellow,black,brown

```
```

        var colors = ["red", "green", "blue", "yellow", "purple"];
        var colors2 = colors.slice(1);
        var colors3 = colors.slice(1,4);
        
        alert(colors2);   //green,blue,yellow,purple
        alert(colors3);   //green,blue,yellow

```
```

    
        var colors = ["red", "green", "blue"];
        var removed = colors.splice(0,1);              //remove the first item
        alert(colors);     //green,blue
        alert(removed);    //red - one item array
        
        removed = colors.splice(1, 0, "yellow", "orange");  //insert two items at position 1
        alert(colors);     //green,yellow,orange,blue
        alert(removed);    //empty array

        removed = colors.splice(1, 1, "red", "purple");    //insert two values, remove one
        alert(colors);     //green,red,purple,orange,blue
        alert(removed);    //yellow - one item array

```
###  位置方法indexOf，lastIndexOf 
indexOf，lastIndexOf。如果没有返回-1
###  迭代方法filter,forEach,map 
every()、some()
```

 var numbers = [1,2,3,4,5,4,3,2,1];
        
        var everyResult = numbers.every(function(item, index, array){
            return (item > 2);
        });
        
        alert(everyResult);       //false
        
        var someResult = numbers.some(function(item, index, array){
            return (item > 2);
        });
        
        alert(someResult);       //true


```
filter()
```

        var numbers = [1,2,3,4,5,4,3,2,1];
        
        var filterResult = numbers.filter(function(item, index, array){
            return (item > 2);
        });
        
        alert(filterResult);   //[3,4,5,4,3]

```
map（）、forEach()
```

        var numbers = [1,2,3,4,5,4,3,2,1];
        
        var mapResult = numbers.map(function(item, index, array){
            return item * 2;
        });
        
        alert(mapResult);   //[2,4,6,8,10,8,6,4,2]

	numbers.forEach(function(item, index, array){});//无返回值

```
###  归并方法reduce、reduceRight 
```

        var values = [1,2,3,4,5];
        var sum = values.reduce(function(prev, cur, index, array){
            return prev + cur;
        });
        alert(sum);

```
##  Date类型 
var now=new Date();
new Date(Date.parse("May 25, 2004"));
日期格式化:
getTime()返回毫秒数
getFullYear()返回4位数年份
getMonth()
getDate()返回天数
getDay()返回星期(0-6)
getHours()
getMinutes()
getSeconds()
getMilliseconds()

##  RegExp类型 
var exp=/pattern/flags
正则表达式支持3个标志:
  * g-全局模式
  * i-忽略大小写
  * m-多行模式
所有元字符必须转义:( [ { \ ^ $ | ) ? * + . ] }
**RegExp实例方法:**
exec()、test()
exec()专门为捕获组而设计的，接受一个参数。然后返回一个数组或null。第一项匹配整个模式，其他项是与模式中的捕获组匹配的字符串。
```

        var text = "mom and dad and baby";
        
        var pattern = /mom( and dad( and baby)?)?/gi;
        var matches = pattern.exec(text);
        
        alert(matches.index);    //0
        alert(matches.input);    //"mom and dad and baby"
        alert(matches[0]);       //"mom and dad and baby"
        alert(matches[1]);       //" and dad and baby"
        alert(matches[2]);       //" and baby"

```
test()判断模式是否匹配
```

        var text = "this has been a short summer";
        var pattern = /(.)hort/g;

        if (pattern.test(text)){
            alert(RegExp.input);               //this has been a short summer
            alert(RegExp.leftContext);         //this has been a            
            alert(RegExp.rightContext);        // summer
            alert(RegExp.lastMatch);           //short
            alert(RegExp.lastParen);           //s
            alert(RegExp.multiline);           //false
        }

```
RegExp.$1,RegExp.$2,....RegExp.$9分别用于存储第一、第二。。。第九个匹配的捕获组。

##  Function类型 
函数实例是对象，每个函数都是` Function类型 `的实例。而且具有属性和方法。**由于函数是对象，因此函数名只是一个指向函数对象的指针**。即函数是对象，函数名是指针
函数末尾有一个分号
**函数本身可以作为参数传递，也可以作为返回值被返回。**
**函数没有重载。**原因：函数名是指针，函数调用传递参数随意

函数声明提升：
```

alert(sum(10,10)); //运行正确
function sum(num1,num2){
	return num1+num2;
};

alert(sum2(10,10);//运行错误
var sum2=function(num1,num2){
	return num1+num2;
};

```
###  函数内部属性arguments和this 
**在函数内部有两个特殊的对象:arguments和this。**其中arguments是一个类数组对象，包含着传入函数中的所有参数。
` this在函数定义时并不能确定，具体确定在函数被调用的时候。在不同的执行环境中，this的值不同。this代表了调用函数的对象。 `
```

        window.color = "red";
        var o = { color: "blue" };
        
        function sayColor(){
            alert(this.color);
        }
        
        sayColor();     //red
        
        o.sayColor = sayColor;
        o.sayColor();   //blue

```
###  函数属性和方法 
每个函数都包含两个属性**length和prototype**，其中length表示期望接受参数的个数。
prototype是保存所有实例方法的所在。在创建自定义引用类型以及实现继承时，prototype属性是极为重要的。
每个函数都包含三个非继承而来的方法:**apply(),call(),bind()**，他们的用途都是在特定的作用域中调用函数，**实际上等于设置函数体内this对象的值。**
```

        function sum(num1, num2){
            return num1 + num2;
        }
        
        function callSum1(num1, num2){
            return sum.apply(this, arguments);
        }
        
        function callSum2(num1, num2){
            return sum.apply(this, [num1, num2]);
        }
        
        alert(callSum1(10,10));   //20
        alert(callSum2(10,10));   //20


```
```

        window.color = "red";
        var o = { color: "blue" };
        
        function sayColor(){
            alert(this.color);
        }
        
        sayColor();            //red
        
        sayColor.call(this);   //red
        sayColor.call(window); //red
        sayColor.call(o);      //blue

```
```

        window.color = "red";
        var o = { color: "blue" };
                           
        function sayColor(){
            alert(this.color);
        }
        var objectSayColor = sayColor.bind(o);//不是立即执行。这是与call()和apply()的区别
        objectSayColor();   //blue

```
##  基本包装类型 
```

Number: 10.005.toFixed(2) //10.01
String: "hello ".concat("world") ; //hello world
String:"hello ".concat("world","!") ; //hello world!
        var stringValue = "hello world";
        alert(stringValue.slice(3));        //"lo world"
        alert(stringValue.substring(3));    //"lo world"
        alert(stringValue.substr(3));       //"lo world"
        alert(stringValue.slice(3, 7));     //"lo w"
        alert(stringValue.substring(3,7));  //"lo w"
        alert(stringValue.substr(3, 7));    //"lo worl"
        
        alert(stringValue.slice(-3));         //"rld"
        alert(stringValue.substring(-3));     //"hello world"
        alert(stringValue.substr(-3));        //"rld"
        alert(stringValue.slice(3, -4));      //"lo w"
        alert(stringValue.substring(3, -4));  //"hel"
        alert(stringValue.substr(3, -4));     //"" (empty string)

//trim(), indexOf() ,lastIndexOf();
//toLowerCase(),toUpperCase()

```
###  字符串模式匹配方法 
```

       var text = "cat, bat, sat, fat"; 
        var pattern = /.at/;
        
        var matches = text.match(pattern);        
        alert(matches.index);        //0
        alert(matches[0]);           //"cat"
        alert(pattern.lastIndex);    //0

        var pos = text.search(/at/);
        alert(pos);   //1

        var result = text.replace("at", "ond");
        alert(result);    //"cond, bat, sat, fat"

        result = text.replace(/at/g, "ond");
        alert(result);    //"cond, bond, sond, fond"

        result = text.replace(/(.at)/g, "word ($1)");
        alert(result);    //word (cat), word (bat), word (sat), word (fat)

```

##  单体内置对象 
内置对象由ES提供，不依赖于宿主环境的对象，这些对象在ES程序执行之前就已经存在了。
Global与Math内置对象。
Global内置对象定义了有用的方法:
  * isNaN(),isFinite(),parseInt,parseFloat()
  * encodeURI(),encodeURIComponent()
  * decodeURI(),decodeURIComponent()
  * eval()
Global对象属性
在浏览器环境下Global对象是作为window对象的一部分而加以实现的。

Math对象属性:
Math.PI,Math.E
Math对象方法:
min(),max()
舍入方法：ceil()一律入,floor()一律舍,round()标准四舍五入
```

        alert(Math.ceil(25.9));     //26
        alert(Math.round(25.9));    //26                
        alert(Math.floor(25.9));    //25

```
random()随机数:Math.random()*10+1


