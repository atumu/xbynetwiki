title: jquery_masonry 

#  jQuery Masonry瀑布流插件 
官网：http://masonry.desandro.com/
```

<script src="https://cdnjs.cloudflare.com/ajax/libs/masonry/3.3.2/masonry.pkgd.min.js"></script>
<div class="grid">
  <div class="grid-item">...</div>
  <div class="grid-item grid-item--width2">...</div>
  <div class="grid-item">...</div>
  ...
</div>

```
```

.grid-item { width: 200px; }
.grid-item--width2 { width: 400px; }

```
```

$('.grid').masonry({
  // options
  itemSelector: '.grid-item',
  columnWidth: 200
});

```
参考：http://www.utubon.com/post/2755.html
http://truemylife.iteye.com/blog/1671053