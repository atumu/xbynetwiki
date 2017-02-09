title: ant风格路径表达式 

#  Ant风格路径表达式 
官方文档：http://ant.apache.org/manual/dirtasks.html#patterns
不论是Spring还是Maven的xml配置文件，普遍采用Ant风格路径表达式。所以有必要了解一点。
ANT通配符有三种：
通配符	说明
```

?	匹配任何单字符
*	匹配0或者任意数量的字符
**	匹配0或者更多的目录

```
例子：
URL路径	说明
```

/app/*.x	匹配(Matches)所有在app路径下的.x文件
/app/p?ttern	匹配(Matches) /app/pattern 和 /app/pXttern,但是不包括/app/pttern
/**/example	匹配(Matches) /app/example, /app/foo/example, 和 /example
/app/**/dir/file.*	匹配(Matches) /app/dir/file.jsp, /app/foo/dir/file.html,/app/foo/bar/dir/file.pdf, 和 /app/dir/file.java
/**/*.jsp	匹配(Matches)任何的.jsp 文件

```
属性：
**最长匹配原则(has more characters)**
```

说明，URL请求/app/dir/file.jsp，现在存在两个路径匹配模式/**/*.jsp和/app/dir/*.jsp，那么会根据模式/app/dir/*.jsp来匹配

```
参考：http://blog.csdn.net/songdexv/article/details/7219686