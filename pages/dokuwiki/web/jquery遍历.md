title: jquery遍历 

#  jQuery遍历 
这些 jQuery 方法很有用，它们用于向上遍历 DOM 树：
parent()
parents()
parentsUntil()
下面是两个用于向下遍历 DOM 树的 jQuery 方法：
children()
find()
在 DOM 树中水平遍历
有许多有用的方法让我们在 DOM 树进行水平遍历：
siblings()
next()
nextAll()
nextUntil()
prev()
prevAll()
prevUntil()
**jQuery 遍历- 过滤**
```

$(document).ready(function(){
  $("div p").first();
});
$(document).ready(function(){
  $("div p").last();
});

```

**jQuery eq() 方法**
eq() 方法返回被选元素中带有指定索引号的元素。
索引号从 0 开始，因此首个元素的索引号是 0 而不是 1。下面的例子选取第二个 <p> 元素（索引号 1）：
```

$(document).ready(function(){
  $("p").eq(1);
});

```

**jQuery filter() 方法**
filter() 方法允许您规定一个标准。不匹配这个标准的元素会被从集合中删除，匹配的元素会被返回。
下面的例子返回带有类名 "intro" 的所有 <p> 元素：
```

$(document).ready(function(){
  $("p").filter(".intro");
});

```

**jQuery not() 方法**
not() 方法返回不匹配标准的所有元素。
提示：not() 方法与 filter() 相反。
下面的例子返回不带有类名 "intro" 的所有 <p> 元素：
```

$(document).ready(function(){
  $("p").not(".intro");
});

```