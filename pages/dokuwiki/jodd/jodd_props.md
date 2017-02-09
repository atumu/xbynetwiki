title: jodd_props 

#  Joddprops配置文件 

Jodd Props – 超强的配置文件使用

#  什么是 Jodd Props 

Jodd Props 是对Java自带的properties 的增强，文法风格类似于ini文件，表现力丰富，比json/xml等配置更简单，更可读，更易使用。
Props 补充了很多JDK所需要的: 
对UTF8的支持, 插值, 区段, 多配置, fully configurable… 等等! 
配置可以保存在*.props 文件中, 也可以通过其他形式传入， 
例如：字符串、File、InputStream 、Map等。而且, Props 兼容Java自带的properties.
基本文法.
以下是 props 文件的基本文法。具体用法可见附带的例子:
```

@profiles=proTest

baseValue=baseV
name=base_name_测试
name<proTest>=base_name_proTest

[Cmd]
CipherCache=0
Desc=没有指定1_${baseValue}
name+=${baseValue}

[Cmd<test>]
CipherCache=test
Desc=test_Desc

[Cmd<base>]
CipherCache=base
Desc=base_Desc

#[Cmd<com>]
#CipherCache=base
#Desc=com

[Cmd<jn>]
CipherCache=base
Desc=jn

[Cmd<sd>]
CipherCache=base
Desc=sd

```
##  UTF8 编码 

props 文件采用UTF8 作为默认编码, 当然你也可以指定其它的编码。 但是不管设置了什么编码，Props将会一直使用ISO 8859-1编码加载 Java自带的Properties文件（后缀名是.properties）。
##  注释 

单行注释支持两种符号：;和#。也不必须是每行的第一个符号。
##  使用方法 

很简单. 简单的说，都是交给Props 类。
Props p = new Props();
p.load(new File("example.props"));
...
String story = p.getValue("story");
##  变量 
使用${}引用前门定义的变量。
base=baseValue
base1=my_${base}
##  区段（重要） 

区段看起来和 Windows INI 文件的很相似。在 Props里，区段实际上是 接下来几个属性值的前缀，直到下一个区段，或者文件的末尾。区段名使用[ ]包裹。区段名也可以为空
例如:
[users.data]
weight = 49.5
height = 87.7
age = 63
[]
comment=this is base property
等同于:
users.data.weight = 49.5
users.data.height = 87.7
users.data.age = 63
comment=this is base property
区段, 精简了配置文件，同时更可读
##  多配置 

通常情况下，一个应用将会部署在不同的环境中，于是，需要一些不同的配置。 例如一个应用的开发模式和生产模式。其中一个解决方案就是： 
同一个属性允许配多个不同的值。Props 支持这种多配置.如果没有找到指定配置的属性值, Props 将会 从这些“基础配置”里寻找。这样，配置可以被视为一个“不同的角度的视图”或者 
相同属性集的“快照”
例如:
db.port=3086

db.url<develop>=localhost
db.username<develop>=root

db.url<deploy>=192.168.1.101
db.username<deploy>=app2499
注：develop-开发模式 deploy-生产模式 
上面的例子设置了3个属性， 其中有两个属性有两套配置(develop 和 deploy)没有“基础配置”
由于区段只是属性值的前缀，并且配置也可以放在属性值的中间, 于是，配置也可以卸载区段名里面 
于是，上面的例子也可以写成:
db.port=3086

[db<develop>]
url=localhost
username=root

[db<deploy>]
url=192.168.1.101
username=app2499
当查找值的时候, 就可以指定一个配置:
String url = props.getValue("db.url", "develop");
String user = props.getValue("db.username", "develop");
##  激活配置 

通常, 在应用的生命周期中之会激活一个配置。为了方便每次不用都传入配置文件 
给 getValues()。 Props 允许定义激活的配置。
激活的配置是调用getValue(String)时默认使用的配置。
激活的配置可以在 props 文件中设置 – 这样当修改默认配置的时候就不用重新编译 
源代码。 激活的配置使用一个特殊的名字@profiles 。 
例如:
key1=hello
key1<one>=Hi!

@profiles=one
当在Java获取值得时候:
String value = props.getValue("key1");
因为激活了配置’one‘， 将会得到 ‘Hi!‘。
当然激活的配置也可以在Java里设置，只需要调用方法setActiveProfiles()。


参考：http://blog.csdn.net/songylwq/article/details/25232811