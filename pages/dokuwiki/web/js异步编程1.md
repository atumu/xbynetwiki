title: js异步编程1 

#  JS Ajax异步编程之Promise线性化 
##  Promise/Deferred概述 
**Promise（或者Deferred）对象**
在说Promise之前，不得不说一下**JavaScript的嵌套的回调函数**
在JavaScript语言中，无论是写浏览器端的各种事件处理回调、ajax回调，还是写Node.js上的业务逻辑，不得不面对的问题就是各种回调函数。
回调函数少了还好，一旦多了起来而且必须讲究执行顺序的话，回调函数开始嵌套，那代码的恶心程度是**相当不符合常人的线性思维的**。

` Promise的意义就在于 then 链式调用 ，它避免了异步函数之间的层层嵌套，将原来异步函数的 嵌套关系 转变为便于阅读和理解的 链式步骤关系 ，以符合常人的线性思维。 `
Promise的主要用法就是将各个异步操作封装成好多Promise，而一个Promise只处理一个异步逻辑。最后将各个Promise用链式调用写法串联，在这样处理下，如果异步逻辑之间前后关系很重的话，你也不需要层层嵌套，只需要把每个异步逻辑封装成Promise链式调用就可以了。

Promise是CommonJS的规范之一，拥有resolve、reject、done、fail、then、when等方法，**能够帮助我们控制代码的流程，避免函数的多层嵌套。**如今异步在web开发中越来越重要，
对于开发人员来说，这种非线性执行的编程会让开发者觉得难以掌控，**而Promise可以让我们更好地掌控代码的执行流程,jQuery等流行的js库都已经实现了这个对象，最新的ES6也将原生实现Promise**
**想象这样一个场景，两个异步请求，第二个需要用到第一个请求成功的数据**，那么我们代码可以这样写
```

    ajax({
        url: url1,
        success: function(data) {
            ajax({
                url: url2,
                data: data,
                success: function() {
                }
            });
        }
    });

```
**如果继续下去在回调函数中进行下一步操作，嵌套的层数会越来越多。**我们可以进行适当的改进，把回调函数写到外面
```

    function A() {
        ajax({
            url: url1,
            success: function(data) {
                B(data);
            }
        });
    }
    function B(data) {
        ajax({
            url: url2,
            success: function(data) {
                ......
            }
        });
    }

```
即使是改写成这样，代码还是不够直观，但是如果有了Promise对象，代码就可以写得非常清晰，一目了然，请看
new Promise(A).done(B);
这样函数B就不用写在A的回调中了

目前流行的ES5标准中还未支持Promise对象。但是已有很多三方库支持了如jQuery,promise.js等。思路大致是这样的，用2个数组(doneList和failList)分别存储成功时的回调函数队列和失败时的回调队列
* state: 当前执行状态，有pending、resolved、rejected3种取值
* done: 向doneList中添加一个成功回调函数
* fail: 向failList中添加一个失败回调函数
* then: 分别向doneList和failList中添加回调函数
* always: 添加一个无论成功还是失败都会调用的回调函数
* resolve: 将状态更改为resolved,并触发绑定的所有成功的回调函数
* reject: 将状态更改为rejected,并触发绑定的所有失败的回调函数
* **when: 参数是多个异步或者延迟函数，返回值是一个Promise兑现，当所有函数都执行成功的时候执行该对象的resolve方法，反之执行该对象的reject方法**

参考：https://segmentfault.com/a/1190000000684654
https://segmentfault.com/a/1190000002395343
http://www.infoq.com/cn/news/2011/09/js-promise/

##  ES6-Promise语法 
ES6新标准实现了对Promise的原生支持。
ES6 Promise语法参考：
中文:http://es6.ruanyifeng.com/#docs/promise
英文:http://www.html5rocks.com/en/tutorials/es6/promises/
目前IE9以下对ES6新标准支持度不是很好。所以如果我们需要使用第三方库来实现跨浏览器的promise语法。
第三方兼容库:https://github.com/stefanpenner/es6-promise
与原生使用的一点小区别:
使用:
```

require(['es6-promise'],function(ES6Promise)){
	var Promise=ES6Promise.Promise
	new Promise(function(resolve, reject) {
    		//...
  	});
});

```
```

require(['es6-promise'],function(ES6Promise)){
	new ES6Promise.Promise(function(resolve, reject) {
    		//...
  	});
});

```
IE9以下不支持catch子句的解决方案:
```

promise['catch'](function(err) {
  // ...
});
Or use .then instead:
promise.then(undefined, function(err) {
  // ...
});

```
**Promise是异步编程的一种解决方案，**比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，**ES6将其写进了语言标准，统一了用法，原生提供了Promise对象。**
Promise对象有以下两个特点。
（1）` 对象的状态不受外界影响 `。Promise对象代表一个异步操作，**有三种状态：Pending（进行中）、Resolved（已完成，又称Fulfilled）和Rejected（已失败）。**只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。**Promise对象的状态改变，只有两种可能：从Pending变为Resolved和从Pending变为Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。**就算改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。
` 有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。 `此外，Promise对象提供统一的接口，使得控制异步操作更加容易。

**Promise也有一些缺点。**
  * 首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。
  * 其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。
  * 第三，当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。
如果某些事件不断地反复发生，一般来说，使用stream模式是比部署Promise更好的选择。

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。
下面代码创造了一个Promise实例。
```

var promise = new Promise(function(resolve, reject) {
  // ... some code
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

```
**Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由JavaScript引擎提供，不用自己部署。**
  * ` resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从Pending变为Resolved）， `在异步操作成功时调用，**并将异步操作的结果，作为参数传递出去**；
  * ` reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从Pending变为Rejected `），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

Promise实例生成以后，可以用**then方法**分别指定Resolved状态和Reject状态的回调函数。
```

promise.then(function(value) {
  // success
}, function(value) {
  // failure
});

```
then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为Resolved时调用，第二个回调函数是Promise对象的状态变为Reject时调用。
其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。
` **Promise新建后就会立即执行。** `
```

let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('Resolved.');
});
console.log('Hi!');

// Promise
// Hi!
// Resolved

```
上面代码中，Promise新建后立即执行，所以首先输出的是“Promise”。然后，then方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以“Resolved”最后输出。
##  Promise异步加载图片的例子 
```

function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    var image = new Image();
    image.onload = function() {
      resolve(image);
    };
    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };
    image.src = url;
  });
}

```
##  用Promise对象实现的Ajax操作的例子 
```

var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();
    function handler() {
      if ( this.readyState !== 4 ) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);  //记得调用
      } else {
        reject(new Error(this.statusText)); //记得调用
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});

```
上面代码中，getJSON是对XMLHttpRequest对象的封装，用于发出一个针对JSON数据的HTTP请求，并且返回一个Promise对象。需要注意的是，在getJSON内部，resolve函数和reject函数调用时，都带有参数。
**如果调用resolve函数和reject函数时带有参数，那么它们的参数会被传递给回调函数。
reject函数的参数通常是Error对象的实例，表示抛出的错误；
resolve函数的参数除了正常的值以外，还可能是另一个Promise实例，表示异步操作的结果有可能是一个值，也有可能是另一个异步操作，比如像下面这样。**
```

var p1 = new Promise(function(resolve, reject){
  // ...
});

var p2 = new Promise(function(resolve, reject){
  // ...
  resolve(p1);
})

```
上面代码中，p1和p2都是Promise的实例，但是p2的resolve方法将p1作为参数，**即一个异步操作的结果是返回另一个异步操作**。
**注意，这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。**如果p1的状态是Pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是Resolved或者Rejected，那么p2的回调函数将会立刻执行。
```

var p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})
var p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})
p2.then(result => console.log(result))
p2.catch(error => console.log(error))
// Error: fail

```
上面代码中，p1是一个Promise，3秒之后变为rejected。p2的状态由p1决定，2秒之后，**p2调用resolve方法，但是此时p1的状态还没有改变，因此p2的状态也不会变。**又过了2秒，p1变为rejected，p2也跟着变为rejected。

##  Promise.prototype.then() 
Promise实例具有then方法，也就是说，then方法是定义在原型对象Promise.prototype上的。**它的作用是为Promise实例添加状态改变时的回调函数。**前面说过，then方法的第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。
` then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。 `
```

getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});

```
采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。
```

getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});

```
上面代码中，第一个then方法指定的回调函数，返回的是另一个Promise对象。这时，第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为Resolved，就调用funcA，如果状态变为Rejected，就调用funcB。


##  Promise.prototype.catch() 
**Promise.prototype.catch方法是` .then(null, rejection) `的别名，用于指定发生错误时的回调函数。**
```

getJSON("/posts.json").then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});

```
上面代码中，getJSON方法返回一个Promise对象，如果该对象状态变为Resolved，则会调用then方法指定的回调函数；如果异步操作抛出错误，状态就会变为Rejected，就会调用catch方法指定的回调函数，处理这个错误。
```

p.then((val) => console.log("fulfilled:", val))
  .catch((err) => console.log("rejected:", err));
// 等同于
p.then((val) => console.log(fulfilled:", val))
  .then(null, (err) => console.log("rejected:", err));

```
比较上面两种写法，可以发现reject方法的作用，等同于抛出错误。如果Promise状态已经变成Resolved，再抛出错误是无效的。
```

var promise = new Promise(function(resolve, reject) {
  resolve("ok");
  throw new Error('test');
});
promise
  .then(function(value) { console.log(value) })
  .catch(function(error) { console.log(error) });
// ok

```
上面代码中，Promise在resolve语句后面，再抛出错误，不会被捕获，等于没有抛出。
` Promise对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。 `
跟传统的try/catch代码块不同的是，如果没有使用catch方法指定错误处理的回调函数，**Promise对象抛出的错误不会传递到外层代码**，即不会有任何反应。
```

var someAsyncThing = function() {
  return new Promise(function(resolve, reject) {
    // 下面一行会报错，因为x没有声明
    resolve(x + 2);
  });
};
someAsyncThing().then(function() {
  console.log('everything is great');
});

```
上面代码中，someAsyncThing函数产生的Promise对象会报错，**但是由于没有指定catch方法，这个错误不会被捕获，也不会传递到外层代码，导致运行后没有任何输出。**注意，Chrome浏览器不遵守这条规定，它会抛出错误“ReferenceError: x is not defined”。
需要注意的是，**catch方法返回的还是一个Promise对象，因此后面还可以接着调用then方法。**
catch方法之中，还能再抛出错误。

##  Promise.all() 
**Promise.all方法用于将多个Promise实例，包装成一个新的Promise实例。**
```

var p = Promise.all([p1, p2, p3]);

```
上面代码中，Promise.all方法接受一个数组作为参数，p1、p2、p3都是Promise对象的实例，如果不是，就会先调用下面讲到的Promise.resolve方法，将参数转为Promise实例，再进一步处理。（Promise.all方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例。）
p的状态由p1、p2、p3决定，分成两种情况。
（1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
（2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。
下面是一个具体的例子。
```

// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});

```
上面代码中，promises是包含6个Promise实例的数组，只有这6个实例的状态都变成fulfilled，或者其中有一个变为rejected，才会调用Promise.all方法后面的回调函数。

##  Promise.race() 
**Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例。**
var p = Promise.race([p1,p2,p3]);
上面代码中，**只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的Promise实例的返回值，就传递给p的回调函数。**
Promise.race方法的参数与Promise.all方法一样，如果不是Promise实例，就会先调用下面讲到的Promise.resolve方法，将参数转为Promise实例，再进一步处理。
下面是一个例子，如果指定时间内没有获得结果，就将Promise的状态变为reject，否则变为resolve。

##  Promise.resolve() 
**有时需要将现有对象转为Promise对象，Promise.resolve方法就起到这个作用。**
```

var jsPromise = Promise.resolve($.ajax('/whatever.json'));

```
上面代码将jQuery生成的deferred对象，转为一个新的Promise对象。
Promise.resolve方法的参数分成四种情况。
（1）参数是一个Promise实例
如果参数是Promise实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。
（2）参数是一个thenable对象
thenable对象指的是具有then方法的对象。
（3）参数不是具有then方法的对象，或根本就不是对象
如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为Resolved。
```

var p = Promise.resolve('Hello');
p.then(function (s){
  console.log(s)
});
// Hello

```
上面代码生成一个新的Promise对象的实例p。由于字符串Hello不属于异步操作（判断方法是它不是具有then方法的对象），返回Promise实例的状态从一生成就是Resolved，所以回调函数会立即执行。Promise.resolve方法的参数，会同时传给回调函数。
（4）不带有任何参数
Promise.resolve方法允许调用时不带参数，直接返回一个Resolved状态的Promise对象。

##  Promise.reject() 
Promise.reject(reason)方法也会返回一个新的Promise实例，该实例的状态为rejected。它的参数用法与Promise.resolve方法完全一致。
```

var p = Promise.reject('出错了');
// 等同于
var p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s){
  console.log(s)
});
// 出错了

```
上面代码生成一个Promise对象的实例p，状态为rejected，回调函数会立即执行。

##  done() 
**Promise对象的回调链，不管以then方法或catch方法结尾，要是最后一个方法抛出错误，都有可能无法捕捉到（因为Promise内部的错误不会冒泡到全局）。因此，我们可以提供一个done方法，总是处于回调链的尾端，保证抛出任何可能出现的错误。**
```

asyncFunc()
  .then(f1)
  .catch(r1)
  .then(f2)
  .done();

```
它的实现代码相当简单。
```

Promise.prototype.done = function (onFulfilled, onRejected) {
  this.then(onFulfilled, onRejected)
    .catch(function (reason) {
      // 抛出一个全局错误
      setTimeout(() => { throw reason }, 0);
    });
};

```
从上面代码可见，done方法的使用，可以像then方法那样用，**提供Fulfilled和Rejected状态的回调函数**，也可以不提供任何参数。但不管怎样，done都会捕捉到任何可能出现的错误，并向全局抛出。
##  finally() 
**finally方法用于指定不管Promise对象最后状态如何，都会执行的操作。它与done方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。**
下面是一个例子，服务器使用Promise处理请求，然后使用finally方法关掉服务器。
不管前面的Promise是fulfilled还是rejected，都会执行回调函数callback。

##  加载图片 
我们可以将图片的加载写成一个Promise，一旦加载完成，Promise的状态就发生变化。
```

const preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    var image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};

```