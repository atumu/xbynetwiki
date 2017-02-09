title: css伪元素与before_after 

#  css伪元素与before/after 
**:first-line 伪元素**
"first-line" 伪元素用于向文本的首行设置特殊样式。
在下面的例子中，浏览器会根据 "first-line" 伪元素中的样式对 p 元素的第一行文本进行格式化：
```

p:first-line
  {
  color:#ff0000;
  font-variant:small-caps;
  }

```

**伪元素和 CSS 类**
伪元素可以与 CSS 类配合使用：
```

p.article:first-letter
  {
  color: #FF0000;
  }

<p class="article">This is a paragraph in an article。</p>

```
上面的例子会使所有 class 为 article 的段落的首字母变为红色。

**CSS2 - :before 伪元素**
":before" 伪元素可以在元素的内容前面插入新内容。
下面的例子` 在每个 <h1> 元素前面插入一幅图片： `
```

h1:before
  {
  content:url(logo.gif);
  }

```

**CSS2 - :after 伪元素**
":after" 伪元素可以在元素的内容之后插入新内容。
下面的例子在每个 <h1> 元素后面插入一幅图片：
```

h1:after
  {
  content:url(logo.gif);
  }

```

伪元素
W3C："W3C" 列指示出该属性在哪个 CSS 版本中定义（CSS1 还是 CSS2）。
属性	描述	CSS
:first-letter	向文本的第一个字母添加特殊样式。	1
:first-line	向文本的首行添加特殊样式。	1
:before	在元素之前添加内容。	2
:after	在元素之后添加内容。
参考：http://www.w3school.com.cn/css/css_pseudo_elements.asp