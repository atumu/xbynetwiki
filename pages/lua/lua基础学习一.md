location:lua/lua基础学习一
createAt:2016-12-22 17:28:15
modifyAt:2016-12-22 17:37:38
author:xbynet
title:Lua基础学习一

Lua 是一门强大、轻量的嵌入式脚本语言，可供任何需要的程序使用。Lua 没有 "main" 程序的概念： 它只能 嵌入 一个宿主程序中工作.宿主程序可以调用函数执行一小段 Lua 代码，可以读写 Lua 变量，可以注册 C 函数让 Lua 代码调用。
Lua 是一门动态类型语言。 这意味着变量没有类型；只有值才有类型。
Lua 中所有的值都是 一等公民。 这意味着所有的值均可保存在变量中、 当作参数传递给其它函数、以及作为返回值。
文档：5.3：http://cloudwu.github.io/lua53doc/manual.html
5.1：http://book.luaer.cn/
# 基础
## 八种基本类型
Lua 中有八种基本类型： **nil、boolean、number、string、function、userdata、 thread 和 table。**
**Nil** 是值 nil 的类型.和Python中None，Java中null类似。
**Boolean** 是 false 与 true 两个值的类型。与其他语言不通的是**`nil 和 false 都会导致条件判断为假； 而其它任何值都表示为真。`**而其它语言判断比如0时为假,但lua为真。
**Number** 代表了整数和实数（浮点数）。 它也按需作**自动转换**
**String** 表示一个`不可变`的字节序列。Lua 的`字符串与编码无关`； 它不关心字符串中具体内容。
Lua 可以调用（以及操作）用 Lua **或 C** 编写的函数。 这两种函数有统一类型 **function**。
**userdata** 类型允许将 C 中的数据保存在 Lua 变量中。 用户数据类型的值是一个内存块， 有两种用户数据： 完全用户数据 ，指一块由 Lua 管理的内存对应的对象； 轻量用户数据 ，则指一个简单的 C 指针。 **用户数据在 Lua 中除了赋值与相等性判断之外没有其他预定义的操作。** 通过使用 元表 ，程序员可以给完全用户数据定义一系列的操作。 **你只能通过 C API 而无法在 Lua 代码中创建或者修改用户数据的值， 这保证了数据仅被宿主程序所控制。**
**thread** 类型表示了一个独立的执行序列，被用于实现**协程** 。 `Lua 的线程与操作系统的线程毫无关系`。 Lua 为所有的系统，包括那些不支持原生线程的系统，提供了**协程支持**。
**table** 是一个**关联数组**， 也就是说，这个数组不仅仅以数字做索引，除了 nil 和 NaN 之外的所有 Lua 值 都可以做索引。（Not a Number 是一个特殊的数字，它用于表示未定义或表示不了的运算结果，比如 0/0。） 表可以是 异构 的； 也就是说，表内可以包含任何类型的值（ **nil 除外**）。
**表是 Lua 中唯一的数据结构， 它可被用于表示普通数组、序列、符号表、集合、记录、图、树等等。** 对于记录，Lua 使用域名作为索引。 语言提供了 a.name 这样的**语法糖**来替代 a["name"]。 
我们使用 **序列** 这个术语来表示一个用 {1..n} 的正整数集做索引的表.`注意：lua中索引从1开始，而非0`
和索引一样，表中每个域的值也可以是任何类型。 需要特别指出的是：**既然函数是一等公民，那么表的域也可以是函数**。 这样，表就可以携带 方法 了。

表、函数、线程、以及完全用户数据在 Lua 中被称为 **对象**： 变量并不真的 持有 它们的值，而仅保存了对这些对象的 引用。 赋值、参数传递、函数返回，都是**针对引用**而不是针对值的操作， 这些操作均不会做任何形式的隐式拷贝。

### 类型判断
**库函数 type** 用于以字符串形式返回给定值的类型。

## 错误处理
由于 Lua 是一门嵌入式扩展语言，其所有行为均源于宿主程序中 C 代码对某个 Lua 库函数的调用。 （单独使用 Lua 时，lua 程序就是宿主程序。） 所以，在编译或运行 Lua 代码块的过程中，无论何时发生错误， 控制权都返回给宿主，由宿主负责采取恰当的措施（比如打印错误消息）。
可以在 Lua 代码中调用 **error 函数**来显式地抛出一个错误。 如果你需要在 Lua 中捕获这些错误， 可以**使用 pcall 或 xpcall 在 保护模式 下调用一个函数。**
无论何时出现错误，都会抛出一个携带错误信息的 错误对象 （错误消息）,这是一个字符串对象。
使用 xpcall 或 lua_pcall 时， 你应该提供一个 消息处理函数 用于错误抛出时调用。 该函数需接收原始的错误消息，并返回一个新的错误消息。 

## 元表及元方法
Lua 中的每个值都可以有一个 元表。 这个 元表 就是一个普通的 Lua 表， 它用于定义原始值在特定操作下的行为。元表中的键对应着不同的 事件 名； 键关联的那些值被称为 元方法。 
你可以用 `getmetatable` 函数 来获取任何值的元表。
使用 `setmetatable` 来替换一张表的元表。在 Lua 中，你不可以改变表以外其它类型的值的元表 （除非你使用调试库）； 若想改变这些非表类型的值的元表，请使用 C API。
表和完全用户数据有独立的元表 （当然，多个表和用户数据可以共享同一个元表）。 其它类型的值按类型共享元表； 也就是说所有的数字都共享同一个元表， 所有的字符串共享另一个元表等等。
元表决定了一个对象在数学运算、位运算、比较、连接、 取长度、调用、索引时的行为。 元表还可以定义一个函数，当表对象或用户数据对象在垃圾回收时调用它。
其它具体见文档。。。。

## 垃圾收集
Lua 采用了自动内存管理。Lua 实现了一个增量标记-扫描收集器。
垃圾收集元方法：
你可以为表设定垃圾收集的元方法，对于完全用户数据， 则需要使用 C API 。 该元方法被称为 终结器。 终结器允许你配合 Lua 的垃圾收集器做一些额外的资源管理工作 （例如关闭文件、网络或数据库连接，或是释放一些你自己的内存）。
如果要让一个对象（表或用户数据）在收集过程中进入终结流程， 你必须 标记 它需要触发终结器。 当你为一个对象设置元表时，若此刻这张元表中用一个以字符串 "`__gc`" 为索引的域，那么就标记了这个对象需要触发终结器。

## 协程
调用函数 `coroutine.create` 可创建一个协程。 其唯一的参数是该协程的主函数。 create 函数只负责新建一个协程并返回其句柄 （一个 thread 类型的对象）； 而不会启动该协程。

调用 `coroutine.resume` 函数执行一个协程。 
协程的运行可能被两种方式终止： 正常途径是主函数返回 （显式返回或运行完最后一条指令）； 非正常途径是发生了一个未被捕获的错误。 **对于正常结束， coroutine.resume 将返回 true， 并接上协程主函数的返回值。 当错误发生时， coroutine.resume 将返回 false 与错误消息。**

通过调用 `coroutine.yield` 使协程暂停执行，让出执行权。 协程让出时，对应的最近 coroutine.resume 函数会立刻返回，即使该让出操作发生在内嵌函数调用中 （即不在主函数，但在主函数直接或间接调用的函数内部）。 **在协程让出的情况下， coroutine.resume 也会返回 true， 并加上传给 coroutine.yield 的参数。** 当下次重启同一个协程时， 协程会接着从让出点继续执行。

与 coroutine.create 类似， `coroutine.wrap` 函数也会创建一个协程。 不同之处在于，它不返回协程本身，而是返回一个函数。 调用这个函数将启动该协程。 传递给该函数的任何参数均当作 coroutine.resume 的额外参数。 **coroutine.wrap 返回 coroutine.resume 的所有返回值，除了第一个返回值（布尔型的错误码）。** 和 coroutine.resume 不同， **coroutine.wrap 不会捕获错误； 而是将任何错误都传播给调用者。**

下面的代码展示了一个协程工作的范例：

         function foo (a)
           print("foo", a)
           return coroutine.yield(2*a)
         end
         
         co = coroutine.create(function (a,b)
               print("co-body", a, b)
               local r = foo(a+1)
               print("co-body", r)
               local r, s = coroutine.yield(a+b, a-b)
               print("co-body", r, s)
               return b, "end"
         end)
         
         print("main", coroutine.resume(co, 1, 10))
         print("main", coroutine.resume(co, "r"))
         print("main", coroutine.resume(co, "x", "y"))
         print("main", coroutine.resume(co, "x", "y"))
    当你运行它，将产生下列输出：
    
         co-body 1       10
         foo     2
         main    true    4
         co-body r
         main    true    11      -9
         co-body x       y
         main    true    10      end
         main    false   cannot resume dead coroutine

你也可以通过 C API 来创建及操作协程： 参见函数 lua_newthread， lua_resume， 以及 lua_yield。

# 命令行
独立版 Lua
虽然 Lua 被设计成一门扩展式语言，用于嵌入一个宿主程序。 但经常也会被当成独立语言使用。 独立版的 Lua 语言解释器随标准包发布，就叫 lua。 其命令行用法为：

    lua [options] [script [args]]

选项有：

    -e stat: 执行一段字符串 stat ；
    -l mod: “请求模块” mod ；
    -i: 在运行完 脚本 后进入交互模式；
    -v: 打印版本信息；
    -E: 忽略环境变量；
    --: 中止对后面选项的处理；
    -: 把 stdin 当作一个文件运行，并中止对后面选项的处理。

在处理完选项后，lua 运行指定的 脚本。 如果不带参数调用， 在标准输入（stdin）是终端时，lua 的行为和 lua -v -i 相同。 否则相当于 lua - 。

为了让 Lua 可以用于 Unix 系统的脚本解释器。
#!/usr/local/bin/lua
如果 lua 在你的 PATH 中， **写成 #!/usr/bin/env lua更为通用。**

```
-- file 'lib1.lua'
 
function norm (x, y)
    local n2 = x^2 + y^2
    return math.sqrt(n2)
end
 
function twice (x)
    return 2*x
end
```
在交互模式下：

    > lua -i 
    > dofile("lib1.lua")     -- load your library
    > n = norm(3.4, 1.0)
    > print(twice(n))        --> 7.0880180586677

-i和dofile在调试或者测试Lua代码时是很方便的。

命令行方式
```
> lua -e "print(math.sin(12))"   --> -0.53657291800043
```

全局变量arg存放Lua的命令行参数。
prompt> lua script a b c
在运行以前，Lua使用所有参数构造`arg`表。脚本名索引为0，**脚本的参数从1开始增加**。脚本前面的参数从-1开始减少。

# 语言定义
Lua 语言的格式自由。 它会忽略语法元素（符记）间的空格（包括换行）和注释， 仅把它们看作为名字和关键字间的分割符。
Lua 语言对大小写敏感。
字面串 可以用单引号或双引号括起。 字面串内部可以包含下列 C 风格的转义串。
`转义串 '\z'` 会忽略其后的一系列空白符，包括换行； 它在你需要对一个很长的字符串常量断行为多行并希望在每个新行保持缩进时非常有用。
对于用 UTF-8 编码的 Unicode 字符，你可以用 转义符 \u{XXX} 来表示 

代码注释：

    --这是行注释
    --[[这是块
    注释]]

## 变量：
Lua 中有三种变量： 全局变量、局部变量和表的域。
**所有没有显式声明为局部变量的变量名都被当做`全局变量`。** 这一点倒是和js很相似。
全局变量 x = 1234 的赋值等价于 `_ENV`.x = 1234
局部变量 `local` namelist [‘=’ explist] 比如 local name='xbynet'

## 语句：
每个语句结尾的分号（;）是可选的，但如果同一行有多个语句最好用;分开

if, while, and repeat 这些控制结构符合通常的意义，而且也有类似的语法：

    while exp do block end
    repeat block until exp
    if exp then block elseif exp then block else block end

for 有两种形式：一种是数字形式，另一种是通用形式。
数值for循环：

    for var=exp1,exp2,exp3 do
        loop-part
    end

**for将用exp3作为step从exp1（初始值）到exp2（终止值），执行loop-part。其中exp3可以省略，默认step=1**
有几点需要注意：
1. **三个表达式只会被计算一次，并且是在循环开始前。**

    for i=1,f(x) do
        print(i)
    end
     
    for i=10,1,-1 do
        print(i)
    end

第一个例子f(x)**只会在循环前被调用一次**。

通用形式的 for 通过一个叫作 **迭代器** 的函数工作。 每次迭代，迭代器函数都会被调用以产生一个新的值， **当这个值为 nil 时，循环停止**。 

    for var_1, ···, var_n in explist do block end
    
    -- print all values of array 'a'
    for i,v in ipairs(a) do print(v) end
    
    再看一个遍历表key的例子：
    -- print all keys of table 't'
    for k in pairs(t) do print(k) end  
     
控制结构中的条件表达式可以返回任何值。 false 与 nil 两者都被认为是假。 **所有不同于 nil 与 false 的其它值都被认为是真 （特别需要注意的是，数字 0 和空字符串也被认为是真）。**
**有break，但是没有continue**
return 被用于从函数或是代码块（其实它就是一个函数） 中返回值。 函数可以返回不止一个值。
**return 只能被写在一个语句块的最后一句。** 如果你真的需要从语句块的中间 return， 你可以使用显式的定义一个内部语句块， 一般写作 do return end。 可以这样写是因为现在 return 成了（内部）语句块的最后一句了。

**`Lua语法要求break和return只能出现在block的结尾一句`**（也就是说：作为chunk的最后一句，或者在end之前，或者else前，或者until前），例如：

    local i = 1
    while a[i] do
        if a[i] == v then break end
        i = i + 1
    end

有时候为了调试或者其他目的需要在block的中间使用return或者break，可以显式的使用do..end来实现：

    function foo ()
        return            --<< SYNTAX ERROR
        -- 'return' is the last statement in the next block
        do return end        -- OK
        ...               -- statements not reached
    end

## 表达式
数学运算操作符

    +: 加法
    -: 减法
    *: 乘法
    /: 浮点除法
    //: 向下取整除法
    %: 取模
    ^: 乘方
    -: 取负

比较操作符

    ==: 等于
    ~=: 不等于
    <: 小于
    >: 大于
    <=: 小于等于
    >=: 大于等于
    
**注意：不等于是~=,而不是!=**
这些操作的结果不是 false 就是 true。
**等于操作 （==）先比较操作数的类型。 如果类型不同，结果就是 false。 否则，继续比较值。** 字符串按一般的方式比较。 数字遵循二元操作的规则
表，用户数据，以及线程都按引用比较： 只有两者引用同一个对象时才认为它们相等。
你可以通过使用 "eq" 元方法来改变 Lua 比较表和用户数据时的方式。

逻辑操作符
Lua 中的逻辑操作符有 `and， or，以及 not`。 
和控制结构一样， 所有的逻辑操作符把 false 和 nil 都作为假， 而其它的一切都当作真。

**字符串连接**
Lua 中字符串的连接操作符写作两个点（'`..`'）。 如果两个操作数都是字符串或都是数字， 连接操作将以中提到的规则把其转换为字符串。 否则，会调用元方法 __concat 
**多行字符串**
还可以使用[[...]]表示字符串。这种形式的字符串可以包含多行
```
page = [[
qwwqwq
adas
ss
]]
```

**取长度操作符**
取长度操作符写作一元前置符 `#`。 字符串的长度是它的字节数（就是以一个字符一个字节计算的字符串长度）。而Python是内置的len()函数，Java和JS都是字符串函数.length()
程序可以通过 __len 元方法来修改对字符串类型外的任何值的取长度操作行为。


### 表构建
表构造子是一个构造表的表达式。 每次构造子被执行，都会构造出一张新的表。 构造子可以被用来构造一张空表， 也可以用来构造一张表并初始化其中的一些域。
构造器是创建和初始化表的表达式。表是Lua特有的功能强大的东西。最简单的构造函数是`{}`，用来创建一个空表。可以直接初始化数组:
    days = {"Sunday", "Monday", "Tuesday", "Wednesday",
                  "Thursday", "Friday", "Saturday"}

Lua将"Sunday"初始化days[1]（**第一个元素索引为1**），用"Monday"初始化days[2]...
如果想初始化一个表作为record使用可以这样：
    a = {x=0, y=0}       <-->       a = {}; a.x=0; a.y=0
不管用何种方式创建table，我们都可以向表中添加或者删除任何类型的域，构造函数仅仅影响表的初始化。

    w = {x=0, y=0, label="console"}
    x = {sin(0), sin(1), sin(2)}
    w[1] = "another field"
    x.f = w
    print(w["x"])     --> 0
    print(w[1])       --> another field
    print(x.f[1])     --> another field
    w.x = nil         -- remove field "x"
值得注意的是：w.x = nil         -- remove field "x"

在构造函数中域分隔符逗号（","）可以用分号（";"）替代，通常我们使用分号用来分割不同类型的表元素。
    {x=10, y=45; "one", "two", "three"}

举个例子：
    a = { [f(1)] = g; "x", "y"; x = 1, f(x), [30] = 23; 45 }
等价于

     do
       local t = {}
       t[f(1)] = g
       t[1] = "x"         -- 1st exp
       t[2] = "y"         -- 2nd exp
       t.x = 1            -- t["x"] = 1
       t[3] = f(x)        -- 3rd exp
       t[30] = 23
       t[4] = 45          -- 4th exp
       a = t
     end

如果表单中最后一个域的形式是 exp ， 而且其表达式是一个函数调用或者是一个可变参数， 那么这个表达式所有的返回值将依次进入列表

### 函数
函数定义

     function f () body end
     local function f () body end
     
当一个函数被调用， 如果函数并非一个 可变参数函数， 即在形参列表的末尾注明三个点 ('`...`')， 那么实参列表就会被调整到形参列表的长度。

```
     function f(a, b) end
     function g(a, b, ...) end
     function r() return 1,2,3 end
```
下面看看实参到形参数以及可变长参数的映射关系：

     CALL            PARAMETERS
     
     f(3)             a=3, b=nil
     f(3, 4)          a=3, b=4
     f(3, 4, 5)       a=3, b=4
     f(r(), 10)       a=1, b=10
     f(r())           a=1, b=2
     
     g(3)             a=3, b=nil, ... -->  (nothing)
     g(3, 4)          a=3, b=4,   ... -->  (nothing)
     g(3, 4, 5, 8)    a=3, b=4,   ... -->  5  8
     g(5, r())        a=5, b=1,   ... -->  2  3
     
**冒号 语法**可以用来定义 方法， 就是说，函数可以有一个隐式的形参 self。 因此，如下语句
     function t.a.b.c:f (params) body end
是这样一种写法的语法糖
     t.a.b.c.f = function (self, params) body end

**Lua函数可以返回多个结果值**

```
function maximum (a)
    local mi = 1             -- maximum index
    local m = a[mi]          -- maximum value
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end
 
print(maximum({8,10,23,12,5}))     --> 23   3
```
Lua总是调整函数返回值的个数以适用调用环境，当作为独立的语句调用函数时，所有返回值将被忽略
第一，当作为表达式调用函数时，有以下几种情况：
1. 当调用作为表达式最后一个参数或者仅有一个参数时，根据变量个数函数尽可能多地返回多个值，不足补nil，超出舍去。
2. 其他情况下，函数调用仅返回第一个值（如果没有返回值为nil）
第二，函数调用作为函数参数被调用时，和多值赋值是相同。
第三，函数调用在表构造函数中初始化时，和多值赋值时相同。

可变参数与命名参数

```
printResult = ""
 
function print(...)
    for i,v in ipairs(arg) do
       printResult = printResult .. tostring(v) .. "\t"
    end
    printResult = printResult .. "\n"
end
```
命名参数用表来作为参数传递。
     
## 可见性规则
Lua 语言有词法作用范围。 变量的作用范围开始于声明它们之后的第一个语句段， 结束于包含这个声明的最内层语句块的最后一个非空语句。 

```
     x = 10                -- 全局变量
     do                    -- 新的语句块
       local x = x         -- 新的一个 'x', 它的值现在是 10
       print(x)            --> 10
       x = x+1
       do                  -- 另一个语句块
         local x = x+1     -- 又一个 'x'
         print(x)          --> 12
       end
       print(x)            --> 11
     end
     print(x)              --> 10 （取到的是全局的那一个）
```
局部变量可以被在它的作用范围内定义的函数自由使用。 当一个局部变量被内层的函数中使用的时候， 它被内层函数称作 **上值**，或是 **外部局部变量**。

注意，每次执行到一个 local 语句都会定义出一个新的局部变量。 看看这样一个例子：

     a = {}
     local x = 20
     for i=1,10 do
       local y = 0
       a[i] = function () y=y+1; return x+y end
     end

这个循环创建了十个**闭包**（这指十个匿名函数的实例）。 这些闭包中的每一个都使用了不同的 y 变量， 而它们又共享了同一份 x。