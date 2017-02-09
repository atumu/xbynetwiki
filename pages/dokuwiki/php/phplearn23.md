title: phplearn23 

#  PHP学习之图像处理 
**php.ini配置开启php_gd2.dll**
还有三方库如**ImageMagick**更为强大。

##  PHP 图像处理函数 
PHP 提供了丰富的图像处理函数，主要包括：
获取图像信息
  * getimagesize()：获取图像尺寸，类型等信息。
  * imagesx()：获取图像宽度。
  * imagesy()：获取图像高度。
创建图像
  * imagecreate()：创建一幅空白图像。
  * imagecreatetruecolor()：创建一幅真彩色空白图像。
销毁图像资源
  * imagedestroy()：销毁图像资源。
载入图像
  * imagecreatefromgif()：创建一块画布，并从 GIF 文件或 URL 地址载入一副图像
  * imagecreatefromjpeg()：创建一块画布，并从 JPEG 文件或 URL 地址载入一副图像
  * imagecreatefrompng()：创建一块画布，并从 PNG 文件或 URL 地址载入一副图像
  * imagecreatefromwbmp()：创建一块画布，并从 WBMP 文件或 URL 地址载入一副图像
  * imagecreatefromstring()：创建一块画布，并从字符串中的图像流新建一副图像
输出图像
  * imagegif()：以 GIF 格式将图像输出到浏览器或文件
  * imagejpeg()：以 JPEG 格式将图像输出到浏览器或文件
  * imagepng()：以 PNG 格式将图像输出到浏览器或文件
  * imagewbmp()：以 WBMP 格式将图像输出到浏览器或文件
分配/取消图像颜色
  * imagecolorallocate()：为图像分配颜色。
  * imagecolordeallocate()：取消先前由 imagecolorallocate() 等函数为图像分配的颜色。
拷贝图像
  * imagecopy()：拷贝图像。
  * imagecopyresized()：拷贝图像并调整大小。
合并图像（水印制作实例）
imagecopymerge()：拷贝并合并图像的一部分。
绘制线段与圆弧
imageline()：绘制一条线段。
imagesetstyle()：设定画线风格。
imagearc()：绘制椭圆弧（包括圆弧）。
图像填充
imagefill()：填充图像区域。
imagefilledarc()：画一椭圆弧并填充。
imagefilledrectangle()：画一矩形并填充。
imagefilledpolygon()：画一多边形并填充。
**GD 库**
使用 PHP 图像处理函数，需要加载 GD 支持库。请确定 php.ini 加载了 GD 库：
extension = php_gd2.dll
##  getimagesize() 
` getimagesize() 函数 `用于获取图像大小及相关信息，成功返回一个数组，失败则返回 FALSE 并产生一条 E_WARNING 级的错误信息。
语法：array getimagesize( string filename )
例子：
```

<?php
$array = getimagesize("images/flower_1.jpg");
print_r($array);
?>
浏览器显示如下：
Array
(
    [0] => 350
    [1] => 318
    [2] => 2
    [3] => width="350" height="318"
    [bits] => 8
    [channels] => 3
    [mime] => image/jpeg
)

```
返回结果说明
  * 索引 0 给出的是图像**宽度的像素值**
  * 索引 1 给出的是图像**高度的像素值**
  * 索引 2 给出的是**图像的类型**，返回的是数字，其中1 = GIF，2 = JPG，3 = PNG，4 = SWF，5 = PSD，6 = BMP，7 = TIFF(intel byte order)，8 = TIFF(motorola byte order)，9 = JPC，10 = JP2，11 = JPX，12 = JB2，13 = SWC，14 = IFF，15 = WBMP，16 = XBM
  * 索引 3 给出的是**一个宽度和高度的字符串**，可以直接用于 HTML 的 <image> 标签
  * 索引 bits 给出的是图像的每种颜色的位数，二进制格式
  * 索引 channels 给出的是图像的通道值，RGB 图像默认是 3
  * 索引 mime 给出的是**图像的 MIME 信息**，此信息可以用来在 HTTP Content-type 头信息中发送正确的信息，如：header("Content-type: image/jpeg");

更多参考系列:http://www.5idev.com/p-php_imagecopymerge.shtml

##  PHP图片验证码 
###  示例1 
http://www.oschina.net/code/snippet_258733_12375
```

<?php
/**
 * vCode(m,n,x,y) m个数字  显示大小为n   边宽x   边高y
 */
session_start(); 
vCode(4, 15); //4个数字，显示大小为15
 
function vCode($num = 4, $size = 20, $width = 0, $height = 0) {
    !$width && $width = $num * $size * 4 / 5 + 5;
    !$height && $height = $size + 10; 
    // 去掉了 0 1 O l 等
    $str = "23456789abcdefghijkmnpqrstuvwxyzABCDEFGHIJKLMNPQRSTUVW";
    $code = '';
    for ($i = 0; $i < $num; $i++) {
        $code .= $str[mt_rand(0, strlen($str)-1)];
    } 
    // 画图像
    $im = imagecreatetruecolor($width, $height); 
    // 定义要用到的颜色
    $back_color = imagecolorallocate($im, 235, 236, 237);
    $boer_color = imagecolorallocate($im, 118, 151, 199);
    $text_color = imagecolorallocate($im, mt_rand(0, 200), mt_rand(0, 120), mt_rand(0, 120)); 
    // 画背景
    imagefilledrectangle($im, 0, 0, $width, $height, $back_color); 
    // 画边框
    imagerectangle($im, 0, 0, $width-1, $height-1, $boer_color); 
    // 画干扰线
    for($i = 0;$i < 5;$i++) {
        $font_color = imagecolorallocate($im, mt_rand(0, 255), mt_rand(0, 255), mt_rand(0, 255));
        imagearc($im, mt_rand(- $width, $width), mt_rand(- $height, $height), mt_rand(30, $width * 2), mt_rand(20, $height * 2), mt_rand(0, 360), mt_rand(0, 360), $font_color);
    } 
    // 画干扰点
    for($i = 0;$i < 50;$i++) {
        $font_color = imagecolorallocate($im, mt_rand(0, 255), mt_rand(0, 255), mt_rand(0, 255));
        imagesetpixel($im, mt_rand(0, $width), mt_rand(0, $height), $font_color);
    } 
    // 画验证码
    @imagefttext($im, $size , 0, 5, $size + 3, $text_color, 'c:\\WINDOWS\\Fonts\\simsun.ttc', $code);
    $_SESSION["VerifyCode"]=$code; 
    header("Cache-Control: max-age=1, s-maxage=1, no-cache, must-revalidate");
    header("Content-type: image/png;charset=gb2312");
    imagepng($im);
    imagedestroy($im);
} 
 

```