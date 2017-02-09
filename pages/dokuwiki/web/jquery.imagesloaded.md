title: jquery.imagesloaded 

#  jQuery图片加载插件 imagesloaded 
官网：https://github.com/desandro/imagesloaded
imagesLoaded插件是在图片加载成功后才做一系列操作
```

<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery.imagesloaded/3.2.0/imagesloaded.pkgd.min.js"></script>

$('#container').imagesLoaded( function() {
  // images have loaded
});

// options
$('#container').imagesLoaded( {
  // options...
  },
  function() {
    // images have loaded
  }
);

```
.imagesLoaded() returns a jQuery Deferred object. This allows you to use .always(), .done(), .fail() and .progress().

```

$('#container').imagesLoaded()
  .always( function( instance ) {
    console.log('all images loaded');
  })
  .done( function( instance ) {
    console.log('all images successfully loaded');
  })
  .fail( function() {
    console.log('all images loaded, at least one is broken');
  })
  .progress( function( instance, image ) {
    var result = image.isLoaded ? 'loaded' : 'broken';
    console.log( 'image is ' + result + ' for ' + image.img.src );
  });

```