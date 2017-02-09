title: 全面了解jquery_事件 

#  全面了解 jQuery 事件 
jQuery 有许多快捷方法，像 **contextmenu()、hover() 和 keyup()**，可以处理不同的事件。除了专门的方法，jQuery 还提供了通用的方法——**on(‘eventName’, handler)**，方便你处理任何事件。牢记一点，这些方法只是在标准 DOM 事件外封装了一层，你可以用原生 JavaScript 来处理这些事件。
##  浏览器事件 
浏览器事件有三种，分别是 **error、resize 和 scroll**。像 images 这样的元素没有正确加载的时候会触发 error 事件。该快键方法在 jQuery 1.8 版本中被舍弃了，所以现在应该用 on(‘error’, handler) 方法来代替。

**浏览器窗口大小发生变化时会触发 resize 事件**。不同浏览器调用 resize 处理方法也不同。Internet Explorer 和 基于 WebKit 的浏览器能够连续的调用处理方法，像 Opera 只能在 resize 事件最后调用。
下面的代码片段是根据窗口宽度改变 image src 属性。
```

$(window).resize(function() {
  var windowWidth = $(window).width();
  if (windowWidth <= 600) {
    $("img").attr("src", "image-src-here.jpg");
    // Image src changed at this point.
  }
});

```

**scroll事件**
在一个指定的元素中，当用户滚动到不同的位置，可以关闭此事件。除了 window（窗口）对象，任何带 scrollbar 的元素都能触发此事件。例如，所有将 overflow 属性设置为 scroll 的元素，或者任何可滚动的 iframe 可以触发此事件。
请牢记，scroll 位置发生改变时，将调用处理方法。滚动的原因无关紧要。可能是按了方向键，单击或者拖拽滚动条或者使用了鼠标滚轮。在下面的代码，检查用户向下滚动是否超过 500px，然后触发一些动作。
```

$(window).scroll(function() {
  if ($(window).scrollTop() >= 500) {
    $("#alert").text("You have scrolled enough!");
    // Update the text inside alert box.
  }
});

```
` 这里需要理解$(window).scrollTop() , $(window).height() , $(document).height()是不同的。 `

##  加载事件 
基于 document 或 DOM 状态的三种事件，jQuery 有对应的方法，分别是 load、unload 和 ready。
load() 和 unload() 在 1.8 版本中都被废弃了。
```

$(document).ready(function() {
  $('h2').text('Document Ready!');
});

```
##  Keyboard Events 
keyboard（键盘） 事件是由用户和键盘之间的交互触发的。每个 keyboard 事件包含按键和事件类型信息。
jQuery 中有三个 keyboard 快捷方法——keydown()、keyup() 和 keypress()。
keyup 就是当用户释放键盘上的一个键触发，keydown 就是按住键盘上一个键触发。这些事件的处理方法可以绑定到任何元素上，但是只有当前焦点元素才会被触发。
推荐使用 event 对象中的 **which 属性**来确定哪个键被按下了。
```

$("#alert").keydown(function(event) {
  switch (event.which) {
    case 89: // keycode for y
    $("#element").remove(); // Remove element from the DOM
    break;
  }
});

```
keypress 事件和 keydown 事件非常相似。一个主要的区别就是修饰符和一些像 Shift 、Esc 等无法打印的键，不会触发 keypress 事件。不应该用 keypress 事件捕捉像箭头这类的特殊键。当你想知道输入的是哪个字符，比如 A 或者 a ，一般可以使用 keypress 来处理。
```

$("body").keypress(function(event) {
  switch (event.keyCode) {
    case 75: 
    // 75 stands for capital K in keypress event
    $(".K").css("display", "none");
    break;
  }
});

```

##  Mouse Events 
当用户使用像鼠标这种指点设备时，mouse 事件会被触发。该事件基于点击，如单击、双击和右键快捷菜单或者移动动作，如 mouseenter、mouseleave 和 mousemove。在本节中，将简要地讨论所有这些动作，包括一些演示，说明它们之间的细微差别。

Click-Based Events
jQuery 中定义了五种基于点击事件的方法。mousedown 和 mouseup 事件，从名字上显而易见其意思，当用户在一个元素上分别按住、释放鼠标按键就会触发。另一方面，只有当鼠标按键在指定元素上按住然后释放才会触发点击事件。

dblclick 稍微复杂一点。对于注册为 dblclick 的事件，应该在系统限制时间前快速的点击两次。不能给一个单独的元素同时绑定单击和双击的处理方法，对于双击事件的触发浏览器的处理比较特殊。一些浏览器在双击之前可能会注册两个单独的单击事件，而其他浏览器在双击之前可能只注册一个单击事件。


###  contextmenu 事件 
在一个元素上右键单击，在显示内容菜单前触发 contextmenu 事件。这意味着处理方法中可以用代码阻止显示默认菜单。
下面的代码就阻止了右键单击默认的菜单显示，而是显示了一个自定义菜单。
```

$("img").contextmenu(function(event) {
  event.preventDefault();
  $("#custom-menu")
    .show().css({
      top: event.pageY + 10,
      left: event.pageX + 10
      // Show the menu near the mouse click
    });
});

$("#custom-menu #option").click(function() {
   $("img").toggleClass("class-name");
   // Toggle an image class.
});

```
##  Movement-Based Events 
一些事件是基于鼠标指针移动的，进入或离开元素。有六种基于鼠标移动的事件。
让我们从 mouseover 和 mouseenter 事件开始。正如名字的字面意思，当鼠标指针进入一个元素的时候会触发。类似的，鼠标指针离开一个元素 mouseleave 和 mouseout 事件触发。
mouseleave 和 mouseout 之间的一个区别就是前者鼠标指针移出绑定的元素就会触发。后者移出元素的任意后代元素也会触发。mouseenter 和 mouseover 的不同也是一样的（同理与 mouseleave 和 mouseout）。
当鼠标指针移入一个元素内 mousemove 事件触发。每当有鼠标移动就会触发，即使移动距离小到只有一个像素。因此，段时间内就可能触发上百次。可以想象，在处理方法中执行复杂的操作会造成浏览器卡顿。明智的做法就是让 mousemove 事件处理方法尽可能高效，当不需要的时候就解除绑定。

**当鼠标指针进入并离开元素才会触发 hover。**有两种方法调用 hover 方法。第一种是：当鼠标指针移入元素时执行 handlerIn()，鼠标指针移出时执行 handlerOut()。
```

$("your-selector").hover( handlerIn, handlerOut );

```
第二种方法是：这次鼠标指针移入移出元素时都执行 handlerInOut 方法。
```

$("your-selector").hover(handlerInOut);

```
##  Form Events 
表单在网络中无所不在。几乎每个用户都填写过一些表单。jQuery 有指定方法专门处理表单事件。 表单值改变或者丢失焦掉都可能触发这些表单事件。有七种表单事件，一个一个来讨论它们。
The Blur, Focus, Focusin and Focusout Events
The Select, Change and Submit Events

参考:http://web.jobbole.com/86368/