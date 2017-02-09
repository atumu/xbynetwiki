title: chapter5_继承 

#  Chapter5 继承 
Created Wednesday 11 March 2015

继承(inheritance)

#  类、子类、超类 
'is-a'关系是继承的一个明显的特性。
super关键字

##  多态与动态绑定 
多态(polymorphism):一个变量可以指示多种实际类型的现象。
动态绑定（dynamic binding）:在运行时能够自动地选择调用哪个方法的现象。

##  继承层次(inheritance hierarchy) 
继承链（inheritance chain）

##  多态 
is-a原则用来判断是否应该设计为继承关系。它表明子类的每个对象也是超类的对象。
is-a原则另一种表述就是置换原则。它表明程序中出现超类对象的任何地方都可以用子类对象置换。

##  动态绑定 

##  阻止继承：final类和方法 

##  强制类型转换 
instanceof运算符

##  抽象类 
抽象类可以包含具体数据和方法
抽象类不能被实例化

##  受保护的访问protected 
应该谨慎使用protected修饰符以确保最大限度的封装性。

###  关于java中用于控制可见性的4个修饰符 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062143.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062150.png)

#  Object：所有类的超类 
在java中只有基本类型不是Object

##  equals方法 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062156.png)
```

 private String name;
   private double salary;
   private Date hireDay;
  	   /**
	* 这是equals比较全面的稳定的写法。先判断是否==如果否，,然后再判断是否为null为后续判断奠定基础，如果否，则需要判断是否属于同一个类型，
		如果为是，此时我们便可以进行转换。
	* 使用Objects.equals()可以防止出现null，
	* 使用Double.compare()用于比较浮点数。记住不能直接用==比较浮点数，
	* @param obj 
	* @return boolean
		*/
	   @Override
	   public boolean equals(Object obj){
		   if(this==obj) return true;
		   if(obj==null) return false;
		   if(getClass()!=obj.getClass()) return false;
		   Employee other=(Employee)obj;
		   
		   return Objects.equals(name, other.name)
				   &&(Double.compare(salary, other.salary)==0?true:false)
				   &&Objects.equals(hireDay, other.hireDay);
	   }

```	   
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062214.png)

###  相等测试与继承 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062219.png)

###  完美equals写法的建议： 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062226.png)

注意：对于数组类型的域，可以使用Arrays.equals()方法进行判断相等性。

##  hashCode方法 
散列码（hash code）是由对象导出的一个整型值。没有规律。
```

 public int hashCode()
   {
	  return Objects.hash(name, salary, hireDay); 
   }

```
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062331.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062254.png)
、
注意：如果重新定义了equals()方法就必须重新定义hashCode方法。

![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062338.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062347.png)

##  toString方法 
用于表示对象值的字符串
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062401.png)

注意：关于数组的打印：Arrays.toString(array);
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062405.png)

#  泛型数组列表 
ArrayList<?>采用类型参数的泛型类
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062417.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062749.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062425.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062550.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062445.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062459.png)

#  对象包装器与自动装箱 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-062811.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063005.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063020.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063038.png)

#  参数数量可变的方法 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063108.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063115.png)

#  枚举类 
public enum Size{SMALL,LARGE，MEDIUM}
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063127.png)

#  反射 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063138.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063143.png)

##  Class类 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063156.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063215.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063225.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063236.png)

##  利用反射分析类的能力 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063322.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063326.png)

##  在运行时使用反射分析对象 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063337.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063344.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063349.png)

##  使用反射编写泛型数组代码 

##  调用任意方法 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063408.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063412.png)

#  继承设计的技巧 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063427.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063545.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063436.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063443.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063449.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063502.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063456.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063511.png)
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-063517.png)

#  需要学习的类库： 
java.lang.reflect.Array
java.lang.reflect包
java.lang.reflect.Constructor
java.lang.Throwable
java.lang.Enum<E>
java.lang.Integer
java.text.NumberFormat
java.util.ArrayList<T>
java.util.Arrays
java.util.Objects :since java7
java.lang.Object
java.lang.Class


