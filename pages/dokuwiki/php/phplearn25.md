title: phplearn25 

#  PHP学习之常用内置常量 
##  常用预定义常量 
DIRECTORY_SEPARATOR    路径分隔符，Win下是"\"而*inux下是"/"。
PATH_SEPARATOR         多个路劲分隔符，比如使用include多个路劲时候，Win下用";"，而*inux下为":"
PHP_VERSION PHP的版本
PHP_OS 当前服务器的操作系统
TRUE 同true
FALSE 同false
E_ERROR 到最近的错误处
E_WARNING 到最近的警告处
E_PARSE 语法有错误处
E_NOTICE PHP语言中有异常处
M_PI 圆周率
M_E 科学常数e
##  八个魔术常量 
PHP 向它运行的任何脚本提供了大量的预定义常量。不过很多常量都是由不同的扩展库定义的，只有在加载了这些扩展库时才会出现，或者动态加载后，或者在编译时已经包括进去了。
有八个魔术常量它们的值随着它们在代码中的位置改变而改变。例如 _ _LINE_ _ 的值就依赖于它在脚本中所处的行来决定。这些特殊的常量不区分大小写，如下：
![](/data/dokuwiki/php/pasted/20160408-010144.png)
参考:http://www.php.net/manual/zh/language.constants.predefined.php
http://www.php.net/manual/zh/reserved.constants.php