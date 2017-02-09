title: js迭代对象属性 

#  js迭代对象属性 
```

fontPicker.styleToString=function(style){
		var str='';
		for(var name in style){
			str+=name+":"+style[name]+";";
		}
		return str;
	}

```
参考：http://my.oschina.net/liangrockman/blog/138557