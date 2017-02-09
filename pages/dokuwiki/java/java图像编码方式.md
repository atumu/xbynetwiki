title: java图像编码方式 

#  java:java图像编码方式 

方式一、
采用JPEGImageEncoder,但是会遇到一点问题，Eclipse会报错，解决：
Eclipse默认把这些受访问限制的API设成了ERROR。只要把Windows-Preferences-Java-Complicer-Errors/Warnings里面的Deprecated and restricted API中的Forbidden references(access rules)选为Warning就可以编译通过。
```

//bufImg为已解码图像数据，故而需要进行编码,jpegOutputStream用于存储编码后的图像流。
JPEGImageEncoder jpegEncoder =JPEGCodec.createJPEGEncoder(jpegOutputStream);
jpegEncoder.encode(bufImg);//编码图像数据并存入jpegOutputStream中。

```

方式二：
采用javax.imageio.ImageIO:
```

ImageIO.write(bufImg,"jpg",jpegOutputStream);

```