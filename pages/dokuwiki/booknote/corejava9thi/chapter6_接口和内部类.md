title: chapter6_接口和内部类 

#  Chapter6 接口和内部类 
Created Thursday 12 March 2015

#  概要 
到目前为止，读者已经学习了面向对象的全部基本知识(继承，多态，封装。对象和类)。这章节将介绍几个高级技术。
主要介绍：
* 接口(interface):主要用来描述类具有什么功能，而并不给出每个功能的具体实现。一个类可以实现多个接口，并在需要接口的地方使用实现了接口的对象。
* 克隆对象clone：对象的克隆是指创建一个新对象，新对象与原始对象的状态相同。当对克隆的新对象进行状态修改时，不会影响原始对象的状态。也称为深拷贝。
* 内部类inner class机制：主要用于设计具有相互协作关系的类集合。特别是编写GUI事件处理代码时。
* 代理proxy:这是一种实现任意接口的对象。代理是一种非常专业的构造工具。它可以用来构造系统级的工具。

#  接口 
接口不是类，而是对类的一组需求描述。这些类要遵从接口描述的统一格式进行定义。“如果类遵从某个接口，那么就必须履行这项服务”
![](/data/dokuwiki/android/pasted/20150521-063655.png)
![](/data/dokuwiki/android/pasted/20150521-063702.png)

在接口中还可以定义常量
接口中不能含有实例域或者实现方法
implements关键字
![](/data/dokuwiki/android/pasted/20150521-063709.png)
需要学习的类：
java.lang.Comparable<T>
java.util.Arrays
java.lang.Integer

##  接口的特性 
尽管接口不能构造对象，但是却能声明接口的变量。接口变量必须引用实现了接口的类对象。
有些接口自定义了常量，没有定义方法。
示例接口:Cloneable,Comparable

#  对象克隆 
![](/data/dokuwiki/android/pasted/20150521-063718.png)
![](/data/dokuwiki/android/pasted/20150521-063722.png)
![](/data/dokuwiki/android/pasted/20150521-063729.png)
![](/data/dokuwiki/android/pasted/20150521-063734.png)
![](/data/dokuwiki/android/pasted/20150521-063741.png)

Cloneable接口是标记接口。就如Comparable一样。一般的标记接口是没有方法的。通常使用标记接口的目的是为了确保某个类实现了某个特定方法或一组特定方法。

![](/data/dokuwiki/android/pasted/20150521-063750.png)
![](/data/dokuwiki/android/pasted/20150521-063755.png)
![](/data/dokuwiki/android/pasted/20150521-063802.png)
![](/data/dokuwiki/android/pasted/20150521-063806.png)

#  接口与回调 
回调(callback)是一种常见的设计模式。在这种模式中可以指定某个特定事件发生时可以采取的动作。如GUI编程中的事件驱动模型中的事件监听器回调接口。

#  内部类 
内部类inner class.是定义在另外一个类中的类。为什么我们需要内部类?:以下为几点原因：
![](/data/dokuwiki/android/pasted/20150521-063814.png)
![](/data/dokuwiki/android/pasted/20150521-063820.png)
![](/data/dokuwiki/android/pasted/20150521-063825.png)
内部类主要分为以下几类：局部内部类，匿名内部类，静态内部类。全局内部类。

##  由外部方法访问final变量 
![](/data/dokuwiki/android/pasted/20150521-063830.png)
![](/data/dokuwiki/android/pasted/20150521-063837.png)
![](/data/dokuwiki/android/pasted/20150521-063842.png)
![](/data/dokuwiki/android/pasted/20150521-063849.png)

#  代理（proxy） 
![](/data/dokuwiki/android/pasted/20150521-063856.png)
![](/data/dokuwiki/android/pasted/20150521-063937.png)
![](/data/dokuwiki/android/pasted/20150521-063904.png)

InvocationHandler示例：用于处理具体代理调用代码。
```

class TraceHandler implements InvocationHandler
{
   private Object target;

   /**
	* Constructs a TraceHandler
	* @param t the implicit parameter of the method call
	*/
   public TraceHandler(Object t)
   {
	  target = t;
   }

   public Object invoke(Object proxy, Method m, Object[] args) throws Throwable
   {
	  // print implicit argument
	  System.out.print(target);
	  // print method name
	  System.out.print("." + m.getName() + "(");
	  // print explicit arguments
	  **if (args != null) //非null检查**
**      {**
		 for (int i = 0; i < args.length; i++)
		 {
			System.out.print(args[i]);
			if (i < args.length - 1) System.out.print(", ");
		 }
	  }
	  System.out.println(")");

	  // invoke actual method
	  return m.invoke(target, args);
   }
}
下面构造代理对象：
public static void main(String[] args)
   {
	 
		 Integer value = 1;
		 InvocationHandler handler = new TraceHandler(value);
		 Object proxy = Proxy.newProxyInstance(null, new Class[] { Comparable.class } , handler);
		 Object element = proxy;
	  }

	  
	  if (result >= 0) System.out.println(element.compareTo(2));
   }
}

```
##  代理类的特性 
![](/data/dokuwiki/android/pasted/20150521-064014.png)
![](/data/dokuwiki/android/pasted/20150521-064058.png)
![](/data/dokuwiki/android/pasted/20150521-064029.png)

需要学习的类库：
java..lang.reflect.Invocationhandler
java.lang.reflect.Proxy
