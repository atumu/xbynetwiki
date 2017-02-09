title: phplearn4 

#  PHP学习之include文件 
服务器端包含 (SSI) 用于创建可在多个页面重复使用的函数、页眉、页脚或元素。
include （或 require）语句会获取指定文件中存在的所有文本/代码/标记，并复制到使用 include 语句的文件中。
包含文件很有用，如果您需要在网站的多张页面上引用相同的 PHP、HTML 或文本的话。

**PHP include 和 require 语句**
通过 include 或 require 语句，可以将 PHP 文件的内容插入另一个 PHP 文件**（在服务器执行它之前）**。
**区别：**
  * include 和 require 语句是相同的，除了**错误处理方面**：require 会生成致命错误（E_COMPILE_ERROR）并停止脚本、include 只生成警告（E_WARNING），并且脚本会继续，大多情况下推荐使用 require() 函数，以避免在错误引用发生后的程序继续执行
  * **不管 require() 语句是否执行，程序执行包含文件都被加入进来，include() 只有执行的时候文件才会被包含。**所以**如果是有条件判断的情况下，用 include() 显然更合适**
  * **使用 require() 多次引用时，只执行一次对被引用文件的引用动作，而 include() 则每次都要进行读取和评估后引用文件**
如果想设定系统包含路径[PHP 设定系统 include_path 包含路径](http://www.5idev.com/p-php_ini_set_include_path.shtml)
语法
include 'filename';
或
require 'filename';

**include和require语句包含并运行指定文件的顺序:**
1、被包含文件先按参数给出的路径寻找，
2、如果没有给出目录（只有文件名）时则按照 ` include_path 指定的目录寻找 `。
3、如果在 include_path 下没找到该文件则 include 最后才在调用脚本文件所在的目录和当前工作目录下寻找。
4、如果最后include仍未找到文件则 include 结构会发出一条警告；这一点和 require 不同，后者会发出一个致命错误。
**如果定义了路径——不管是绝对路径（在 Windows 下以盘符或者 \ 开头，在 Unix/Linux 下以 / 开头）还是当前目录的相对路径（以 . 或者 .. 开头）并且该路径下文件存在。——include_path 都会被完全忽略。**例如一个文件以 ../ 开头，则解析器会在当前目录的父目录下寻找该文件。
当一个文件被包含时，其中所包含的代码继承了 include 所在行的变量范围。从该处开始，调用文件在该行处可用的任何变量在被调用的文件中也都可用。不过所有在包含文件中定义的函数和类都具有全局作用域。

  * get_include_path — 获取当前的 include_path 配置选项
  * restore_include_path() - 还原 include_path 配置选项的值
  * set_include_path() - 设置 include_path 配置选项


PHP include 实例
假设我们有一个名为 "footer.php" 的标准的页脚文件，就像这样：
```

<?php
echo "<p>Copyright © 2006-" . date("Y") . " W3School.com.cn</p>";
?>

```
如需在一张页面中引用这个页脚文件，请使用 include 语句：
```

<html>
<body>
<h1>欢迎访问我们的首页！</h1>
<p>一段文本。</p>
<p>一段文本。</p>
<?php include 'footer.php';?>
</body>
</html>

```
结合之后相当于:
```

<html>
<body>
<h1>欢迎访问我们的首页！</h1>
<p>一段文本。</p>
<p>一段文本。</p>
<?php
echo "<p>Copyright © 2006-" . date("Y") . " W3School.com.cn</p>";
?>
</body>
</html>

```
PHP include vs. require
require 语句同样用于向 PHP 代码中引用文件。
不过，include 与 require 有一个巨大的差异：如果用 include 语句引用某个文件并且 PHP 无法找到它，脚本会继续执行：
```

<html>
<body>
<h1>Welcome to my home page!</h1>
<?php
include 'noFileExists.php';
echo "I have a $color $car.";
?>
</body>
</html>

```
如果我们使用 require 语句完成相同的案例，echo 语句不会继续执行，因为在 require 语句返回严重错误之后脚本就会终止执行：
```

<html>
<body>

<h1>Welcome to my home page!</h1>
<?php
require 'noFileExists.php';
echo "I have a $color $car.";
?>

</body>
</html>

```
注释：
  * 请在此时使用 require：当文件被应用程序请求时。
  * 请在此时使用 include：当文件不是必需的，且应用程序在文件未找到时应该继续运行时。