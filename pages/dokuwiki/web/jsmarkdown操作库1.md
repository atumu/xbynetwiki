title: jsmarkdown操作库1 

#  jsMarkDown操作库marked.js 
github:https://github.com/chjj/marked
A full-featured markdown parser and compiler, written in JavaScript. Built for speed.
实现解析了[GFM features](https://help.github.com/categories/writing-on-github/)特性
```

npm install marked --save

```
##  基本使用 
```

var marked = require('marked');
console.log(marked('I am using __markdown__.'));
// Outputs: <p>I am using <strong>markdown</strong>.</p>
Example setting options with default values:

var marked = require('marked');
marked.setOptions({
  renderer: new marked.Renderer(),
  gfm: true,
  tables: true,
  breaks: false,
  pedantic: false,
  sanitize: true,
  smartLists: true,
  smartypants: false
});
console.log(marked('I am using __markdown__.'));

```
```

<!doctype html>
<html>
<head>
  <meta charset="utf-8"/>
  <title>Marked in the browser</title>
  <script src="lib/marked.js"></script>
</head>
<body>
  <div id="content"></div>
  <script>
    document.getElementById('content').innerHTML =
      marked('# Marked in browser\n\nRendered by **marked**.');
  </script>
</body>
</html>

```
##  参数与说明 
marked(markdownString [,options] [,callback])
markdownString
Type: string
String of markdown source to be compiled.

options
Type: object
Hash of options. Can also be set using the`  marked.setOptions ` method as seen above.

callback
Type: function
Function called when the markdownString has been fully parsed **when using async highlighting**. If the options argument is omitted, this can be used as the second argument.

###  异步高亮与回调 
使用https://github.com/isagalaev/highlight.js实现
```

var marked = require('marked');
var markdownString = '```js\n console.log("hello"); \n```';
// Using async version of marked
marked(markdownString, function (err, content) {
  if (err) throw err;
  console.log(content);
});

// Synchronous highlighting with highlight.js
marked.setOptions({
  highlight: function (code) {
    return require('highlight.js').highlightAuto(code).value;
  }
});
console.log(marked(markdownString));

```
highlight函数接受的参数：
code
Type: string
The section of code to pass to the highlighter.

lang
Type: string
The programming language specified in the code block.

callback
Type: function
The callback function to call when using an async highlighter.