title: js模版引擎mustache 

#  js模版引擎mustache 
项目地址：http://mustache.github.io/#demo
文档地址：http://mustache.github.io/mustache.5.html
Mustache是基于JavaScript实现的模版引擎，类似于JQuery Template，但是这个模版更加的轻量级，语法更加的简单易用，很容易上手。不过它同时还有相应的Java、PHP等版本。
##  模版语法 
示列：
```

一个模版写法如下：
Hello ![](/data/dokuwikiname)
You have just won ![](/data/dokuwikivalue) dollars!
![](/data/dokuwiki#in_ca)
Well, ![](/data/dokuwikitaxed_value) dollars, after taxes.
![](/data/dokuwiki/in_ca)

指定填充模版的json数据。
{
  "name": "Chris",
  "value": 10000,
  "taxed_value": 10000 - (10000 * 0.4),
  "in_ca": true
}
产生的输出。

Hello Chris
You have just won 10000 dollars!
Well, 6000.0 dollars, after taxes.

```

**1.简单的变量绑定：&lt;nowiki&gt;![](/data/dokuwikiname)&lt;/nowiki&gt;**
```

1 var data = { "name": "Willy" };
2 Mustache.render（"![](/data/dokuwikiname) is awesome."，data）;
返回成果 Willy is awesome.

 function show(t) {
 2                 $("#content").html(t);
 3             }
 4 
 5             var view = {
 6                 title: 'YZF',
 7                 cacl: function () {
 8                     return 6 + 4;
 9                 }
10             };
11             $("#content").html(Mustache.render("![](/data/dokuwikititle) spends ![](/data/dokuwikicacl)", view));
结果：YZF spends 10

```


**1.1绑定子属性**
```

  var view = {
      "name": {
          first: "Y",
          second: "zf"
      },
      "age": 21
  };
  show(Mustache.render("![](/data/dokuwikiname.first)![](/data/dokuwikiname.second) age is ![](/data/dokuwikiage)", view));

```

**2.若是变量含有html的代码的，例如：<br>、<tr>等等而不想转义可以在用&lt;nowiki&gt;{![](/data/dokuwikiname)}或![](/data/dokuwiki&amp;name)&lt;/nowiki&gt;** （默认将"<"和">"转义）
```

var data = {
            "name" : "<br>Willy<br>"
        };
        var output = Mustache.render（"{![](/data/dokuwikiname)} is awesome."， data）;
        console.log（output）;

成果：<br>Willy<br> is awesome.
如果用![](/data/dokuwikiname)的成果是转义为：&lt;br&gt;Willy&lt;br&gt; is awesome.（默认将"<"和">"转义）

```

**3.&lt;nowiki&gt;![](/data/dokuwiki＃param)内容![](/data/dokuwiki/param)&lt;/nowiki&gt;这个语法很强大，有if断定、forEach的功能。**
```

var data = {
                "nothin":true
            };
            var output = Mustache.render（
                    "Shown.![](/data/dokuwiki＃nothin)Never shown!![](/data/dokuwiki/nothin)"， data）;
          console.log（output）;
若是nothin是空或者null，或者是false都会输出Shown.相反则是Shown.Never shown!。

迭代，传入参数为对象数组时
  var data = {
              "stooges" : [ {
                  "name" : "Moe"
              }， {
                  "name" : "Larry"
              }， {
                  "name" : "Curly"
              } ]
          };
         var output = Mustache.render（"![](/data/dokuwiki＃stooges)<b>![](/data/dokuwikiname)</b>![](/data/dokuwiki/stooges)"，
                 data）;
         console.log（output）;
输出：
  <b>Moe</b>
  <b>Larry</b>
  <b>Curly</b>

若是迭代的是数组，还可以用&lt;nowiki&gt;![](/data/dokuwiki.)&lt;/nowiki&gt;来调换每个元素

var data = {
            "musketeers" : [ "Athos"， "Aramis"， "Porthos"， "D""Artagnan" ]
        };
        var output = Mustache.render（"![](/data/dokuwiki＃musketeers)* ![](/data/dokuwiki&.)![](/data/dokuwiki/musketeers)"，
                data）;
        console.log（output）;
输出：* Athos
        * Aramis
        * Porthos
        * D""Artagnan

```

**循环输出指定函数处理后返回的值**
```

 1             var view = {
 2                 "beatles": [
 3                     { "firstname": "Johh", "lastname": "Lennon" },
 4                     { "firstname": "Paul", "lastname": "McCartney" }
 5                 ],
 6                 "name": function () {
 7                     return this.firstname + this.lastname;
 8                 }
 9             };
10             show(Mustache.render("![](/data/dokuwiki#beatles)![](/data/dokuwikiname)<br />![](/data/dokuwiki/beatles)", view));

```

**部分模板**
```

 1             var view = {
 2                 names: [
 3                     { "name": "y" },
 4                     { "name": "z" },
 5                     { "name": "f" }
 6                 ]
 7             };
 8             var base = "<h2>Names</h2>![](/data/dokuwiki#names)![](/data/dokuwiki>user)![](/data/dokuwiki/names)";
 9             var name = "<b>![](/data/dokuwikiname)</b>";
10             show(Mustache.render(base, view, { user: name }));

```
我们定义了很多模板，但是彼此之间无法互相嵌套使用，也会造成繁琐。这里使用其他模板的方式仅仅是&lt;nowiki&gt;![](/data/dokuwiki&gt;templetename)&lt;/nowiki&gt;。
最大的不同就是Mustache.render方法有了第三个参数。

9.&lt;nowiki&gt;![](/data/dokuwiki^)与![](/data/dokuwiki＃)&lt;/nowiki&gt;相反，若是变量是null、undefined、 false、和空数组讲输出成果
10.&lt;nowiki&gt;![](/data/dokuwiki!  )&lt;/nowiki&gt;注释

**预编译模板**
```

1             Mustache.parse(template);
2             //其他代码
3             Mustache.render(template,view);

```
模板既然有好处，也有坏处。就是编译模板需要时间，所以在我们已知会使用某个模板的前提下，我们可以预先编译好这个模板，以便后面的使用。