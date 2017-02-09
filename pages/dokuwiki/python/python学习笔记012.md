title: python学习笔记012 

#  Python学习笔记之图片处理PIL 
PIL文档:https://pillow.readthedocs.org/
PIL：Python Imaging Library，已经是Python平台事实上的图像处理标准库了。PIL功能非常强大，但API却非常简单易用。
由于PIL仅支持到Python 2.7，加上年久失修，于是一群志愿者在PIL的基础上创建了兼容的版本，**名字叫Pillow，支持最新Python 3.x，又加入了许多新特性，因此，我们可以直接安装使用Pillow。**
##  快速入门 
安装Pillow
在命令行下直接通过pip安装：
```

$ pip install pillow

```
如果遇到Permission denied安装失败，请加上sudo重试。
操作图像
来看看最常见的图像缩放操作，只需三四行代码：
```

from PIL import Image
# 打开一个jpg图像文件，注意是当前路径:
im = Image.open('test.jpg')
# 获得图像尺寸:
w, h = im.size
print('Original image size: %sx%s' % (w, h))
# 缩放到50%:
im.thumbnail((w//2, h//2))
print('Resize image to: %sx%s' % (w//2, h//2))
# 把缩放后的图像用jpeg格式保存:
im.save('thumbnail.jpg', 'jpeg')

```
其他功能如切片、旋转、滤镜、输出文字、调色板等一应俱全。
比如，模糊效果也只需几行代码：
```

from PIL import Image, ImageFilter
# 打开一个jpg图像文件，注意是当前路径:
im = Image.open('test.jpg')
# 应用模糊滤镜:
im2 = im.filter(ImageFilter.BLUR)
im2.save('blur.jpg', 'jpeg')

```
##  使用 
主要功能
  * 图片压缩：创建缩略图、转换格式、打印图片等
  * 图片显示：包括Tk图像接口` PhotoImage和BitmapImage `.
  * 图片处理:图像缩放、切片、旋转、滤镜、输出文字、调色板等
```

#从一个文件打开一个图片
>>> from PIL import Image
>>> im = Image.open("lena.png") #返回Image对象，如果打开失败抛出IOError
#Image对象有很多属性。比如：
>>> print(im.format, im.size, im.mode)
PNG (512, 512) RGB
#显示图片，调用系统自带看图软件打开
>>> im.show()


```
###  读写图片 
Convert files to JPEG
```

import os, sys
from PIL import Image
for infile in sys.argv[1:]:
    f, e = os.path.splitext(infile)
    outfile = f + ".jpg"
    if infile != outfile:
        try:
            Image.open(infile).save(outfile)  #保存图片使用save
        except IOError:
            print("cannot convert", infile)

```
Create JPEG thumbnails
```

import os, sys
from PIL import Image
size = (128, 128)
for infile in sys.argv[1:]:
    outfile = os.path.splitext(infile)[0] + ".thumbnail"
    if infile != outfile:
        try:
            im = Image.open(infile)
            im.thumbnail(size)
            im.save(outfile, "JPEG")
        except IOError:
            print("cannot create thumbnail for", infile)

```
Cutting, pasting, and merging images
```

#裁剪一个图片区域
box = (100, 100, 400, 400)
region = im.crop(box)
#旋转一个角度后补上
region = region.transpose(Image.ROTATE_180)
im.paste(region, box)
#简单变换transforms
out = im.resize((128, 128))
out = im.rotate(45) # degrees counter-clockwise

```

Transposing an image
```

out = im.transpose(Image.FLIP_LEFT_RIGHT)
out = im.transpose(Image.FLIP_TOP_BOTTOM)
out = im.transpose(Image.ROTATE_90)
out = im.transpose(Image.ROTATE_180)
out = im.transpose(Image.ROTATE_270)

```

 Draw a gray cross over an image
```

from PIL import Image, ImageDraw
im = Image.open("lena.pgm")
draw = ImageDraw.Draw(im)
draw.line((0, 0) + im.size, fill=128)
draw.line((0, im.size[1], im.size[0], 0), fill=128)
del draw

# write to stdout
im.save(sys.stdout, "PNG")

```
###  图片加水印 
```

from PIL import Image, ImageDraw, ImageFont
# get an image
base = Image.open('Pillow/Tests/images/lena.png').convert('RGBA')
# make a blank image for the text, initialized to transparent text color
txt = Image.new('RGBA', base.size, (255,255,255,0))

# get a font
fnt = ImageFont.truetype('Pillow/Tests/fonts/FreeMono.ttf', 40)
# get a drawing context
d = ImageDraw.Draw(txt)

# draw text, half opacity
d.text((10,10), "Hello", font=fnt, fill=(255,255,255,128))
# draw text, full opacity
d.text((10,60), "World", font=fnt, fill=(255,255,255,255))
out = Image.alpha_composite(base, txt)
out.show()

```
###  滤镜效果 
out=out.filter(ImageFilter.BLUR)
ImageFilter种类:
  * BLUR
  * CONTOUR
  * DETAIL
  * EDGE_ENHANCE
  * EDGE_ENHANCE_MORE
  * EMBOSS
  * FIND_EDGES
  * SMOOTH
  * SMOOTH_MORE
  * SHARPEN
##  生成字母验证码图片 
PIL的ImageDraw提供了一系列绘图方法，让我们可以直接绘图。**比如要生成字母验证码图片**：
```

#!/usr/bin/env python
#coding=utf-8

import random
from PIL import Image, ImageDraw, ImageFont, ImageFilter

_letter_cases = "abcdefghjkmnpqrstuvwxy" # 小写字母，去除可能干扰的i，l，o，z
_upper_cases = _letter_cases.upper() # 大写字母
_numbers = ''.join(map(str, range(3, 10))) # 数字
init_chars = ''.join((_letter_cases, _upper_cases, _numbers))

def create_validate_code(size=(120, 30),
                         chars=init_chars,
                         img_type="GIF",
                         mode="RGB",
                         bg_color=(255, 255, 255),
                         fg_color=(0, 0, 255),
                         font_size=18,
                         font_type=r"C:\Windows\Fonts\ALGER.TTF",
                         length=4,
                         draw_lines=True,
                         n_line=(1, 2),
                         draw_points=True,
                         point_chance = 2):
    '''
    @todo: 生成验证码图片
    @param size: 图片的大小，格式（宽，高），默认为(120, 30)
    @param chars: 允许的字符集合，格式字符串
    @param img_type: 图片保存的格式，默认为GIF，可选的为GIF，JPEG，TIFF，PNG
    @param mode: 图片模式，默认为RGB
    @param bg_color: 背景颜色，默认为白色
    @param fg_color: 前景色，验证码字符颜色，默认为蓝色#0000FF
    @param font_size: 验证码字体大小
    @param font_type: 验证码字体，默认为 ae_AlArabiya.ttf
    @param length: 验证码字符个数
    @param draw_lines: 是否划干扰线
    @param n_lines: 干扰线的条数范围，格式元组，默认为(1, 2)，只有draw_lines为True时有效
    @param draw_points: 是否画干扰点
    @param point_chance: 干扰点出现的概率，大小范围[0, 100]
    @return: [0]: PIL Image实例
    @return: [1]: 验证码图片中的字符串
    '''

    width, height = size # 宽， 高
    img = Image.new(mode, size, bg_color) # 创建图形
    draw = ImageDraw.Draw(img) # 创建画笔

    def get_chars():
        ` '生成给定长度的字符串，返回列表格式 `'
        return random.sample(chars, length)

    def create_lines():
        ` '绘制干扰线 `'
        line_num = random.randint(*n_line) # 干扰线条数

        for i in range(line_num):
            # 起始点
            begin = (random.randint(0, size[0]), random.randint(0, size[1]))
            #结束点
            end = (random.randint(0, size[0]), random.randint(0, size[1]))
            draw.line([begin, end], fill=(0, 0, 0))

    def create_points():
        ` '绘制干扰点 `'
        chance = min(100, max(0, int(point_chance))) # 大小限制在[0, 100]
       
        for w in range(width):
            for h in range(height):
                tmp = random.randint(0, 100)
                if tmp > 100 - chance:
                    draw.point((w, h), fill=(0, 0, 0))

    def create_strs():
        ` '绘制验证码字符 `'
        c_chars = get_chars()
        strs = ' %s ' % ' '.join(c_chars) # 每个字符前后以空格隔开
       
        font = ImageFont.truetype(font_type, font_size)
        font_width, font_height = font.getsize(strs)

        draw.text(((width - font_width) // 3, (height - font_height) // 3),
                    strs, font=font, fill=fg_color)
       
        return ''.join(c_chars)

    if draw_lines:
        create_lines()
    if draw_points:
        create_points()
    strs = create_strs()

    # 图形扭曲参数
    params = [1 - float(random.randint(1, 2)) // 100,
              0,
              0,
              0,
              1 - float(random.randint(1, 10)) // 100,
              float(random.randint(1, 2)) // 500,
              0.001,
              float(random.randint(1, 2)) // 500
              ]
    img = img.transform(size, Image.PERSPECTIVE, params) # 创建扭曲

    img = img.filter(ImageFilter.EDGE_ENHANCE_MORE) # 滤镜，边界加强（阈值更大）

    return img, strs

if __name__ == "__main__":
    code_img = create_validate_code()
    code_img[0].show()
    code_img.save("validate.gif", "GIF")


```
时候，细心的同学可能要问，如果每次生成验证码，都要先保存生成的图片，再显示到页面。这么做让人太不能接受了。
这个时候，我们需要使用python内置的**StringIO模块，它有着类似file对象的行为，但是它操作的是内存文件。**于是，我们可以这么写代码：
```

try:
    import cStringIO as StringIO
except ImportError:
    import StringIO
mstream = StringIO.StringIO()
img = create_validate_code()[0]
img.save(mstream, "GIF")

``` 
我们用随机颜色填充背景，再画上文字，最后对图像进行模糊，得到验证码图片如下：
![](/data/dokuwiki/python/pasted/20160314-172931.png)
PIL提供了操作图像的强大功能，可以通过简单的代码完成复杂的图像处理。
要详细了解PIL的强大功能，请请参考Pillow官方文档：
https://pillow.readthedocs.org/

##  ImageGrab截图 (OS X and Windows only) 
```

import PIL
from PIL import Image,ImageGrab
im = ImageGrab.grab()
im.save(r"d:\sketch.png")

```