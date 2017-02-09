title: jquery_lightbox2 

#  jQuery.lightbox2点击图片灯箱插件 
官网：http://lokeshdhakar.com/projects/lightbox2/
```

<link href="path/to/lightbox.css" rel="stylesheet">
<script src="path/to/lightbox.js"></script>

<div class="image-row">
  <a class="example-image-link" href="images/image-1.jpg" data-lightbox="example-1">
    <img class="example-image" src="images/thumb-1.jpg" alt="Girl looking out people on beach">
  </a>
</div>

```
Add a ` data-lightbox ` attribute to any image link to **enable Lightbox**. For the value of the attribute, ` **use a unique name for each image** `. 
Optional: Add a`  data-title ` attribute if you want to show a caption.
```

<a href="images/image-1.jpg" data-lightbox="image-1" data-title="My caption">Image #1</a>

```

If you have** a group of related images** that you would like to combine into a set, use the same data-lightbox attribute value for all of the images. For example:
```

<a href="images/image-2.jpg" data-lightbox="roadtrip">Image #2</a>
<a href="images/image-3.jpg" data-lightbox="roadtrip">Image #3</a>
<a href="images/image-4.jpg" data-lightbox="roadtrip">Image #4</a>

```

##  选项 
```

    lightbox.option({
      'resizeDuration': 200,
      'wrapAround': true
    })

```
