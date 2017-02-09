author:xbynet
title:Lua基础学习之面向对象
location:lua/lua基础学习之面向对象
createAt:2016-12-22 22:19:26
modifyAt:2016-12-22 22:26:27

# 表知识回顾
Lua最具特色的数据类型就是 表(Table) , 可以实现 数组 、 Hash 、 **对象** 所有功能的万能数据类型
```
-- array
local array = { 1, 2, 3 }
print(array[1], #array)

-- hash
local hash = { x = 1, y = 2, z = 3 }
print(hash.x, hash['y'], hash["z"], #hash)
```
Tips:

* 数组索引从 1 开始;
* 获取数组长度操作符 # 其’长度’**只包括以 (正)整数 为索引**的数组元素.
* Lua用 表管理全局变量 , 将其放入一个叫 _G 的table内:

```
-- pairs会遍历所有值不为nil的索引, 与此类似的ipairs只会从索引1开始递遍历到最后一个值不为nil的整数索引.
for k, v in pairs(_G) do
    print(k, " -> ", v, " type: " .. type(v))
end
```
# metatable/元表
metatable中的键名称为 事件/event , 值称为 元方法/metamethod , 我们可通过 getmetatable() 来获取任一值的 metatable , 也可通过 setmetatable() 来替换 table 的 metatable .
每个事件的键名用加有 '__' 前缀的字符串来表示； 例如 "add" 操作的键名为字符串 "__add"。 注意、Lua 从元表中直接获取元方法； 访问元表中的元方法永远不会触发另一次元方法。 下面的代码模拟了 Lua 从一个对象 obj 中获取一个元方法的过程：

     rawget(getmetatable(obj) or {}, "__" .. event_name)
     
 Lua 事件一览表:
 
"add": + 操作。 如果任何不是数字的值（包括不能转换为数字的字符串）做加法， Lua 就会尝试调用元方法。 首先、Lua 检查第一个操作数（即使它是合法的）， 如果这个操作数没有为 "__add" 事件定义元方法， Lua 就会接着检查第二个操作数。 一旦 Lua 找到了元方法， 它将把两个操作数作为参数传入元方法， 元方法的结果（调整为单个值）作为这个操作的结果。 如果找不到元方法，将抛出一个错误。
"sub": - 操作。 行为和 "add" 操作类似。
"mul": * 操作。 行为和 "add" 操作类似。
"div": / 操作。 行为和 "add" 操作类似。
"mod": % 操作。 行为和 "add" 操作类似。
"pow": ^ （次方）操作。 行为和 "add" 操作类似。
"unm": - （取负）操作。 行为和 "add" 操作类似。
"idiv": // （向下取整除法）操作。 行为和 "add" 操作类似。
"band": & （按位与）操作。 行为和 "add" 操作类似， 不同的是 Lua 会在任何一个操作数无法转换为整数时 尝试取元方法。
"bor": | （按位或）操作。 行为和 "band" 操作类似。
"bxor": ~ （按位异或）操作。 行为和 "band" 操作类似。
"bnot": ~ （按位非）操作。 行为和 "band" 操作类似。
"shl": << （左移）操作。 行为和 "band" 操作类似。
"shr": >> （右移）操作。 行为和 "band" 操作类似。
"`concat`": .. （连接）操作。 行为和 "add" 操作类似， 不同的是 Lua 在任何操作数即不是一个字符串 也不是数字（数字总能转换为对应的字符串）的情况下尝试元方法。
"`len`": # （取长度）操作。 如果对象不是字符串，Lua 会尝试它的元方法。 如果有元方法，则调用它并将对象以参数形式传入， 而返回值（被调整为单个）则作为结果。 如果对象是一张表且没有元方法， Lua 使用表的取长度操作（参见 §3.4.7）。 其它情况，均抛出错误。
"`eq`": == （等于）操作。 和 "add" 操作行为类似， 不同的是 Lua 仅在两个值都是表或都是完全用户数据 且它们不是同一个对象时才尝试元方法。 调用的结果总会被转换为布尔量。
"lt": < （小于）操作。 和 "add" 操作行为类似， 不同的是 Lua 仅在两个值不全为整数也不全为字符串时才尝试元方法。 调用的结果总会被转换为布尔量。
"le": <= （小于等于）操作。 和其它操作不同， 小于等于操作可能用到两个不同的事件。 首先，像 "lt" 操作的行为那样，Lua 在两个操作数中查找 "__le" 元方法。 如果一个元方法都找不到，就会再次查找 "__lt" 事件， 它会假设 a <= b 等价于 not (b < a)。 而其它比较操作符类似，其结果会被转换为布尔量。
"`index`": 索引 table[key]。 `当 table 不是表或是表 table 中不存在 key 这个键时，这个事件被触发`。 此时，会读出 table 相应的元方法。
尽管名字取成这样， 这个事件的元方法其实可以是一个函数也可以是一张表。 如果它是一个函数，则以 table 和 key 作为参数调用它。 如果它是一张表，最终的结果就是以 key 取索引这张表的结果。 （这个索引过程是走常规的流程，而不是直接索引， 所以这次索引有可能引发另一次元方法。）

"`newindex`": 索引赋值 table[key] = value 。 和索引事件类似，它发生在 table 不是表或是表 table 中不存在 key 这个键的时候。 此时，会读出 table 相应的元方法。
同索引过程那样， 这个事件的元方法即可以是函数，也可以是一张表。 如果是一个函数， 则以 table、 key、以及 value 为参数传入。 如果是一张表， Lua 对这张表做索引赋值操作。 （这个索引过程是走常规的流程，而不是直接索引赋值， 所以这次索引赋值有可能引发另一次元方法。）

一旦有了 "newindex" 元方法， Lua 就不再做最初的赋值操作。 （如果有必要，在元方法内部可以调用 rawset 来做赋值。）

"`call`": 函数调用操作 func(args)。 当 Lua 尝试调用一个非函数的值的时候会触发这个事件 （即 func 不是一个函数）。 查找 func 的元方法， 如果找得到，就调用这个元方法， func 作为第一个参数传入，原来调用的参数（args）后依次排在后面。


-----

对于这些操作, Lua 都将其关联到 metatable 的事件Key, 当 Lua 需要对一个值发起这些操作时, 首先会去检查其 metatable 中是否有对应的事件Key, 如果有则调用之以 控制Lua解释器作出响应 .

## MetaMethods
MetaMethods主要用作一些类似C++中的 运算符重载 操作, 如重载 + 运算符:

```
local frac_a = { numerator = 2, denominator = 3 }
local frac_b = { numerator = 4, denominator = 8 }

local operator = {
    __add = function(f1, f2)
        local ret = {}
        ret.numerator = f1.numerator * f2.denominator + f1.denominator * f2.numerator
        ret.denominator = f1.denominator * f2.denominator
        return ret
    end,

    __tostring = function(self)
        return "{ " .. self.numerator .. " ," .. self.denominator .. " }"
    end
}

setmetatable(frac_a, operator)
setmetatable(frac_b, operator)

local frac_res = frac_a + frac_b
setmetatable(frac_res, operator) -- 使tostring()方法生效
print(tostring(frac_res))
```

# MetaTables 与 面向对象
Lua本来就不是设计为一种 面向对象 语言, 因此其面向对象功能需要通过 元表(metatable) 这种非常怪异的方式实现, Lua并不直接支持面向对象语言中常见的类、对象和方法: 其 对象 和 类 通过 表 实现, 而 方法 是通过 函数 来实现.

上面的 Event一览表 内我们看到有 `__index` 这个事件重载,这个东西主要是重载了 find key 操作, 该操作可以让Lua变得有点面向对象的感觉(类似JavaScript中的 prototype ). 

方法调用的实现
面向对象的基础是创建对象和调用方法. Lua中, 表作为对象使用, 因此创建对象没有问题, 关于调用方法, 如果表元素为函数的话, 则可直接调用:
```
-- 从obj取键为x的值, 将之视为function进行调用
obj.x(foo)
```
不过这种实现方法调用的方式, 从面向对象角度来说还有2个问题:
首先: obj.x 这种调用方式, 只是将表 obj 的属性 x 这个 函数对象 取出而已, 而在大多数面向对象语言中, 方法的实体位于类中, 而非单独的对象中 . 在JavaScript等 基于原型 的语言中, 是 以原型对象来代替类进行方法的搜索 , 因此 每个单独的对象也并不拥有方法实体 . 
在Lua中, 为了实现基于原型的方法搜索, 需要使用元表的 `__index` 事件:
如果我们有两个对象 a 和 b ,想让 b 作为 a 的 prototype 需要 `setmetatable(a, {__index = b})` , 如下例: 为 obj 设置 `__index` 加上 proto 模板来创建另一个实例:

```
proto = {
    x = function()
        print("x")
    end
}

local obj = {}
setmetatable(obj, { __index = proto })
obj.x()
```
proto 变成了原型对象, 当 obj 中不存在的属性被引用时, 就会去搜索 proto .

其次:** 通过方法搜索得到的函数对象只是单纯的函数, 而无法获得最初调用方法的表( 接收器 )相关信息**. 于是, 过程和数据就发生了分离 .JavaScript中, 关于接收器的信息可由关键字 this 获得, 而在Python中通过方法调用形式获得的 并非单纯的函数对象 , 而是一个 “方法对象” –其接收器会在内部 作为第一参数附在函数的调用过程中.

**而Lua准备了支持方法调用的 语法糖 : `obj:x()` . 表示 obj.x(obj) , 也就是: 通过冒号记法调用的函数, 其接收器`self`会被作为第一参数添加进来** ( obj 的求值只会进行一次, 即使有副作用也只生效一次).

```
-- 这个语法糖对定义也有效
function proto:y(param)
    print(self, param)
end

- Tips: 用冒号记法定义的方法, 调用时最好也用冒号记法, 避免参数错乱
obj:y("parameter")
```

## 基于原型的编程
Lua虽然能够进行面向对象编程, 但用元表来实现, 仿佛把对象剖开看到五脏六腑一样.
< 代码的未来 >中松本行弘老师向我们展示了一个基于原型编程的Lua库, 通过该库, 即使没有深入解Lua原始机制, 也可以实现面向对象:
```
--
-- Author: Matz
-- Date: 16/9/24
-- Time: 下午5:13
--

-- Object为所有对象的上级
Object = {}

-- 创建现有对象副本
function Object:clone()
    local object = {}

    -- 复制表元素
    for k, v in pairs(self) do
        object[k] = v
    end

    -- 设定元表: 指定向自身`转发`
    setmetatable(object, { __index = self })

    return object
end

-- 基于类的编程
function Object:new(...)
    local object = {}

    -- 设定元表: 指定向自身`转发`
    setmetatable(object, { __index = self })

    -- 初始化
    object:init(...)

    return object
end

-- 初始化实例
function Object:init(...)
    -- 默认不进行任何操作
end

Class = Object:new()
```

另存为 prototype.lua , 使用时只需 require() 引入即可:

```
require("prototype")

-- Point类定义
Point = Class:new()
function Point:init(x, y)
    self.x = x
    self.y = y
end

function Point:magnitude()
    return math.sqrt(self.x ^ 2 + self.y ^ 2)
end

-- 对象定义
point = Point:new(3, 4)
print(point:magnitude())

-- 继承: Point3D定义
Point3D = Point:clone()
function Point3D:init(x, y, z)
    self.x = x
    self.y = y
    self.z = z
end

function Point3D:magnitude()
    return math.sqrt(self.x ^ 2 + self.y ^ 2 + self.z ^ 2)
end

p3 = Point3D:new(1, 2, 3)
print(p3:magnitude())

-- 创建p3副本
ap3 = p3:clone()
print(ap3.x, ap3.y, ap3.z)
```

参考：http://www.tuicool.com/articles/bUjYv2i
http://coolshell.cn/articles/10739.html