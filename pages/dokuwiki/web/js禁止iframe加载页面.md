title: js禁止iframe加载页面 

#  js禁止别人以iframe加载你的页面 
下面的代码已经不言自明了，没什么好多说的。
```

if (window.location != window.parent.location) window.parent.location = window.location;

```