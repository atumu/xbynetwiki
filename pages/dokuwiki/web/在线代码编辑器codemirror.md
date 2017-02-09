title: 在线代码编辑器codemirror 

#  代码编辑器库CodeMirror 
github:https://github.com/codemirror/codemirror
官网:http://codemirror.net/
##  快速入门 
首先，要引用是 lib 目录下的 codemirror.js，还有一个就是同目录下的codemirror.css 文件
```

<script src="lib/codemirror.js"></script>
<link rel="stylesheet" href="/lib/codemirror.css">

```
接下来要引用的就是在mode目录下编辑器中要编辑的语言对应的js文件，下面以js文件为例：
```

<script src="mode/javascript/javascript.js"></script>

```
引用的文件用于支持对应语言的语法高亮。然后，调用脚本以创建编辑器：
```

var myCodeMirror = CodeMirror(document.body);

```
这里的调用会在body中添加编辑器，这里因为直接在上面引用了javascript.js，所以这个编辑器会对javascript的关键字高亮显示。
想要高级一点，给编辑器添加一些元素，也可以通过传入配置参数来实现。
```

var myCodeMirror = CodeMirror(document.body,{
lineNumbers: true
});

```
这样，就给编辑器添加了行号。

上面说的是实现编辑器的最简单的方式，然后在实际项目中，一般都不会直接把body作为编辑器的容器。而最常用的，是使用textarea。
**要把 textarea 实现成一个支持高亮的编辑器，CodeMirror 提供了非常简单的方法**：
```

<textarea id="editor" name="editor"></textarea>

var myTextarea = document.getElementById('editor');
var CodeMirrorEditor = CodeMirror.fromTextArea(myTextarea, {
    mode: "text/javascript",
lineNumbers: true
});

```

##  MarkDown版本实现 
markdown版本实现editor.js:http://lab.lepture.com/editor/
改造参考文章:https://segmentfault.com/a/1190000002724430
使用：
```

// init editor    
            var editorObj = $("#editor1");

            if(editorObj.length > 0){
                
                var editor = new Editor({
                    element: editorObj.get(0),
                    status:false
                });
                editor.render();
            }

```
预加载内容：
```

$('#editor').load('markdown.md',
        function(data) {
          var editor = new Editor();
          editor.render();
        }
      );

```
另一个流行开源的markdown编辑器stackedit:https://github.com/benweet/stackedit/
##  配置说明 
在使用CodeMirror的时候，不管是直接使用 CodeMirror() 还是使用`  fromTextArea() ` ，都可以通过传递第二个参数来配置编辑器。
使用方法如下：
```

var myCodeMirror = CodeMirror(el, {
    // options...
});

```
或者
```

var myCodeMirror = CodeMirror.fromTextArea(el, {
    // options...
});

```
options 可以使用的参数
value: string | CodeMirror.Doc
编辑器的初始值（文本），可以是字符串或者CodeMirror文档对象(不同于HTML文档对象)。

mode: string | object
通用的或者在CodeMirror中使用的与mode相关联的mime，当不设置这个值的时候，会默认使用第一个载入的mode定义文件。一般地，会使用关联的mime类型来设置这个值；除此之外，也可以使用一个带有name属性的对象来作为值（如：{name: “javascript”, json: true}）。可以通过访问CodeMirror.modes和CodeMirror.mimeModes获取定义的mode和MIME。

lineSeparator: string|null
明确指定编辑器使用的行分割符（换行符）。默认（值为null）情况下，文档会被 CRLF(以及单独的CR, LF)分割，单独的LF会在所有的输出中用作换行符（如：getValue）。当指定了换行字符串，行就只会被指定的串分割。

theme: string
配置编辑器的主题样式。要使用主题，必须保证名称为 .cm-s-[name] (name是设置的theme的值)的样式是加载上了的。当然，你也可以一次加载多个主题样式，使用方法和html和使用类一样，如： theme: foo bar，那么此时需要cm-s-foo cm-s-bar这两个样式都已经被加载上了。

indentUnit: integer
缩进单位，值为空格数，默认为2 。

smartIndent: boolean
自动缩进，设置是否根据上下文自动缩进（和上一行相同的缩进量）。默认为true。

tabSize: integer
tab字符的宽度，默认为4 。

indentWithTabs: boolean
在缩进时，是否需要把 n*tab宽度个空格替换成n个tab字符，默认为false 。

electricChars: boolean
在输入可能改变当前的缩进时，是否重新缩进，默认为true （仅在mode支持缩进时有效）。

specialChars: RegExp
需要被占位符(placeholder)替换的特殊字符的正则表达式。最常用的是非打印字符。默认为：/[\u0000-\u0019\u00ad\u200b-\u200f\u2028\u2029\ufeff]/。

specialCharPlaceholder: function(char) → Element
这是一个接收由specialChars选项指定的字符作为参数的函数，此函数会产生一个用来显示指定字符的DOM节点。默认情况下，显示一个红点（•），这个红点有一个带有前面特殊字符编码的提示框。

rtlMoveVisually: boolean
Determines whether horizontal cursor movement through right-to-left (Arabic, Hebrew) text is visual (pressing the left arrow moves the cursor left) or logical (pressing the left arrow moves to the next lower index in the string, which is visually right in right-to-left text). The default is false on Windows, and true on other platforms.（这段完全不晓得搞啥子鬼）

keyMap: string
配置快捷键。默认值为default，即 codemorrir.js 内部定义。其它在key map目录下。

extraKeys: object
给编辑器绑定与前面keyMap配置不同的快捷键。

lineWrapping: boolean
在长行时文字是换行(wrap)还是滚动(scroll)，默认为滚动(scroll)。

lineNumbers: boolean
是否在编辑器左侧显示行号。

firstLineNumber: integer
行号从哪个数开始计数，默认为1 。

lineNumberFormatter: function(line: integer) → string
使用一个函数设置行号。

gutters: array<string>
用来添加额外的gutter（在行号gutter前或代替行号gutter）。值应该是CSS名称数组，每一项定义了用于绘制gutter背景的宽度（还有可选的背景）。为了能明确设置行号gutter的位置（默认在所有其它gutter的右边），也可以包含CodeMirror-linenumbers类。类名是用于传给setGutterMarker的键名(keys)。

fixedGutter: boolean
设置gutter跟随编辑器内容水平滚动（false）还是固定在左侧（true或默认）。

scrollbarStyle: string
设置滚动条。默认为”native”，显示原生的滚动条。核心库还提供了”null”样式，此样式会完全隐藏滚动条。Addons可以设置更多的滚动条模式。

coverGutterNextToScrollbar: boolean
当fixedGutter启用，并且存在水平滚动条时，在滚动条最左侧默认会显示gutter，当此项设置为true时，gutter会被带有CodeMirror-gutter-filler类的元素遮挡。
inputStyle: string
选择CodeMirror处理输入和焦点的方式。核心库定义了textarea和contenteditable输入模式。在移动浏览器上，默认是contenteditable，在桌面浏览器上，默认是textarea。在contenteditable模式下对IME和屏幕阅读器支持更好。

readOnly: boolean|string
编辑器是否只读。如果设置为预设的值 “nocursor”，那么除了设置只读外，编辑区域还不能获得焦点。

showCursorWhenSelecting: boolean
在选择时是否显示光标，默认为false。

lineWiseCopyCut: boolean
启用时，如果在复制或剪切时没有选择文本，那么就会自动操作光标所在的整行。

undoDepth: integer
最大撤消次数，默认为200（包括选中内容改变事件） 。

historyEventDelay: integer
在输入或删除时引发历史事件前的毫秒数。

tabindex: integer
编辑器的tabindex。

autofocus: boolean
是否在初始化时自动获取焦点。默认情况是关闭的。但是，在使用textarea并且没有明确指定值的时候会被自动设置为true。

低级选项

下面的选项仅用于一些特殊情况。

dragDrop: boolean
是否允许拖放，默认为true。

allowDropFileTypes: array<string>
默认为null。当设置此项时，只接收包含在此数组内的文件类型拖入编辑器。文件类型为MIME名称。

cursorBlinkRate: number
光标闪动的间隔，单位为毫秒。默认为530。当设置为0时，会禁用光标闪动。负数会隐藏光标。

cursorScrollMargin: number
当光标靠近可视区域边界时，光标距离上方和下方的距离。默认为0 。

cursorHeight: number
光标高度。默认为1，也就是撑满行高。对一些字体，设置0.85看起来会更好。

resetSelectionOnContextMenu: boolean
设置在选择文本外点击打开上下文菜单时，是否将光标移动到点击处。默认为true。

workTime, workDelay: number
通过一个假的后台线程高亮 workTime 时长，然后使用 timeout 休息 workDelay 时长。默认为200和300 。（完全不懂这个功能是在说啥）

pollInterval: number
指明CodeMirror向对应的textarea滚动（写数据）的速度（获得焦点时）。大多数的输入都是通过事件捕获，但是有的输入法（如IME）在某些浏览器上并不会生成事件，所以使用数据滚动。默认为100毫秒。

flattenSpans: boolean
默认情况下，CodeMirror会将使用相同class的两个span合并成一个。通过设置此项为false禁用此功能。

addModeClass: boolean
当启用时（默认禁用），会给每个标记添加额外的表示生成标记的mode的以cm-m开头的CSS样式类。例如，XML mode产生的标记，会添加cm-m-xml类。

maxHighlightLength: number
当需要高亮很长的行时，为了保持响应性能，当到达某些位置时，编辑器会直接将其他行设置为纯文本(plain text)。默认为10000，可以设置为Infinity来关闭此功能。

viewportMargin: integer
指定当前滚动到视图中内容上方和下方要渲染的行数。这会影响到滚动时要更新的行数。通常情况下应该使用默认值10。可以设置值为Infinity始终渲染整个文档。注意：这样设置在处理大文档时会影响性能。