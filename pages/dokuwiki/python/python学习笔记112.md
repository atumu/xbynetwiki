title: python学习笔记112 

#  Python学习笔记之PyQt4之多线程3之图像加载 
当我们把图片换成一副较大图片287KB,1058 x 1058的“earth.png”的时候就出现问题了，图片无法显示，程序的界面是一片空白。
据测算，“earth.png”被完全解码后存储在graphics memory中会占用大约4.3MB的空间。如果此时还有其他加载的窗口和QPixmap，很可能就没有空间了。
##  使用QImage加载后转换成QPixmap 显示 
那么安全和正确的方法应该是什么呢？答案是我们需要用QImage做一下预处理：
```

//correct and recommended way
QImage image;
image.load( ":/pics/earth.png" );
QPainter painter(this);
QPixmap pixmapToShow = QPixmap::fromImage( image.scaled(size(), Qt::KeepAspectRatio) );
painter.drawPixmap(0,0, pixmapToShow);

```
` 和QPixmap 不同，QImage是独立于硬件的，它可以同时被另一个线程访问。 `QImage是存储在客户端的，对QImage的使用是非常方便和安全的。 
` 又由于 QImage 也是一种QPaintDevice，因此我们可以在另一个线程中对其进行绘制，而不需要在GUI 线程中处理 `，使用这一方式可以很大幅度提高UI响应速度。 
` 因此当图片较大时，我们可以先通过QImage将图片加载进来，然后把图片缩放成需要的尺寸，最后转换成QPixmap 进行显示。 ` 下图是显示效果（图片是按照earth.png的原始尺寸比例缩放后显示的）：
**其中需要注意的是Qt::KeepAspectRatio的使用，默认参数是Qt::IgnoreAspectRatio，如果我们在程序中这么写：**
```

QPixmap pixmapToShow = QPixmap::fromImage( image.scaled(size(), Qt::IgnoreAspectRatio) );

```
效果就是下面这个样子，earth.png被拉伸以充满整个屏幕：
直接使用QImage 显示
我们也可以直接使用QImage做显示，而不转换成QPixmap ，这要根据我们应用的具体需求来决定，如果需要的话我们可以这么写：
```

//correct, some times may be needed
QImage image;
image.load( ":/pics/earth.png" );

QPainter painter(this);
painter.drawImage(0,0, image);

```
下面是显示效果（当然我们也可以对其进行缩放之后再显示） 从图片可以看出来它是按照原始尺寸显示earth.png的： 

QPixmap是专门为绘图而生，当需要绘制图片时你需要使用QPixmap。QImage则是为I/O，为图片像素访问以及修改而设计的。如果你 想访问图片的像素或是修改图片像素，则需要使用QImage，或者借助于QPainter来操作像素。另外跟QImage不同是，QPixmap跟硬件是 相关的，如X11, Mac 以及 Symbian平台上，QPixmap 是存储在服务器端，而QImage则是存储在客户端，在Windows平台上，QPixmap和QImage都是存储在客户端，并不使用任何的GDI资 源。
参考http://www.cnblogs.com/s_agapo/archive/2012/03/14/2395603.html
http://blog.csdn.net/zzwdkxx/article/details/39480559