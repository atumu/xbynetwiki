title: txtmark_markdown 

txtmark是最快的java markdown解析器，基于它的开源库还有markdown4j。
项目地址：https://github.com/rjeschke/txtmark/
maven中心：http://search.maven.org/#search%7Cga%7C1%7Ctxtmark
使用：
```

String result = txtmark.Processor.process("This is ***TXTMARK***");

```
启用扩展
在markdown文件中添加下面一行:
```

[$PROFILE$]: extended

```