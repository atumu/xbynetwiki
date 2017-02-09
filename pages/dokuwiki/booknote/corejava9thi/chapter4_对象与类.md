title: chapter4_对象与类 

#  Chapter4 对象与类 
Created Tuesday 10 March 2015

#  面向对象程序设计概述 
OOP面向对象程序设计
面向对象的程序是由对象组成。每个对象包含对用户公开的特定功能部分和隐藏的实现部分。
OOP与面向过程的区别：算法+数据结构=程序，OOP将数据放在第一位，而后者则反之。
面向对象更加适用于规模较大的问题。

##  类 
类是构造对象的模板。
封装（encapsulation）：将数据和行为组合在一起，并对对象的使用者隐藏了数据的实现方式。对象中的数据成为实例域，这些值的集合就是这个对象的当前状态。操作数据的过程称为方法。
无论何时，只要向对象发送一个消息（方法调用），它的状态就可能会改变。
程序仅仅通过方法与对象数据进行交互，而不是直接与对象数据进行交互。
Object： java中的顶级超类

##  对象 
要想使用OOP，必须明白对象的三个特性：
	* 对象的行为
	* 对象的状态
	* 对象的标识

##  识别类 
“找名词与动词”原则

##  类之间的关系 
* 依赖uses-a :dependency如果一个类的方法操作另一个类的对象。我们就说一个类依赖另一个类。
* 聚合 has-a：aggregation
* 继承 is-a : inheritance :表示特殊与一般的关系。
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-061853.png)

#  使用预定义类 
java中，并不是所有类都具有面向对象的特征。比如Math类。因为它只封装了功能，它不需要也不必隐藏数据。由于没有数据。因此也没有必要实例化和生成对象。

##  对象与对象变量 
注意：一个对象变量并没有实际包含一个对象，它仅仅是引用一个对象。

##  java类库中的java.util.GregorianCalendar 

##  隐式参数与显式参数 
this隐式参数
注意：尽量不要编写返回引用可变对象的访问器方法，因为这破坏了封装性。安全性的应该先对它进行克隆clone然后再返回。

##  final域 

#  静态域和静态方法 
注意：本地方法可以绕过java的存取控制机制。

#  方法参数 
java中的方法参数都是按值调用的。对象引用进行的是值传递。这点要特别注意。

#  对象构造 
重载overloading：重载多个方法，重载多个构造器
无参构造器
使用this（...）调用构造器。
初始化块。

##  对象析构与finalize方法 

#  包 

##  类的导入 
注意：import java.util.*对代码的大小没有任何负面影响。

##  静态导入 
import static java.lang.System.*

##  包密封机制保证代码的安全性 

#  类路径 

#  文档注释 
javadoc工具。```
/** */</code>
文档注释的插入位置可以为：
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-061905.png)
[[/**]] */之后的第一句为概要性句子。
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-061910.png)
	
##  类注释 
可以添加@author标记
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-061943.png)

##  方法注释 
<code>
@param
@return
@throws
/**
		Raises the salary of an employee
		@param byPercent the percentage by which to raise salary
		@return the amount of the raise
	*/	
		public double raiseSalary(double byPercent){
			
		}

```
##  域注释 
```

[[/**]]
	The card suite
*/
public static final int HART=1;

```
##  通用注释 
```

@author 姓名
@version 文本
@since 文本
@deprecated 文本
@see与@link可以使用超链接。链接到文档的相关部分或者外部文档
@see引用: @see net.xby1993.Card#raiseSalary(double); @see <a href="www.baidu.com">baidu</a>; @see "haha";
{@link package.class#feature label}如{@link net.xby1993.Card#raiseSalary seeRaise};
	/**
		Raises the salary of an employee
		@param byPercent the percentage by which to raise salary
		@return the amount of the raise
		@author xby
		@version 1
		@since 1.0
		@deprecated 
		@see Card#raise(double) 
		@see <a href="www.badu.com">baidu</a> 
		@see "core java"
		
	*/	
		public double raiseSalary(double byPercent){

		}
		public double raise(double byPercent){

		}

```
##  包与概述注释 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062031.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062037.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062044.png)

#  类设计技巧 
* 一定要保证数据私有：绝对不要破坏封装性。
* 一定要对数据初始化：对数据进行显式初始化，不要依赖系统默认行为
* 不要在类中使用过多的基本类型：一些基本类型可以组合成一个对象类型。
* 不是所有的域都需要独立的域访问器和域更改器：有些只需要访问，有些只需要修改
* 将职责过多的类进行分解：一个类应该只负责自己的职责。
* 类名和方法名应该能体现他们的职责

下一章学习面向对象的其他技术继承和多态。

