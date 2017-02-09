title: java反射相关知识点 

#  java反射相关知识点 
**Class.isAssignableFrom(Class clz)方法 与 instanceof 关键字的区别**
ClassA.` isAssignableFrom `(ClassB).即A是否为B的父类或接口. 调用者和参数都是java.lang.Class类型。   
` instanceof `是用来判断一个对象实例是否是一个类或接口的或其子类子接口的实例。  
格式是：o instanceof TypeName     
第一个参数是对象实例名，第二个参数是具体的类名或接口名
```

package com.bill99.pattern;

public class AssignableTest {
	
	public AssignableTest(String name) {
	}
	/**
	 * 判断一个类是否是另一个类的父类
	 * 是打印true
	 * 否打印false
	 */
	public static void testIsAssignedFrom1() {
		System.out.println("String是Object的父类:"+String.class.isAssignableFrom(Object.class));
	}
	/**
	 * 判断一个类是否是另一个类的父类
	 * 是打印true
	 * 否打印false
	 */
	public static void testIsAssignedFrom2() {
		System.out.println("Object是String的父类:"+Object.class.isAssignableFrom(String.class));
	}
	/**
	 * 判断一个类是否和另一个类相同
	 * 是打印true
	 * 否打印false
	 */
	public static void testIsAssignedFrom3() {
		System.out.println("Object和Object相同:"+Object.class.isAssignableFrom(Object.class));
	}

	/**
	 * 判断str是否是Object类的实例
	 * 是打印true
	 * 否打印false
	 */
	public static void testInstanceOf1() {
		String str = new String();
		System.out.print("str是Object的实例:");
		System.out.println(str instanceof Object);
	}
	/**
	 * 判断o是否是Object类的实例
	 * 是打印true
	 * 否打印false
	 */
	public static void testInstanceOf2() {
		Object o = new Object();
		System.out.print("o是Object的实例:");
		System.out.println(o instanceof Object);
	}
	
	public static void main(String[] args) {
		testIsAssignedFrom1();
		testIsAssignedFrom2();
		testIsAssignedFrom3();
		testInstanceOf1();
		testInstanceOf2();
	}
}


```
结果:
String是Object的父类:false
Object是String的父类:true
Object和Object相同:true
str是Object的实例:true
o是Object的实例:true

参考：
http://sunnylocus.iteye.com/blog/555676