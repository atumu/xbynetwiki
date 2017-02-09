title: js复制文本到剪切板clipboard.js 

#  js复制文本到剪切板clipboard.js 
https://clipboardjs.com/
```

<script src="dist/clipboard.min.js"></script>
new Clipboard('.btn');

```
方式一：复制属性文本
![](/data/dokuwiki/web/pasted/20160510-143129.png)
```

<!-- Trigger -->
<button class="btn" data-clipboard-text="Just because you can doesn't mean you should — clipboard.js">
    Copy to clipboard
</button>

```
方式二、复制其他元素值
![](/data/dokuwiki/web/pasted/20160510-143112.png)
```

<!-- Target -->
<input id="foo" value="https://github.com/zenorocha/clipboard.js.git">

<!-- Trigger -->
<button class="btn" data-clipboard-target="#foo">
    <img src="assets/clippy.svg" alt="Copy to clipboard">
</button>

```