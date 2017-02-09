title: thumbnailator生成缩略图的java类库 

#  Thumbnailator生成缩略图的Java类库 
Thumbnailator的下载地址：
http://code.google.com/p/thumbnailator/downloads/list
Java Doc
http://thumbnailator.googlecode.com/hg/javadoc/index.html
**Thumbnailator**是一个用来生成图像缩略图的 Java类库，通过很简单的代码即可生成图片缩略图，也可直接对一整个目录的图片生成缩略图。
有了这玩意，就不用在费心思使用Image I/O API,Java 2D API等等来生成缩略图了。
**支持：图片缩放，区域裁剪，水印，旋转，保持比例。**
![](/data/dokuwiki/javaweb/pasted/20151020-100317.png)
的确是爽歪歪的说，一行代码就把大鸟变小鸟。
那我要是有一个文件夹都需要生成缩略图，那还是很麻烦，有没有对文件夹下所有图片生成缩略图呢？答案是肯定的：
```

Thumbnails.of(new File("path/to/directory")
.listFiles())         
.size(640, 480)         
.outputFormat("jpg")         
.toFiles(Rename.PREFIX_DOT_THUMBNAIL);

```
**2．特点**
2.1.可以根据现有的图片生成高质量的缩略图
![](/data/dokuwiki/javaweb/pasted/20151020-100414.png)
2.2.可以在缩略图中嵌入水印，并且可以设置水印的透明度：
![](/data/dokuwiki/javaweb/pasted/20151020-100438.png)
2.3.支持生成经过旋转后的缩略图：
```

for (int i : new int[] {0, 90, 180, 270, 45}) {
    Thumbnails.of(new File("coobird.png"))
            .size(100, 100)
            .rotate(i)
            .toFile(new File("image-rotated-" + i + ".png"));
}

```
2.4.可以生成多种质量模式的缩略图
2.5.如果需要的话，在生成缩略图的时候可以保持和源图像一样的的宽高比

**3.更多实战例子**
3、1、指定大小进行缩放 
```

//size(宽度, 高度)  
  
/*   
 * 若图片横比200小，高比300小，不变   
 * 若图片横比200小，高比300大，高缩小到300，图片比例不变   
 * 若图片横比200大，高比300小，横缩小到200，图片比例不变   
 * 若图片横比200大，高比300大，图片按比例缩小，横为200或高为300   
 */   
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(200, 300)  
        .toFile("c:/a380_200x300.jpg");  
  
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(2560, 2048)   
        .toFile("c:/a380_2560x2048.jpg"); 

```
```

Thumbnails.of(new File("original.jpg"))
        .size(160, 160)
        .toFile(new File("thumbnail.jpg"));

```
```

Thumbnails.of("original.jpg")
        .size(160, 160)
        .toFile("thumbnail.jpg");

```
3.2.生成一个带有旋转和水印的缩略图：
```

Thumbnails.of(new File("original.jpg"))
        .size(160, 160)
        .rotate(90)
        .watermark(Positions.BOTTOM_RIGHT, ImageIO.read(new File("watermark.png")), 0.5f)
        .outputQuality(0.8f)
        .toFile(new File("image-with-watermark.jpg"));

```
这段代码是从original.jpg这张图片生成最大尺寸160*160，顺时针旋转90°，水印放在右下角，50%的透明度，80%的质量压缩的缩略图。

3.3.把生成的图片输出到输出流（OutPutStream）中
```

OutputStream os = ...;
                 
Thumbnails.of("large-picture.jpg")
        .size(200, 200)
        .outputFormat("png")
        .toOutputStream(os);

```
3、3输出到BufferedImage
```

//asBufferedImage() 返回BufferedImage  
BufferedImage thumbnail = Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(1280, 1024)  
        .asBufferedImage();  
ImageIO.write(thumbnail, "jpg", new File("c:/a380_1280x1024_BufferedImage.jpg"));   

```
**3.4.按一定的比例生成缩略图**
```

BufferedImage originalImage = ImageIO.read(new File("original.png"));
 
BufferedImage thumbnail = Thumbnails.of(originalImage)
        .scale(0.25f)
        .asBufferedImage();

```
生成缩略图的大小是原来的25%

3.5不按照比例，指定大小进行缩放 
```

//默认是按照比例缩放的,设置keepAspectRatio(false) 取消默认行为
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(200, 200)   
        .keepAspectRatio(false)   
        .toFile("c:/a380_200x200.jpg");

```  
3、6旋转:
```

//rotate(角度),正数：顺时针 负数：逆时针  
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(1280, 1024)  
        .rotate(90)   
        .toFile("c:/a380_rotate+90.jpg");   
  
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(1280, 1024)  
        .rotate(-90)   
        .toFile("c:/a380_rotate-90.jpg");  

```
3、7裁剪：
```

//sourceRegion()  
  
//图片中心400*400的区域  
Thumbnails.of("images/a380_1280x1024.jpg")  
        .sourceRegion(Positions.CENTER, 400,400)  
        .size(200, 200)  
        .keepAspectRatio(false)   
        .toFile("c:/a380_region_center.jpg");  
  
//图片右下400*400的区域  
Thumbnails.of("images/a380_1280x1024.jpg")  
        .sourceRegion(Positions.BOTTOM_RIGHT, 400,400)  
        .size(200, 200)  
        .keepAspectRatio(false)   
        .toFile("c:/a380_region_bootom_right.jpg");  
  
//指定坐标  
Thumbnails.of("images/a380_1280x1024.jpg")  
        .sourceRegion(600, 500, 400, 400)  
        .size(200, 200)  
        .keepAspectRatio(false)   
        .toFile("c:/a380_region_coord.jpg");  

```
3、8转换图像格式：
```

//outputFormat(图像格式)  
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(1280, 1024)  
        .outputFormat("png")   
        .toFile("c:/a380_1280x1024.png");   
  
Thumbnails.of("images/a380_1280x1024.jpg")   
        .size(1280, 1024)  
        .outputFormat("gif")   
        .toFile("c:/a380_1280x1024.gif");  

```
参考：http://www.oschina.net/question/76860_25758
http://rensanning.iteye.com/blog/1545708
http://blog.csdn.net/chenleixing/article/details/44685817