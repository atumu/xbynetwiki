location:lua/lua基础学习之版本兼容性
createAt:2016-12-22 17:32:27
modifyAt:2016-12-22 17:32:27
author:xbynet
title:Lua基础学习之版本兼容性

# 与之前版本(5.2)不兼容的地方
该版本主要实现了对整数、位操作、UTF-8 的支持以及打包和解包的功能。
Lua 5.2 到 Lua 5.3 最大的变化是引入了数字的整数子类型,你可以通过把数字都强制转换为浮点数来消除差异 （在 Lua 5.2 中，所有的数字都是浮点数）。
bit32 库废弃了。 使用一个外部兼容库很容易， 不过最好直接用对应的位操作符来替换它。 （注意 bit32 只能针对 32 位整数运算， 而标准 Lua 中的位操作可以用于 64 位整数。）
io.read 的选项名不再用 '*' 打头。 但出于兼容性考虑，Lua 会继续忽略掉这个字符。
数学库中的这些函数废弃了： atan2， cosh， sinh， tanh， pow， frexp， 以及 ldexp 。 你可以用 x^y 替换 math.pow(x,y)；
require 在搜索 C 加载器时处理版本号的方式有所变化。 现在，版本号应该跟在模块名后（其它大多数工具都是这样干的）。 出于兼容性考虑，如果使用新格式找不到加载器的话，搜索器依然会尝试旧格式。
可以参考：http://blog.codingnow.com/2015/01/lua_53_update.html
http://blog.csdn.net/weiyuefei/article/details/43070021

## 5.2与5.1的变化
5.1 与 5.2 是完全不兼容的，相关的第三方库必须重新为 5.2 适配。
luajit不支持5.2，5.3
具体见
http://www.lua.org/manual/5.2/readme.html#changes