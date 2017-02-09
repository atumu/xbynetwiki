title: phplearn29 

#  PHP学习之API文档生成工具 
phpDocumentor:API描述自动生成工具
https://phpdoc.org/
https://github.com/phpDocumentor/phpDocumentor2
文档:https://phpdoc.org/docs/latest/index.html
**phpDocumentor的安装**
参考https://github.com/phpDocumentor/phpDocumentor2
  * 如果通过pear自动安装,在命令行下输入
pear channel-discover pear.phpdoc.org
pear install phpdoc/phpDocumentor
安装之后就可以在命令行使用phpdoc命令了。

  * 如果是手动安装:则下载最新版本的[PhpDocumentor.pear](http://phpdoc.org/phpDocumentor.phar)即可.

  * 如果是Composer
composer require --dev phpdocumentor/phpdocumentor
使用时这样既可：$ php vendor/bin/phpdoc

##  命令行使用说明 
```

phpdoc  -d <SOURCE_DIRECTORY> -t <TARGET_DIRECTORY> --template="clean"
$ phpdoc -d "./src" -t "./docs/api" --template="clean"

```
模板使用:
可使用模板:参考https://phpdoc.org/templates
![](/data/dokuwiki/php/pasted/20160409-020106.png)

-- 最后来说一下怎么写注解 --
phpDocumentor 的注解有一定的规格，类似于javadoc注释
简单的来看个范例好了
```

<?php
/**
* 这里是这个类的说明
*/

class MyClass {

   /**
   * 这里是变量的说明.
   * @var string 这里也可以放说明
   */

   var $b ;


   /**
   * 这是针对函式的说明
   * 也是一样可以多行
   * 若是简单的范例也可以放这里
   * @param int $a 可以放入传入的型态
   * @return array 可以说明回传的型态
   */
   function first ( $a ) {
      return array();
   }
}
?>

```

注解比较常用到参数的应该是
@author 程序作者名称，联络方式
@const 常数
@deprecate 不建议使用的 API
@global 全域变量
@param 函数的参数
@return 回传值
@see 可参考函数
@since 开始时间
@static 静态变量
@var 物件成员变量
@todo 计划中要进行的项目
参考https://phpdoc.org/docs/latest/guides/docblocks.html

##  安装与使用过程中问题与解决 
http://www.xuebuyuan.com/280759.html?mobile=1
**一定要先安装GraphViz，再安装PhpDoc。**
PHPDoc跟XDoc（JavaDoc，NDoc。。。）一样，根据注释生成HTML格式的程序帮助文档。
GraphViz用于绘制DOT语言脚本描述的图形。安装它之后PhpDoc可以输出类图。
发生如下错误：
 The XSL writer was unable to find your XSLTProcessor; please check if you have installed the PHP XSL extension
解决办法如下：
获取php_xsl (这个文件在我的php5.3.22上测试通过)，在PHP.ini中添加
**extension=php_xsl.dll**
4.再次运行
phpdoc -d . -t C:\docs
OK，在C：\docs目录下生成了HTML格式的说明。
5.但是，仔细查看执行phpdoc的执行结果发现命令行终端上出现如下错误：
Unable to find the `dot` command of the GraphViz package. Is GraphViz correctl
y installed and present in your path?
打开生成的phpDoc,点击Charts菜单下的子菜单，发现PhpDoc还会生成类图，但是因为我没有安装GraphViz,没有办法实现。。。决定搞定它。
GraphViz官网:http://www.graphviz.org/
下载安装包，顺利装上不表。怀着激动的心情再次执行phpdoc命令，OMG！还是原来的错误提示，迫不急待打开命令行敲下“dot”命令。。。输出了dot的帮助信息，说明已经装上了。但是，肿么用不鸟！
执行
pear uninstall phpdoc/phpDocumentor
卸载PhpDoc!再重装!OMG！PHPDoc也装不上了(当时撞墙的心都有了)！提示错误信息：
No releases available for package ...
冷静下来仔细分析pear的命令参数，发现有一个参数是clear-cache，觉得可能是pear缓存了安装信息，所以认为本机已经安装过phpdoc。执行了一下clear-cache，然后重新安装PHPDoc，成功!重新执行PHPDoc的导出命令，成功！说明之前找不到GraphViz错误是因为安装顺序导致的，应该先安装GraphViz然后再安装phpDoc,打开生成的phpDoc,Chart也可以正常显示！oye！